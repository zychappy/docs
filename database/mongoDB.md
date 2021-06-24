# 概念
|SQL|MongoDB|说明|
|:-----|:-----|:-----|
database|database|数据库
table|collection|数据库表/集合
row|document|数据记录行/文档
column|field|数据字段/域
index|index|索引
table joins	| 	|表连接,MongoDB不支持
primary key|primary key|主键,MongoDB自动将_id字段设置为主键
# 数据类型
## ObjectId
ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes，含义是：

前 4 个字节表示创建 unix 时间戳,格林尼治时间 UTC 时间，比北京时间晚了 8 个小时
接下来的 3 个字节是机器标识码
紧接的两个字节由进程 id 组成 PID
最后三个字节是随机数

MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型的，默认是个 ObjectId 对象

由于 ObjectId 中保存了创建的时间戳，所以你不需要为你的文档保存时间戳字段，你可以通过 getTimestamp 函数来获取文档的创建时间
## 字符串
BSON 字符串都是 UTF-8 编码。

# 分布式高可用数据库的几个关键问题
## 复制与分片
