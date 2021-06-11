2021-06-11
### 为什么redisDb 有id？
当 Redis 服务器初始化时， 它会创建出 redis.h/REDIS_DEFAULT_DBNUM 个数据库， 并将所有数据库保存到 redis.h/redisServer.db 数组中， 每个数据库的 id 为从 0 到 REDIS_DEFAULT_DBNUM - 1 的值。当执行 SELECT number 命令时，程序直接使用 redisServer.db[number] 来切换数据库。id就是这个数据的下标。 AOF 程序、复制程序和 RDB 程序， 需要知道当前数据库的号码， 如果没有 id 域的话， 程序就只能在当前使用的数据库的指针， 和 redisServer.db 数组中所有数据库的指针进行对比， 以此来弄清楚自己正在使用的是那个数据库。
### 如何内存回收？
redis使用的c编写，是没有自动回收机制的，redis使用的对象引用计数法，当引用计数为0是，释放内存。