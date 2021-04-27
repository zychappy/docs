# ES选举原理

## 选举算法

[Bully算法](https://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)

1. 假设集群有6个node，n1~n6，拥有最大number的n6是leader；
2. leader宕机，n3发现了，于是发起选举，向number比自己大的node，n4，n5，n6发送请求信息，只要有一个相应，n3就会放弃选举， 这是n4和n5会响应n3的请求；
3. n4开始向n5，n6发起选举请求，n5响应；
4. n5开发向n6发起请求，n6无响应，于是n5通知集群中其他节点他成为新的leader；

## 选举时间点

1. 集群启动初始化；
2. 集群master挂了；
3. 任何一个节点发现当前集群中的Master节点没有得到n/2 + 1节点认可的时候，触发选举 ，n为当前集群总节点数；

## 选举流程

### 1 找到activeMasters列表

Elasticsearch节点成员首先向集群中的所有成员发送Ping请求，elasticsearch默认等待discovery.zen.ping_timeout时间，
然后elasticsearch针对获取的全部response进行过滤，筛选出其中activeMasters列表，activeMaster列表是其它节点认为的当前集群的Master节点。
elasticsearch在获取activeMasters列表的时候会排除本地节点。  
`目的是为了避免脑裂，假设这样一个场景，当前最小编号的节点P0认为自己就是master并且P0和其它节点发生网络分区，
同时es允许将自己放在activeMaster中，因为P0编号最小，那么P0永远会选择自己作为master节点，那么就会出现脑裂的情况。
`
```java
List<DiscoveryNode> activeMasters = new ArrayList<>();
    for (ZenPing.PingResponse pingResponse : pingResponses) {
        // We can't include the local node in pingMasters list, otherwise we may up electing ourselves without
        // any check / verifications from other nodes in ZenDiscover#innerJoinCluster()
        if (pingResponse.master() != null && !localNode.equals(pingResponse.master())) {
            activeMasters.add(pingResponse.master());
        }
    }
```
### 2 找到所有的可成为master的节点集合masterCandidates
```java
//配置某个节点没有成为master资格
  //  node.master:false
    // nodes discovered during pinging
    List<ElectMasterService.MasterCandidate> masterCandidates = new ArrayList<>();
    for (ZenPing.PingResponse pingResponse : pingResponses) {
        if (pingResponse.node().isMasterNode()) {
            masterCandidates.add(new ElectMasterService.MasterCandidate(pingResponse.node(), pingResponse.getClusterStateVersion()));
        }
    }
```
### 3 检查activeMasters列表
如果activeMasters 不为空，则从中选择最小的ID成为Master节点；
```java
if (activeMasters.isEmpty()) {
        if (electMaster.hasEnoughCandidates(masterCandidates)) {
            final ElectMasterService.MasterCandidate winner = electMaster.electMaster(masterCandidates);
            logger.trace("candidate {} won election", winner);
            return winner.getNode();
        } else {
            // if we don't have enough master nodes, we bail, because there are not enough master to elect from
            logger.warn("not enough master nodes discovered during pinging (found [{}], but needed [{}]), pinging again",
                        masterCandidates, electMaster.minimumMasterNodes());
            return null;
        }
    } else {
        assert !activeMasters.contains(localNode) : "local node should never be elected as master when other nodes indicate an active master";
        // lets tie break between discovered nodes
        return electMaster.tieBreakActiveMasters(activeMasters);
    }
```
### 4 从masterCandidates列表选举Master节点
如果列表为空，也就是说不存在活着的master节点，
同时当前活着的节点满足配置中discovery.zen.minimum_master_nodes的数量，那么就从masterCandidates 挑出ID最小的节点，让其成为master节点。
先比较集群版本号，相同的情况下再比较节点id。
```java
public static int compare(MasterCandidate c1, MasterCandidate c2) {
    int ret = Long.compare(c2.clusterStateVersion, c1.clusterStateVersion);
    if (ret == 0) {
        ret = compareNodes(c1.getNode(), c2.getNode());
    }
    return ret;
}
```
`集群中节点的id是由discovery定义的，默认es有两种实现方式，一种是
org.elasticsearch.discovery.local.LocalDiscovery
表示把es的节点启动在同一个jvm的环境下，这样就可以通过AtomicLong来进行数字递增的id生成。
另一种是
org.elasticsearch.discovery.zen.ZenDiscovery
它是分布式环境下的节点发现机制，由于是分布式环境，数字递增形式比较难行得通，于是在zenDiscovery里面是使用64位的uuid来作为节点id。
每次节点重启其id都是会变的，重新生成一个uuid。`
### 5 确定master节点
经过3,4两步，其实是生成一个准master节点，接下来
1. 如果本节点被选为临时Master，等待（默认30s）足够多节点（过半还是？）加入本节点（如果有discovery.zen.minimum_master_nodes-1个节点投票认为当前节点是master）后，完成选举，并发布新的clusterState；
2. 如果其他节点本选为临时Master节点，向Master节点发送加入集群的请求，等待（1分钟）回复，最终master会发布集群状态，并确认客户join请求；
   如果失败，重新再来新一轮的选举；

### 6 节点失效监测
1. Master节点启动NodesFaultDetection定期监测加入集群的节点是否活跃；
2. 非Master节点启动MasterFaultDetection定期监测Master节点是否活跃；
   节点监测都是通过定期（1s）发送ping探测节点是否正常，如果超过3次，则开始处理节点离开事件；

### 7 NodesFaultDetection事件处理
检查集群是否达到了法定节点数（过半），如果不足，则放弃Master重新加入集群，
这样做的目的是为了防止产生双主，例如集群中有5个节点[A,B,C,D,E]，A为主节点，由于某种原因，
集群网络出现分区[A,B],[C,D,E],这时候[C,D,E] 组成的分区监测不到主节点开始处理MasterFaultDetection重新选举主节点，
假设选举C为主节点；由于A节点无法监测到[C,D,E]三个节点，开始处理NodesFaultDetection，
这时候如果A所在分区如果不监测法定节点数，继续作为主节点；整个集群会出现两个主节点A,C;

### 8 MasterFaultDetection事件处理
监测到主节点离线后回重新监测法定节点数，并进行新一轮的选择；

# 参考资料
[elasticsearch更改node id生成方法](https://blog.csdn.net/july_2/article/details/38320519)  
[ES 选举Master节点机制](https://www.cnblogs.com/arax/p/14426285.html)  
[Elasticsearch选举流程详解](https://zhuanlan.zhihu.com/p/110079342)  
