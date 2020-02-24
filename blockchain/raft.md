什么是一致性分布式算法raft？  
首先我们从一个单节点开始，你可以一个单点数据库，保存着唯一一份数据value=8；  
然后一个客户端（client）发来数据将这份数据修改了value=10；然后这个value就是10了，
就一个数据库，他说是啥就是啥；  
进一步，如果我们有3台数据库，客户端只是将一台修改了，其他两台还没有同步修改，这就是
分布式一致性算法需要做的事情：如何能够准确有效地同步数据。  
主要过程：  
1、节点有三种状态，follower，candidate，leader；  
2、一开始的状态都是follower；  
3、如果follower得不到leader的响应，自己就变为candidate；  
4、candidate向其他节点发起投票，如果得到超过半数节点的投票vote，就会变成leader，这个过程称为**leader election**；  
5、在leader出现后，所有的数据变动都要个leader一致，就是所谓的跟随；
6、每次改动，都作为一个entry加入到node的log中，这个 log entry现在还未提交，所以节点的数据也没变化；  
7、leader开始将此次改动发给follower，直到超过半数的follower更新entry，然后leader才将此次改动正式入库；  
8、在leader正式改动后，通知节点，此时该entry已提交完毕，各节点跟随，此时整个集群才算对此次修改达成共识，这个过程称为**Log Replication**；  

####Leader Election
1、两个超时timeout来控制选举；  
2、一个是election timeout，指的是follower在等待leader响应超时，然后变为candidate，一般为150-300ms之间的随机数；  
3、election timeout后就开始了新一轮选举，candidate向其他节点发起投票，并且各个节点重置election timeout，candidate在收到超半数投票后变为leader；  
4、leader不断发送Append Entries给followers，就是心跳检测，followers每个都要响应；
5、只要followers能正常收到leader的心跳检测，本次的election term就会继续有效，直到某个follower出现1的情况，该节点就会重复以上步骤进入下一个任期，分为几种情况；  
5.1 一种是leader宕机了，某个follower会率先变为candidate，然后重复步骤2；
5.2 另一种可能是网络问题，  
6、同时出现两个candidate的情况（比如4个node的情况） 
6.1 两个candidate同时向其他节点发起投票，并且都收到了同样的follower的投票，candidate是不会响应其他candidate发起的投票的；  
6.2 本轮投票票数没有过半，没有选出leader，重新开始下一轮选举，next term，因为每次刷新后的election timeout都一样的，总会有一个先结束，
然后发起新一轮投票；

####Log Replication 
1、只有leader可以接收来自客户端的请求，当leader收到请求时，会在下一个心跳检测中把这个entry发送给所有follower；
2、一但超过半数的follower响应，leader就确认并提交commit这个新的修改，然后返回给客户端，并在下一个entry告诉所有的follower；

####分区情况下如何保证一致性
1、以5个节点为例，a是leader，a，b，c，d，e被分成了a，b与c，d，e，因为网络分区，
这个时候c，d，e得不到leader的响应，开始执行Leader Election，选出c为leader，进入到了term=2，a和b还是跟从前一样，term=1；  
2、这个时候，有客户端向a,b分区修改数据，set b=3，但是拿不到超过半数的响应，所以迟迟不能提交这个改动；  
3、另一个客户端向另外一个c,d,e分区发起了 set c=8，这个修改成功了，因为有超过半数的响应；  
4、现在网络恢复，a,b,c,d,e又回到同一个网络，两个leader都发出心跳，节点a收到了更高级轮次的leader的心跳，
就变回了follower，回滚他们之前未提交uncommitted的数据，开始同步leader的数据；
