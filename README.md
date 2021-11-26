# Fork说明

RedisLV是2015年创建，在18-19年断更的缓存+数据库的卡牌游戏缓存解决方案。

它利用Redis作为缓存，LevelDB落地。二者都在内存中进行，同时实现快速缓存+落地。

数据同步逻辑是，监听redis aof写入行为，将操作写入到LevelDB。

但是由于Redis数据都在内存中，并且没有考虑集群、迁移和LevelDB SST文件合并导致瞬间磁盘IO压力等问题导致这个方案并不可靠。

所以在互联网公司应用不多。

但是在游戏中，分服卡牌手游要求的特点，他都能满足，所以基本可以作为单服存储方案。比如少年三国志二、三。

在类似的技术方案上。360在此基础上开发了pika（redis+rocksdb）。另外还有ssdb（redis+leveldb）。相对来说，pika是更好的选择。

在pika、codis、redis基础上，喜马拉雅有xcache可以做冷热数据分离

[pika](https://github.com/OpenAtomFoundation/pika/blob/master/README_CN.md)

[ssdb](https://github.com/ideawu/ssdb)

[xcache](https://github.com/XimalayaCloud/xcache)

---

## 对原项目的使用做出说明：

安装：找到Makefile位置，make

redis-server、redis-cli可执行文件在src下

修改配置文件

在同路径下找到redis.conf，备份，修改如下：
```
port 7272 # redis-server启动端口
bind 0.0.0.0 # 改成允许远程访问
databases 255 # 默认数据库数量16太少
```

其他配置说明：
```
daemonize yes # 默认后台启动
dir ./ # 工作路径
leveldb yes # 存储引擎：是为leveldb，否为redis
leveldb-path ./var # leveldb数据存储路径
```

其他配置根据使用情况调整

启动：
```
RedisLV/src/redis-server RedisLV/redis.conf
```

测试：
```
RedisLV/src/redis-cli -h 127.0.0.1 -p 7272
```

--- 
[RedisLV-支持基于LevelDB的Redis持久化方法](https://github.com/ivanabc/RedisLV)
---

### Redis持久化的问题
1. RDB方式: 数据持久化的过程中可能存在大量额外内存消耗。
2. AOF方式: 通过aof文件恢复数据库的过程慢。

### RedisLV优点
1. 对于内存有限的服务，数据持久化不会带来额外的内存消耗。
2. 相对AOF方式，数据库的恢复更快。

### RedisLV缺点
1. 由于对redis写入操作需要同步到leveldb，导致性能损耗(读操作不受影响)。

### RedisLV备份
```
redis-cli backup dir(备份文件目录)
```
* 当备份目录中包含BACKUP.log文件并且文件中有SUCCESS字段，表示备份成功

### Redis命令支持状况(yes: 支持; no: 不支持), 当redis使用leveldb引擎时，命令支持状况(yes: 支持; no: 不支持)

| Key         |  redis_in_ldb  | redis_no_ldb |
|-------------|----------------| -------------|
| DEL         |       yes      |      yes     |
| DUMP        |       yes      |      yes     |
| EXISTS      |       yes      |      yes     |
| EXPIRE      |       no       |      yes     |
| EXPIREAT    |       no       |      yes     |
| KEYS        |       yes      |      yes     |
| MIGRATE     |       no       |      yes     |
| MOVE        |       no       |      yes     |
| OBJECT      |       yes      |      yes     |
| PERSIST     |       no       |      yes     |
| PEXPIRE     |       no       |      yes     |
| PEXPIREAT   |       no       |      yes     |
| PTTL        |       no       |      yes     |
| RANDOMKEY   |       yes      |      yes     |
| RENAME      |       no       |      yes     |
| RENAMENX    |       no       |      yes     |
| RESTORE     |       no       |      yes     |
| SORT        |  yes(not store)|      yes     |
| TTL         |       no       |      yes     |
| TYPE        |       yes      |      yes     |
| SCAN        |       yes      |      yes     |

---

| String      |  redis_in_ldb  | redis_no_ldb |
|-------------|----------------|--------------|
| APPEND      |       yes      |      yes     |
| BITCOUNT    |       yes      |      yes     |
| BITOP       |       yes      |      yes     |
| DECR        |       yes      |      yes     |
| DECRBY      |       yes      |      yes     |
| GET	      |       yes      |      yes     |
| GETBIT      |       yes      |      yes     |
| GETRANGE    |       yes      |      yes     |
| GETSET      |       yes      |      yes     |
| INCR        |       yes      |      yes     |
| INCRBY      |       yes      |      yes     |
| INCRBYFLOAT |       yes      |      yes     |
| MGET        |       yes      |      yes     |
| MSET        |       yes      |      yes     |
| MSETNX      |       yes      |      yes     |
| PSETEX      |       no       |      yes     |
| SET         |       yes      |      yes     |
| SETBIT      |       yes      |      yes     |
| SETEX       |       no       |      yes     |
| SETNX       |       yes      |      yes     |
| SETRANGE    |       yes      |      yes     |
| STRLEN      |       yes      |      yes     |

---

| Hash        |   redis_in_ldb | redis_no_ldb |
|-------------|----------------|--------------|
| HDEL        |       yes      |      yes     | 
| HEXISTS     |       yes      |      yes     |
| HGET        |       yes      |      yes     |
| HGETALL     |       yes      |      yes     |
| HINCRBY     |       yes      |      yes     |
| HINCRBYFLOAT|       yes      |      yes     |
| HKEYS       |       yes      |      yes     |
| HLEN        |       yes      |      yes     |
| HMGET       |       yes      |      yes     |
| HMSET       |       yes      |      yes     |
| HSET        |       yes      |      yes     |
| HSETNX      |       yes      |      yes     |
| HVALS       |       yes      |      yes     |
| HSCAN       |       yes      |      yes     |

---

| Set         |   redis_in_ldb | redis_no_ldb |
|-------------|----------------|--------------|
| SADD        |       yes      |      yes     |
| SCARD       |       yes      |      yes     |
| SDIFF       |       yes      |      yes     |
| SDIFFSTORE  |       no       |      yes     |
| SINTER      |       yes      |      yes     |
| SINTERSTORE |       no       |      yes     |
| SISMEMBERS  |       yes      |      yes     |
| SMEMBERS    |       yes      |      yes     |
| SMOVE       |       no       |      yes     |
| SPOP        |       no       |      yes     |
| SRANDMEMBER |       yes      |      yes     |
| SREM        |       yes      |      yes     |
| SUNION      |       yes      |      yes     |
| SUNIONSTORE |       no       |      yes     |
| SSCAN       |       yes      |      yes     |

---

| SortedSet       | redis_in_ldb | redis_no_ldb |
|-----------------|--------------|--------------|
| ZADD            |       yes    |      yes     |
| ZCARD           |       yes    |      yes     |
| ZCOUNT          |       yes    |      yes     |
| ZINCRBY         |       yes    |      yes     |
| ZRANGE          |       yes    |      yes     |
| ZRANGEBYSCORE   |       yes    |      yes     |
| ZRANK           |       yes    |      yes     |
| ZREM            |       yes    |      yes     |
| ZREMRANGEBYRANK |       yes    |      yes     |  
| ZREMRANGEBYSCORE|       yes    |      yes     |
| ZREVRANGE       |       yes    |      yes     |
| ZREVRANKBYSCORE |       yes    |      yes     |
| ZREVRANK        |       yes    |      yes     |
| ZSCORE          |       yes    |      yes     |
| ZUNIONSTORE     |       no     |      yes     |
| ZINTERSTORE     |       no     |      yes     |
| ZSCAN           |       yes    |      yes     |
| ZRANGEBYLEX     |       yes    |      yes     |
| ZLEXCOUNT       |       yes    |      yes     |
| ZREMRANGEBYLEX  |       yes    |      yes     |  
