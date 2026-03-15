请设计一个 Key-Value 存储引擎(Design a key-value store)。

这是一道频繁出现的题目，个人认为也是一道很好的题目，这题纵深非常深，内行的人可以讲的非常深。

首先讲两个术语，**数据库**和**存储引擎**。数据库往往是一个比较丰富完整的系统, 提供了 SQL 查询语言，事务和水平扩展等支持。然而存储引擎则是小而精, 纯粹专注于单机的读/写/存储。一般来说, 数据库底层往往会使用某种存储引擎。

目前开源的 KV 存储引擎中，RocksDB 是流行的一个，MongoDB 和 MySQL 底层可以切换成 RocksDB，TiDB 底层直接使用了 RocksDB。大多数分布式数据库的底层不约而同的都选择了 RocksDB。

RocksDB 最初是从 LevelDB 进化而来的，我们先从简单一点的 LevelDB 入手，借鉴它的设计思路。

## LevelDB 整体结构

有一个反直觉的事情是，**内存随机写甚至比硬盘的顺序读还要慢**，磁盘随机写就更慢了，说明我们要避免随机写，最好设计成顺序写。因此好的 KV 存储引擎，都在尽量避免更新操作，把更新和删除操作转化为顺序写操作。LevelDB 采用了一种 SSTable 的数据结构来达到这个目的。

SSTable(Sorted String Table)就是一组按照 key 排序好的 key-value 对, key 和 value 都是字节数组。SSTable 既可以在内存中，也可以在硬盘中。SSTable 底层使用 LSM Tree(Log-Structured Merge Tree)来存放有序的 key-value 对。

LevelDB 整体由如下几个组成部分，

