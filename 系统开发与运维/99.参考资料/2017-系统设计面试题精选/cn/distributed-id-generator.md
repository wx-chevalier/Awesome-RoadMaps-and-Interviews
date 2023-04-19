如何设计一个分布式 ID 生成器(Distributed ID Generator)，并保证 ID 按时间粗略有序？

### 应用场景(Scenario)

现实中很多业务都有**生成唯一 ID**的需求，例如：

- 用户 ID
- 微博 ID
- 聊天消息 ID
- 帖子 ID
- 订单 ID

### 需求(Needs)

这个 ID 往往会作为数据库主键，所以需要保证**全局唯一**。数据库会在这个字段上建立聚集索引(Clustered Index，参考 MySQL InnoDB)，即该字段会影响各条数据再物理存储上的顺序。

ID 还要尽可能**短**，节省内存，让数据库索引效率更高。基本上 64 位整数能够满足绝大多数的场景，但是如果能做到比 64 位更短那就更好了。需要根据具体业务进行分析，预估出 ID 的最大值，这个最大值通常比 64 位整数的上限小很多，于是我们可以用更少的 bit 表示这个 ID。

查询的时候，往往有分页或者**排序**的需求，所以需要给每条数据添加一个时间字段，并在其上建立普通索引(Secondary Index)。但是普通索引的访问效率比聚集索引慢，如果能够让 ID**按照时间粗略有序**，则可以省去这个时间字段。为什么不是按照时间精确有序呢？因为按照时间精确有序是做不到的，除非用一个单机算法，在分布式场景下做到精确有序性能一般很差。

这就引出了 ID 生成的三大核心需求：

- 全局唯一(unique)
- 按照时间粗略有序(sortable by time)
- 尽可能短

下面介绍一些常用的生成 ID 的方法。

### UUID

用过 MongoDB 的人会知道，MongoDB 会自动给每一条数据赋予一个唯一的[ObjectId](https://docs.mongodb.com/manual/reference/method/ObjectId/),保证不会重复，这是怎么做到的呢？实际上它用的是一种 UUID 算法，生成的 ObjectId 占 12 个字节，由以下几个部分组成，

- 4 个字节表示的 Unix timestamp,
- 3 个字节表示的机器的 ID
- 2 个字节表示的进程 ID
- 3 个字节表示的计数器

[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)是一类算法的统称，具体有不同的实现。UUID 的有点是每台机器可以独立产生 ID，理论上保证不会重复，所以天然是分布式的，缺点是生成的 ID 太长，不仅占用内存，而且索引查询效率低。

### 多台 MySQL 服务器

既然 MySQL 可以产生自增 ID，那么用多台 MySQL 服务器，能否组成一个高性能的分布式发号器呢？ 显然可以。

假设用 8 台 MySQL 服务器协同工作，第一台 MySQL 初始值是 1，每次自增 8，第二台 MySQL 初始值是 2，每次自增 8，依次类推。前面用一个 round-robin load balancer 挡着，每来一个请求，由 round-robin balancer 随机地将请求发给 8 台 MySQL 中的任意一个，然后返回一个 ID。

[Flickr 就是这么做的](http://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)，仅仅使用了两台 MySQL 服务器。可见这个方法虽然简单无脑，但是性能足够好。不过要注意，在 MySQL 中，不需要把所有 ID 都存下来，每台机器只需要存一个 MAX_ID 就可以了。这需要用到 MySQL 的一个[REPLACE INTO](http://dev.mysql.com/doc/refman/5.0/en/replace.html)特性。

这个方法跟单台数据库比，缺点是**ID 是不是严格递增的**，只是粗略递增的。不过这个问题不大，我们的目标是粗略有序，不需要严格递增。

### Twitter Snowflake

比如 Twitter 有个成熟的开源项目，就是专门生成 ID 的，[Twitter Snowflake](https://github.com/twitter/snowflake) 。Snowflake 的核心算法如下：

![](http://121.40.136.3/wp-content/uploads/2015/04/snowflake-64bit.jpg)

最高位不用，永远为 0，其余三组 bit 占位均可浮动，看具体的业务需求而定。默认情况下 41bit 的时间戳可以支持该算法使用到 2082 年，10bit 的工作机器 id 可以支持 1023 台机器，序列号支持 1 毫秒产生 4095 个自增序列 id。

[Instagram 用了类似的方案](https://engineering.instagram.com/sharding-ids-at-instagram-1cf5a71e5a5c)，41 位表示时间戳，13 位表示 shard Id(一个 shard Id 对应一台 PostgreSQL 机器),最低 10 位表示自增 ID，怎么样，跟 Snowflake 的设计非常类似吧。这个方案用一个 PostgreSQL 集群代替了 Twitter Snowflake 集群，优点是利用了现成的 PostgreSQL，容易懂，维护方便。

有的面试官会问，如何让 ID 可以粗略的按照时间排序？上面的这种格式的 ID，含有时间戳，且在高位，恰好满足要求。如果面试官又问，如何保证 ID 严格有序呢？在分布式这个场景下，是做不到的，要想高性能，只能做到粗略有序，无法保证严格有序。

### 参考资料

- [Sharding & IDs at Instagram](https://engineering.instagram.com/sharding-ids-at-instagram-1cf5a71e5a5c)
- [Ticket Servers: Distributed Unique Primary Keys on the Cheap](http://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)
- [Twitter Snowflake](https://github.com/twitter/snowflake)
- [细聊分布式 ID 生成方法 - 沈剑](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=403837240&idx=1&sn=ae9f2bf0cc5b0f68f9a2213485313127&scene=21)
- [服务化框架－分布式 Unique ID 的生成方法一览 - 江南白衣](http://calvin1978.blogcn.com/articles/uuid.html)
- [生成全局唯一 ID 的 3 个思路，来自一个资深架构师的总结](https://mp.weixin.qq.com/s?__biz=MzI4MTY5NTk4Ng==&mid=2247489561&idx=1&sn=7396f373af4efa62ba4dbecc6d7f83b3&source=41)
