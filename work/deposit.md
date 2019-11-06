### 1 wallet-deposit
#### 1 实现rpc.go接口
```
NextBlock(handleRollback HandleRollbackBlock) (*Block, error)
	GetTxs(hashes []string) ([]*models.Tx, error)
	GetTxConfirmations(hash string) (uint64, error)
	ReuseAddress() bool
```
##### 1.1 NextBlock
该方法的主要作用是同步区块链高度，保存当前已检测的最新高度，检查每个区块下，是否有转账到自己地址的交易，
如果有，保存该交易的交易信息，包括金额，交易hash等；
对于账户模型，则先需要初始化账号信息，对账号list循环检查新增的交易流水，然后保存交易信息；

##### 1.2 GetTxs
根据入参，交易hash数组，链上查询该笔交易是否是转账到自己地址的交易，如果有，按照上面的逻辑处理，
用于处理一些没有正常上账，或者漏掉的交易；

##### 1.3 GetTxConfirmations
根据hash获取该交易的信息，并计算该交易的确认数，确认数=当前高度-交易所在高度+1；

##### 1.4 ReuseAddress
是否重用地址，如果是返回ture即可，大部分是返回true，一些一次性地址的 ***//TODO***

#### 2 代码规范
##### 2.1 命名规范
名称浅显易懂，见名知义；
##### 2.2 空行原则
函数里面的分段，将一小块逻辑分开，让代码易于阅读；
##### 2.3 error处理
有error必须处理，必须返回，中断此次遍历；
##### 2.4 代码优化
2.4.1 可以使用一个变量重复使用，就不需要多个；
2.4.2 逻辑处理if中，先处理不满足条件的，这样可以直接返回，如：
```
if isNotwork{
return
}
//code body
```
而不是：
```$xslt
if isWork {
    //code body
}
```
2.4.3 日志记录，对于error，非最外层加上简易说明，或者可以直接返回上层，在最外层一定要记得log输出；
info，和debug根据需要直接输出即可；
2.4.4 抽取共用代码段，杜绝代码冗余；
2.4.5 空指针！！！要在不为空的情况下才进行相关操作
#### 2 handler

### 2 wallet-deposit-sandbox-config
#### 2.1 增加币种的配置文件
#### 2.2 增加nodeapi的配置