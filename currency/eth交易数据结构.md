#### 1 普通转账eth
```$xslt
{
    "id": 83,
    "jsonrpc": "2.0",
    "result": {
        "blockHash": "0xc808382e98ebca97c2910a1e3781cdc3f4d9d86f2b9fa555f06ae94886297484",
        "blockNumber": "0x11f8",
        "from": "0x526e12e667b264c7264385e2d3d0439cc8f87735",
        "gas": "0x18a88",
        "gasPrice": "0x430e23400",
        "hash": "0x3551bc4b2237fa7291ed218a0f9e130f7183c488c49ad634fbd9666b905b4ba4",
        "input": "0x",
        "nonce": "0x5",
        "r": "0xc3bb9f1fd1f9071f537713ae6fa76a65a1cbbfd86692f778c39094b9bc4f7802",
        "s": "0x713e033975548a3c506f570e8e9d5ed4d3399b642745a41320e67f2fe8d7d1df",
        "to": "0xd87063c7074bc1a4689197a9222be197f7d39d85",
        "transactionIndex": "0x0",
        "v": "0x43721",
        "value": "0xf971e5914ac8000"
    }
}
```
#### 2 token转账
```$xslt
{
    "id": 83,
    "jsonrpc": "2.0",
    "result": {
        "blockHash": "0x82a35c92a01b6a0cd5f8b2cdec4a2275a2e9efa9507fc551e0735c2c9c6d885a",
        "blockNumber": "0x8bb021",
        "from": "0xac8b30bfca3ce01bba9312571c15f4c6c8fd3a82",
        "gas": "0x34bc0",
        "gasPrice": "0x300e66100",
        "hash": "0x531b93ab553930a9566585a6262b992d1fc0c0c6a78a68d5ccfe601bff8fd341",
        "input": "0xa9059cbb0000000000000000000000006a8b1e24967edeab380ca44f3413e0e865c0a2090000000000000000000000000000000000000000000000000000041568dae400",
        "nonce": "0x3f9a",
        "r": "0x765a1ad0c5b9afc4d5dbb677429c6a22999b449670992fb7b86def98feb1f0dc",
        "s": "0x66c77f20a2b847bd93325d621f3e98c06e11958fb2976c5220d370ee5c265233",
        "to": "0xac08809df1048b82959d6251fbc9538920bed1fa",
        "transactionIndex": "0x45",
        "v": "0x1b",
        "value": "0x0"
    }
}
```
input包含了transfer toaddr amount，但是缺乏状态,  
交易成功与否需要查看收据receipt  
status=0x1表示成功
```$xslt
{
    "id": 83,
    "jsonrpc": "2.0",
    "result": {
        "blockHash": "0x82a35c92a01b6a0cd5f8b2cdec4a2275a2e9efa9507fc551e0735c2c9c6d885a",
        "blockNumber": "0x8bb021",
        "contractAddress": null,
        "cumulativeGasUsed": "0x27b48e",
        "from": "0xac8b30bfca3ce01bba9312571c15f4c6c8fd3a82",
        "gasUsed": "0x9f8a",
        "logs": [
            {
                "address": "0xac08809df1048b82959d6251fbc9538920bed1fa",
                "blockHash": "0x82a35c92a01b6a0cd5f8b2cdec4a2275a2e9efa9507fc551e0735c2c9c6d885a",
                "blockNumber": "0x8bb021",
                "data": "0x0000000000000000000000000000000000000000000000000000041568dae400",
                "logIndex": "0x29",
                "removed": false,
                "topics": [
                    "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
                    "0x000000000000000000000000ac8b30bfca3ce01bba9312571c15f4c6c8fd3a82",
                    "0x0000000000000000000000006a8b1e24967edeab380ca44f3413e0e865c0a209"
                ],
                "transactionHash": "0x531b93ab553930a9566585a6262b992d1fc0c0c6a78a68d5ccfe601bff8fd341",
                "transactionIndex": "0x45"
            }
        ],
        "logsBloom": "0x00000000000000000000000400000000000000010000000000000000008000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000008000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000010400000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000",
        "status": "0x1",
        "to": "0xac08809df1048b82959d6251fbc9538920bed1fa",
        "transactionHash": "0x531b93ab553930a9566585a6262b992d1fc0c0c6a78a68d5ccfe601bff8fd341",
        "transactionIndex": "0x45"
    }
}
```
以标准的erc20合约转账为例  
log.data 是金额  
topics[2]为接收地址  