1. MemTable。即内存中的 SSTable，新数据会写入到这里，然后批量写入磁盘，以此提高写的吞吐量。
1. Log 文件。写 MemTable 前会写 Log 文件，即用 WAL(Write Ahead Log)方式记录日志，如果机器突然掉电，内存中的 MemTable 丢失了，还可以通过日志恢复数据。WAL 日志是很多传统数据库例如 MySQL 采用的技术，详细解释可以参考[数据库如何用 WAL 保证事务一致性？ - 知乎专栏](https://zhuanlan.zhihu.com/p/24900322)。
1. Immutable MemTable。内存中的 MemTable 达到指定的大小后，将不再接收新数据，同时会有新的 MemTable 产生，新数据写入到这个新的 MemTable 里，Immutable MemTable 随后会写入硬盘，变成一个 SST 文件。
1. `SSTable`文件。即硬盘上的 SSTable，文件尾部追加了一块索引，记录 key->offset，提高随机读的效率。SST 文件为 Level 0 到 Level N 多层，每一层包含多个 SST 文件；单个 SST 文件容量随层次增加成倍增长；Level0 的 SST 文件由 Immutable MemTable 直接 Dump 产生，其他 Level 的 SST 文件由其上一层的文件和本层文件归并产生。
1. Manifest 文件。Manifest 文件中记录 SST 文件在不同 Level 的分布，单个 SST 文件的最大最小 key，以及其他一些 LevelDB 需要的元信息。
1. Current 文件。从上面的介绍可以看出，LevelDB 启动时的首要任务就是找到当前的 Manifest，而 Manifest 可能有多个。Current 文件简单的记录了当前 Manifest 的文件名。

![](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/book/Andrew-Ng-DeepLearning-AI/leveldb/architecture.png)

LevelDB 的一些核心逻辑如下，

1. 首先 SST 文件尾部的索引要放在内存中，这样读索引就不需要一次磁盘 IO 了
1. 所有读要先查看`MemTable`，如果没有再查看内存中的索引
1. 所有写操作只能写到`MemTable`, 因为 SST 文件不可修改
1. 定期把`Immutable MemTable`写入硬盘，成为`SSTable`文件，同时新建一个`MemTable`会继续接收新来的写操作
1. 定期对`SSTable`文件进行合并
1. 由于硬盘上的`SSTable`文件是不可修改的，那怎么更新和删除数据呢？对于更新操作，追加一个新的 key-value 对到文件尾部，由于读`SSTable`文件是从前向后读的，所以新数据会最先被读到；对于删除操作，追加“墓碑”值(tombstone)，表示删除该 key，在定期合并`SSTable`文件时丢弃这些 key, 即可删除这些 key。

## Manifest 文件

Manifest 文件记录各个 SSTable 各个文件的管理信息，比如该 SST 文件处于哪个 Level，文件名称叫啥，最小 key 和最大 key 各自是多少，如下图所示，

![](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/book/Andrew-Ng-DeepLearning-AI/leveldb/manifest.png)

## Log 文件

Log 文件主要作用是系统发生故障时，能够保证不会丢失数据。因为在数据写入内存中的 MemTable 之前，会先写入 Log 文件，这样即使系统发生故障，MemTable 中的数据没有来得及 Dump 到磁盘，LevelDB 也可以根据 log 文件恢复内存中的 MemTable，不会造成系统丢失数据。这个方式就叫做 WAL(Write Ahead Log)，很多传统数据库例如 MySQL 也使用了 WAL 技术来记录日志。

每个 Log 文件由多个 block 组成，每个 block 大小为 32K，读取和写入以 block 为基本单位。下图所示的 Log 文件包含 3 个 Block，

![](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/book/Andrew-Ng-DeepLearning-AI/leveldb/log.png)

### SSTable

![](https://ngte-superbed.oss-cn-beijing.aliyuncs.com/book/Andrew-Ng-DeepLearning-AI/leveldb/sstable.png)

## MemTable

MemTable 是内存中的数据结构，存储的内容跟硬盘上的 SSTable 一样，只是格式不一样。Immutable MemTable 的内存结构和 Memtable 是完全一样的，区别仅仅在于它是只读的，而 MemTable 则是允许写入和读取的。当 MemTable 写入的数据占用内存到达指定大小，则自动转换为 Immutable Memtable，等待 Dump 到磁盘中，系统会自动生成一个新的 MemTable 供写操作写入新数据，理解了 MemTable，那么 Immutable MemTable 自然不在话下。

MemTable 里的数据是按照 key 有序的，因此当插入新数据时，需要把这个 key-value 对插入到合适的位置上，以保持 key 有序性。MemTable 底层的核心数据结构是一个跳表(Skip List)。跳表是红黑树的一种替代数据结构，具有更高的写入速度，而且实现起来更加简单，请参考[跳表(Skip List)](appendix/skip-list.md)。

前面我们介绍了 LevelDB 的一些内存数据结构和文件，这里开始介绍一些动态操作，例如读取，写入，更新和删除数据，分层合并，错误恢复等操作。

## 添加、更新和删除数据

LevelDB 写入新数据时，具体分为两个步骤：

1. 将这个操作顺序追加到 log 文件末尾。尽管这是一个磁盘操作，但是文件的顺序写入效率还是跟高的，所以不会降低写入的速度
1. 如果 log 文件写入成功，那么将这条 key-value 记录插入到内存中 MemTable。

LevelDB 更新一条记录时，并不会本地修改 SST 文件，而是会作为一条新数据写入 MemTable，随后会写入 SST 文件，在 SST 文件合并过程中，新数据会处于文件尾部，而读取操作是从文件尾部倒着开始读的，所以新值一定会最先被读到。

LevelDB 删除一条记录时，也不会修改 SST 文件，而是用一个特殊值(墓碑值，tombstone)作为 value，将这个 key-value 对追加到 SST 文件尾部，在 SST 文件合并过程中，这种值的 key 都会被忽略掉。

核心思想就是把写操作转换为顺序追加，从而提高了写的效率。

## 读取数据

读操作使用了如下几个手段进行优化：

- MemTable + SkipList
- Binary Search(通过 manifest 文件)
- 页缓存
- bloom filter
- 周期性分层合并

## 分层合并(Leveled Compaction)

### 参考资料

- [SSTable and Log Structured Storage: LevelDB - igvita.com](https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/)
- [数据分析与处理之二（Leveldb 实现原理） - Haippy - 博客园](http://www.cnblogs.com/haippy/archive/2011/12/04/2276064.html)
- [Log Structured Merge Trees(LSM) 原理 - OPEN 开发经验库](http://www.open-open.com/lib/view/open1424916275249.html)
- [从朴素解释出发解释 leveldb 的设计 | ggaaooppeenngg](https://ggaaooppeenngg.github.io/zh-CN/2017/03/31/%E4%BB%8E%E6%9C%B4%E7%B4%A0%E8%A7%A3%E9%87%8A%E5%87%BA%E5%8F%91%E8%A7%A3%E9%87%8Aleveldb%E7%9A%84%E8%AE%BE%E8%AE%A1/index.html)
- [LSM 算法的原理是什么？ - 知乎](https://www.zhihu.com/question/19887265)
- [数据库如何用 WAL 保证事务一致性？ - 知乎专栏](https://zhuanlan.zhihu.com/p/24900322)
- [LevelDB 系列概述 - 360 基础架构组](http://chuansong.me/n/1509342851514)
- [存储引擎技术架构与内幕 (leveldb-1) - GitHub](https://github.com/abbshr/abbshr.github.io/issues/58)
- [leveldb 中的 SSTable (3)](http://bean-li.github.io/leveldb-sstable-bloom-filter/)
