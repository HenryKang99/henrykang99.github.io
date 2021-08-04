## Overview

　　Redis（Remote Dictionary Server 远程字典服务） 是一个由 C 语言编写的、开源的、可持久化的、key-value 形式的 **NoSQL** 数据库。支持的常用数据类型包含：**String，Hash，List，Set，SortedSet**。主要应用场景：缓存热门内容、排行榜、在线好友列表、任务队列、网站访问统计、数据过期处理、分布式集群架构中 session 的分离等。

　　NoSQL（Not Only SQL）意为不仅仅是SQL，泛指非关系型数据库。按照其存储数据的格式可以分为：**键值对存储、列存储、文档型存储、图形存储**。

**了解：**

- [Redis for Windows 下载](https://github.com/tporadowski/redis/releases)
- 使用 `help @<group>` 可以分类查看 API 信息。
- Redis 默认划分了 16 个 database，默认使用的是第 0 个，使用 `select` 命令切换。
  - 使用 `FLUSHALL` 命令可以删除所有库的所有数据。
- Redis 是**二进制安全**的，客户端需要协商好编码方式。
- 取中文加上 `--raw`。
- Redis 中的核心对象称为**RedisObject**，[参考这里](https://redisbook.readthedocs.io/en/latest/datatype/object.html)。主要关注类型 Type 和编码方式 Encoding，Type 记录了对应 Value 的类型，Encoding 记录了编码。

```c
// 可能已经过时了，参考一下即可。
/*
 * Redis 对象
 */
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 对齐位
    unsigned notused:2;

    // 编码方式
    unsigned encoding:4;

    // LRU 时间（相对于 server.lruclock）
    unsigned lru:22;

    // 引用计数
    int refcount;

    // 指向对象的值
    void *ptr;

} robj;

/*
 * 对象类型
 */
#define REDIS_STRING 0  // 字符串
#define REDIS_LIST 1    // 列表
#define REDIS_SET 2     // 集合
#define REDIS_ZSET 3    // 有序集
#define REDIS_HASH 4    // 哈希表
......

/*
 * 对象编码
 */
#define REDIS_ENCODING_RAW 0            // 编码为字符串
#define REDIS_ENCODING_INT 1            // 编码为整数
#define REDIS_ENCODING_HT 2             // 编码为哈希表
#define REDIS_ENCODING_ZIPMAP 3         // 编码为 zipmap
#define REDIS_ENCODING_LINKEDLIST 4     // 编码为双端链表
#define REDIS_ENCODING_ZIPLIST 5        // 编码为压缩列表
#define REDIS_ENCODING_INTSET 6         // 编码为整数集合
#define REDIS_ENCODING_SKIPLIST 7       // 编码为跳跃表
```

​    

## 数据类型

> 　　常用的有五种，分别是：**String，Hash，List，Set，SortedSet**。除此之外还有：HyperLogLog（2.8.9），GEO（3.2），Stream（5.0），下面只记录常用的五种。



### String

### List

### Set

### Hash

### SortedSet



## 消息



## 事务



## 主从



## 持久化

> 将主存中的数据按照配置持久化到硬盘中，redis 持久化提供了RDB和AOF两种机制。

1. RDB机制（default）：在一定的时间内监测key的变化，按照制定好的规则进行持久化操作，粒度较大。

   - 在`redis.windows.conf`文件中进行配置：
   - **注意**：重启服务时要指定配置文件名称；

```mysql
#   after 900 sec (15 min) if at least 1 key changed
save 900 1
#   after 300 sec (5 min) if at least 10 keys changed
save 300 10
#   after 60 sec if at least 10000 keys changed
save 60 10000
```



2. AOF机制：用日志记录每一条操作命令，粒度小。
   - 在`redis.windows.conf`文件中进行配置：

```mysql
appendonly no 改为 yes 开启
# appendfsync always ： 每一次操作都进行持久化
appendfsync everysec ： 每隔一秒进行一次持久化
# appendfsync no	 ： 不进行持久化
```



## BIO、NIO、AIO