## leveldb

+ 写性能十分优秀，基于`LSM(Log Structured-Merge Tree)`树实现，`LSM`树的核心思想就是放弃部分读的性能，换取最大的写入能力。
+ `LSM`树写性能极高的原理，简单地来说就是尽量减少随机写的次数。对于每次写入操作，并不是直接将最新的数据驻留在磁盘中，而是将其拆分成两步：
  + 一次日志文件的顺序写
  + 一次内存中的数据插入
+ `leveldb`正是实践了这种思想，将数据首先更新在内存中，当内存中的数据达到一定的阈值，将这部分数据真正刷新到磁盘文件中，因而获得了极高的写性能。

### 整体架构

![](https://ws1.sinaimg.cn/large/77451733gy1g5uk6wp96mj20sg0lcgtp.jpg)

+ `memtable`

  + `leveldb`的一次写入操作并不是直接将数据刷新到磁盘文件，而是首先写入到内存中作为代替，`memtable`就是一个在内存中进行数据组织与维护的结构。
  + `memtable`中，所有的数据按用户定义的排序方法排序之后**按序存储**，等到其存储内容的容量达到阈值时（默认为`4MB`），便将其转换成`immutable memtable`，与此同时创建一个新的`memtable`，供用户继续进行读写操作。
  + `memtable`底层使用跳表。

+ `immutable memtable`

  + 与`memtable`一样，只是`immutable memtable`是只读的。当一个`immutable memtable`被创建时，`leveldb`的后台压缩进程便会将利用其中的内容，创建一个`sstable`，持久化到磁盘文件中。

+ `log`

  + `leveldb`在写内存之前会首先将所有的写操作写到日志文件中，也就是`log`文件
  + 异常情况发生时，均可以通过日志文件进行恢复`(redo/undo)`
  + 每次日志的写操作都是一次顺序写，因此写效率高，整体写入性能较好
  + leveldb的**用户写操作的原子性**同样通过日志来实现。

+ `sstable`

  + 内存中的数据达到一定容量，就需要将数据持久化到磁盘的`sstable`中。
  + `leveldb`后台会定期整合这些`sstable`文件，该过程也称为`compaction`。
  + 随着`compaction`的进行，`sstable`文件在逻辑上被分成若干层，由内存数据直接`dump`出来的文件称为`level 0`层文件，后期整合而成的文件为`level i` 层文件
  + 所有的`sstable`文件本身的内容是**不可修改**的

+ `manifest`

  + `leveldb`中有个版本的概念，一个版本中主要记录了每一层中所有文件的元数据，包括

    + 文件大小
    + 最大`key`值
    + 最小`key`值

  + 该版本信息十分关键，除了在查找数据时，利用维护的每个文件的最大/最小`key`值来加快查找，还在其中维护了一些进行`compaction`的统计值，来控制`compaction`的进行。

  + 当每次`compaction`完成（或者换一种更容易理解的说法，当每次`sstable`文件有新增或者减少），`leveldb`都会创建一个新的`version`，创建规则如下：

    ```bash
    versionNew = versionOld + versionEdit
    ```

    + `versionEdit`指代的是基于旧版本的基础上，变化的内容（例如新增或删除了某些`sstable`文件）。

  + `manifest`文件就是用来记录这些`versionEdit`信息的。一个`versionEdit`数据，会被编码成一条记录，写入`manifest`文件中。

  + 通过这些信息，`leveldb`便可以在启动时，基于一个空的`version`，不断`apply`这些记录，最终得到一个上次运行结束时的版本信息。

## Bigtable

### 基本概念

+ 用于管理**结构化数据**的分布式存储系统

+ 其实就是一个稀疏的、分布式的、多维持久有序哈希。

  ```bash
  # key -> value
  (row:string(最大64KB), column:string, time:int64) -> string
  ```

  + 对同一个行关键字的读或者写操作都是原子的（不管读或者写这一行里多少个不同列）
  + 列关键字组成的集合叫做列族(`Column Families`)，列族是访问控制的基本单位。
  + 

+ 行关键字

  + `Bigtable` 通过行关键字的字典顺序来组织数据。表中的每个行都可以动态分区。每个分区叫做一个`Tablet`，
    `Tablet` 是数据分布和负载均衡调整的最小单位。

    ![](https://ws1.sinaimg.cn/large/77451733gy1g5umbrx0wwj20rg06f0tl.jpg)

    + 举例来说，在 `Webtable` 里，通过反转 URL 中主机名的方式，可以把同一个域名下的网页聚集起来组织成连续的行。具体来说，我们可以把 `maps.google.com/index.html`的数据存放在关键字 `com.google.maps/index.html` 下。把相同的域中的网页存储在连续的区域可以让基于主机和域名的分析更加有效。

+ 列关键字

  + 列族在使用之前必须先创建，然后才能在列族中任何的列关键字下存放数据；列族创建后，其中的任何一个列关键字下都可以存放数据。根据我们的设计意图，一张表中的列族不能太多（最多几百个），并且列族在运行期间很少改变。与之相对应的，一张表可以有无限多个列。
  + 访问控制、磁盘和内存的使用统计都是在列族层面进行的。

+ 时间戳

  + 在 `Bigtable` 中，表的每一个数据项都可以包含同一份数据的不同版本；不同版本的数据通过时间戳来索引。
  + `Bigtable` 时间戳的类型是 64 位整型。`Bigtable` 可以给时间戳赋值，用来表示精确到毫秒的**实时**时间；用户程序也可以给时间戳赋值。

+ `BigTable` 内部存储数据的文件是 `Google SSTable` 格式的。`SSTable` 是一个持久化的、排序的、不可更改的
  `Map` 结构，而 `Map` 是一个 `key-value` 映射的数据结构，`key` 和 `value` 的值都是任意的 `Byte` 串。

  + 可以对 `SSTable`进行如下的操作：查询与一个 `key` 值相关的 `value`，或者遍历某个 `key` 值范围内的所有的 `key-value` 对。从内部看，`SSTable` 是一系列的数据块（通常每个块的大小是 `64KB`，这个大小是可以配置的）。`SSTable` 使用块索引（通常存储在 `SSTable` 的最后）来定位数据块；在打开 `SSTable` 的时候，索引被加载到内存。每次查找都可以通过一次磁盘搜索完成：首先使用二分查找法在内存中的索引里找到数据块的位置，然后再从硬盘读取相应的数据块。也可以选择把整个 `SSTable` 都放在内存中，这样就不必访问硬盘了。

+ `BigTable` 还依赖一个高可用的、序列化的分布式锁服务组件，叫做 `Chubby`。

### 架构

+ `Bigtable` 包括了三个主要的组件：
  + 链接到客户程序中的库
  + 一个 `Master` 服务器
    + 为 `Tablet` 服务器分配 `Tablets`、检测新加入的或者过期失效的 `Table` 服务器、对 `Tablet` 服务器进行负载均衡、以及对保存在 `GFS` 上的文件进行垃圾收集。除此之外，它还处理对模式的相关修改操作，例如建立表和列族
  + 多个 `Tablet` 服务器
    + 每个 `Tablet` 服务器都管理一个 `Tablet` 的集合（通常每个服务器有大约数十个至上千个 `Tablet`）。每个 `Tablet`服务器负责处理它所加载的 `Tablet` 的读写操作，以及在 `Tablets` 过大时，对其进行分割
+ 客户端读取的数据都不经过 `Master` 服务器：客户程序直接和 `Tablet` 服务器通信进行读写操作。
+ 一个 `BigTable` 集群存储了很多表，每个表包含了一个 `Tablet` 的集合，而每个 `Tablet` 包含了某个范围内的行的所有相关数据。初始状态下，一个表只有一个 `Tablet`。随着表中数据的增长，它被自动分割成多个 `Tablet`，缺省情况下，每个 `Tablet` 的尺寸大约是 `100MB` 到 `200MB`

### tablet的位置

+  `Tablet`的位置

  ![](https://ws1.sinaimg.cn/large/77451733gy1g5unnfjih5j20gc08owfd.jpg)
  + 第一层是一个存储在 `Chubby` 中的文件，它包含了 `Root Tablet` 的位置信息。`Root Tablet` 包含了一个特殊的 `METADATA` 表里所有的 `Tablet` 的位置信息。
  + `METADATA` 表的每个 `Tablet` 包含了一个用户 `Tablet` 的集合。`Root Tablet` 实际上是 `METADATA` 表的第一个 `Tablet`，只不过对它的处理比较特殊：`Root Tablet` 永远不会被分割，这就保证了 `Tablet` 的位置信息存储结构**不会超过三层**。
  + 在 `METADATA` 表里面，每个 `Tablet` 的位置信息都存放在一个行关键字下面，而这个行关键字是由 `Tablet` 所在的表的标识符和 `Tablet` 的最后一行编码而成的。

### tablet的分配

+ `Tablet`的分配
  + 在任何一个时刻，一个`Tablet`只能分配给一个`Tablet`服务器。`Master`服务器记录了当前有哪些活跃的`Tablet`服务器、哪些 `Tablet` 分配给了哪些 `Tablet` 服务器、哪些 `Tablet` 还没有被分配。当一个 `Tablet` 还没有被分配、并且刚好有一个 `Tablet` 服务器有足够的空闲空间装载该 `Tablet` 时，`Master` 服务器会给这个 `Tablet` 服务器发送一个装载请求，把 `Tablet` 分配给这个服务器。
  + `BigTable` 使用 `Chubby` 跟踪记录 Tablet 服务器的状态。当一个 `Tablet` 服务器启动时，它在 `Chubby` 的一个指定目录下建立一个有唯一性名字的文件，并且获取该文件的独占锁。`Master` 服务器实时监控着这个目录（服务器目录），因此 `Master` 服务器能够知道有新的 `Tablet` 服务器加入了。如果 `Tablet` 服务器丢失了 `Chubby` 上的独占锁 — 比如由于网络断开导致 `Tablet` 服务器和 `Chubby` 的会话丢失 — 它就停止对 `Tablet` 提供服务。（`Chubby` 提供了一种高效的机制，利用这种机制，`Tablet` 服务器能够在不增加网络负担的情况下知道它是否还持有锁）
  + `Master` 服务器负责检查一个 `Tablet` 服务器是否已经不再为它的 `Tablet` 提供服务了，并且要尽快重新分配它加载的 Tablet。
    + `Master` 服务器通过轮询 `Tablet` 服务器文件锁的状态来检测何时 `Tablet` 服务器不再为 Tablet提供服务。
    + 如果一个 `Tablet` 服务器报告它丢失了文件锁，或者 `Master` 服务器最近几次尝试和它通信都没有得到响应，`Master` 服务器就会尝试获取该 `Tablet` 服务器文件的独占锁；
    + 如果 `Master` 服务器成功获取了独占锁，那么就说明 `Chubby` 是正常运行的，而 `Tablet` 服务器要么是宕机了、要么是不能和 `Chubby` 通信了，因此，`Master`服务器就删除该 `Tablet` 服务器在 `Chubby` 上的服务器文件以确保它不再给 `Tablet` 提供服务。一旦 `Tablet` 服务器在 `Chubby` 上的服务器文件被删除了，`Master` 服务器就把之前分配给它的所有的 `Tablet` 放入未分配的 `Tablet`集合中。为了确保 `Bigtable` 集群在 `Master` 服务器和 `Chubby` 之间网络出现故障的时候仍然可以使用，`Master`服务器在它的 `Chubby` 会话过期后主动退出。但是不管怎样，如同我们前面所描述的，`Master` 服务器的故障不会改变现有 `Tablet` 在 `Tablet` 服务器上的分配状态。
  + 当集群管理系统启动了一个 `Master` 服务器之后，`Master` 服务器首先要了解当前 `Tablet` 的分配状态，之后才能够修改分配状态。`Master` 服务器在启动的时候执行以下步骤：
    + `Master` 服务器从 `Chubby` 获取一个唯一的 `Master` 锁，用来阻止创建其它的 `Master` 服务器实例；
    + `Master` 服务器扫描 `Chubby` 的服务器文件锁存储目录，获取当前正在运行的服务器列表；
    + `Master` 服务器和所有的正在运行的 `Tablet` 表服务器通信，获取每个 `Tablet` 服务器上 `Tablet` 的分配信息；
    + `Master` 服务器扫描 `METADATA` 表获取所有的 `Tablet` 的集合。
  + 在扫描的过程中，当 `Master` 服务器发现了一个还没有分配的 `Tablet`，`Master` 服务器就将这个 `Tablet` 加入未分配的 `Tablet` 集合等待合适的时机分配。
  + 保存现有 `Tablet` 的集合只有在以下事件发生时才会改变：
    + 建立了一个新表或者删除了一个旧表
    + 两个Tablet 被合并了
    + 一个 `Tablet` 被分割成两个小的 `Tablet`
  + `Master` 服务器可以跟踪记录所有这些事件，因为除了最后一个事件外的两个事件都是由它启动的。
  + `Tablet` 分割事件需要特殊处理，因为它是由 `Tablet` 服务器启动。
    + 在分割操作完成之后，`Tablet` 服务器通过在 `METADATA` 表中记录新的 `Tablet` 的信息来提交这个操作；
    + 当分割操作提交之后，`Tablet` 服务器会通知 `Master` 服务器。如果分割操作已提交的信息没有通知到 `Master` 服务器（可能两个服务器中有一个宕机了），`Master` 服务器在要求 `Tablet` 服务器装载已经被分割的子表的时候会发现一个新的 `Tablet`。
    + 通过对比 `METADATA` 表中 `Tablet` 的信息，`Tablet` 服务器会发现 `Master` 服务器要求其装载的 `Tablet` 并不完整，因此，`Tablet` 服务器会重新向 `Master` 服务器发送通知信息。

### tablet服务

+ `Tablet`服务

  ![](https://ws1.sinaimg.cn/large/77451733gy1g5up452u74j20gr09d3z1.jpg)
  + 如图所示，`Tablet` 的持久化状态信息保存在 `GFS` 上。更新操作提交到 `REDO` 日志中 
  + 在这些更新操作中，最近提交的那些存放在一个排序的缓存中，我们称这个缓存为 `memtable`
  + 较早的更新存放在一系列`SSTable` 中
  + 为了恢复一个 `Tablet`
    + `Tablet` 服务器首先从 `METADATA` 表中读取它的元数据
    + `Tablet` 的元数据包含了组成这个 `Tablet` 的 `SSTable` 的列表，以及一系列的 `Redo Point` ，这些 `Redo Point` 指向可能含有该 `Tablet`数据的已提交的日志记录
    + `Tablet` 服务器把 `SSTable` 的索引读进内存，之后通过重复 `Redo Point` 之后提交的更新来重建 `memtable`
  + 当对 `Tablet` 服务器进行写操作时
    + `Tablet` 服务器首先要检查这个操作格式是否正确、操作发起者是否有执行这个操作的权限。
    + 权限验证的方法是通过从一个 `Chubby` 文件里读取出来的具有写权限的操作者列表来进行验证（这个文件几乎一定会存放在 `Chubby` 客户缓存里）。
    + 成功的修改操作会记录在提交日志里。
    + 可以采用批量提交方式来提高包含大量小的修改操作的应用程序的吞吐量。
    + 当一个写操作提交后，写的内容插入到 `memtable` 里面。
  + 当对 `Tablet` 服务器进行读操作时
    + `Tablet` 服务器会作类似的完整性和权限检查
    + 一个有效的读操作在一个由一系列 `SSTable` 和 `memtable` 合并的视图里执行
    + 由于 `SSTable` 和 `memtable` 是按字典排序的数据结构，因此可以高效生成合并视图
  + 当进行 `Tablet` 的合并和分割时，正在进行的读写操作能够继续进行

### Compactions

+ 随着写操作的执行，`memtable` 的大小不断增加。当 `memtable` 的尺寸到达一个门限值的时候，这个 `memtable`就会被冻结，然后创建一个新的 `memtable`；被冻结住 `memtable` 会被转换成 `SSTable`，然后写入 `GFS`。
+ `Minor Compaction` 过程有两个目的：
  + `shrink Tablet` 服务器使用的内存，以及在服务器灾难恢复过程中，减少必须从提交日志里读取的数据量
  + 在 `Compaction` 过程中，正在进行的读写操作仍能继续
+ 每一次 `Minor Compaction` 都会创建一个新的 `SSTable`
  + 如果 `Minor Compaction` 过程不停滞的持续进行下去，读操作可能需要合并来自多个 `SSTable` 的更新；
  + 我们通过定期在后台执行 `Merging Compaction` 过程合并文件，限制这类文件的数量。
    + `Merging Compaction` 过程读取一些 `SSTable` 和 `memtable` 的内容，合并成一个新的 `SSTable`
    + 只要 `Merging Compaction` 过程完成了，输入的这些 `SSTable` 和 `memtable` 就可以删除了
+ 合并所有的 `SSTable` 并生成一个新的 `SSTable` 的 `Merging Compaction` 过程叫作 `Major Compaction`
  + 由 `non-Major Compaction` 产生的 `SSTable` 可能含有特殊的删除条目，这些删除条目能够隐藏在旧的、但是依然有效的`SSTable`中已经删除的数据 
  + 而`Major Compaction`过程生成的`SSTable`不包含已经删除的信息或数据。`Bigtable` 循环扫描它所有的 `Tablet`，并且定期对它们执行 `Major Compaction`
  + `Major Compaction` 机制允许 `Bigtable` 回收已经删除的数据占有的资源，并且确保 `BigTable` 能及时清除已经删除的数据 ，这对存放敏感数据的服务是非常重要

### 压缩

+ 两遍压缩
+ 在选择压缩算法的时候重点考虑的是速度而不是压缩的空间，但是这种两遍的压缩方式在空间压缩率上的表现也是令人惊叹

### 通过缓存提高读操作的性能

+ 为了提高读操作的性能，Tablet 服务器使用二级缓存的策略。
  + 扫描缓存是第一级缓存，主要缓存 `Tablet`服务器通过 `SSTable` 接口获取的 `Key-Value` 对；
  + `Block` 缓存是二级缓存，缓存的是从 `GFS` 读取的 `SSTable` 的`Block`。
+ 对于经常要重复读取相同数据的应用程序来说，扫描缓存非常有效；
+ 对于经常要读取刚刚读过的数据附近的数据的应用程序来说，`Block` 缓存更有用
  + 例如，顺序读，或者在一个热点的行的局部性群组（客户程序可以将多个列族组合成一个局部性群组）中随机读取不同的列
+ `Bloom`过滤器
  + 允许客户程序对特定局部性群组的 `SSTable` 指定 `Bloom` 过滤器来减少读操作硬盘访问的次数
  + 我们可以使用 Bloom 过滤器查询一个 SSTable 是否包含了特定行和列的数据。
  + 对于某些特定应用程序，我们只付出了少量的、用于存储 Bloom 过滤器的内存的代价，就换来了读操作显著减少的磁盘访问的次数。
  + 使用 Bloom 过滤器也隐式的达到了当应用程序访问不存在的行或列时，大多数时候我们都不需要访问硬盘的目的。
+ `commit`日志的实现
  + 如果我们把对每个 `Tablet` 的操作的 `Commit` 日志都存在一个单独的文件的话，那么就会产生大量的文件，并且这些文件会并行的写入 `GFS`。根据 `GFS` 服务器底层文件系统实现的方案，要把这些文件写入不同的磁盘
    日志文件时 ，会有大量的磁盘 `Seek` 操作。
  + 设置每个 `Tablet` 服务器一个 `Commit` 日志文件，把修改操作的日志以追加方式写入同一个日志文件，因此一个实际的日志文件中混合了对多个 `Tablet` 修改的日志记录。
  + 使用单个日志显著提高了普通操作的性能，但是将恢复的工作复杂化了。当一个 `Tablet` 服务器宕机时，它加载的 `Tablet` 将会被移到很多其它的 `Tablet` 服务器上：每个 `Tablet` 服务器都装载很少的几个原来的服务器的 `Tablet`。当恢复一个 `Tablet` 的状态的时候，新的 `Tablet` 服务器要从原来的 `Tablet` 服务器写的日志中提取修改操作的信息，并重新执行。然而，这些 `Tablet` 修改操作的日志记录都混合在同一个日志文件中的。一种方法新的 `Tablet` 服务器读取完整的 `Commit` 日志文件，然后只重复执行它需要恢复的 `Tablet` 的相关修改操作。使用这种方法，假如有 `100` 台 `Tablet` 服务器，每台都加载了失效的 `Tablet` 服务器上的一个 `Tablet`，那么，这个日志文件就要被读取 `100` 次（每个服务器读取一次）。
  + 为了避免多次读取日志文件，我们首先把日志按照关键字（`table`，`row name`，`log sequence number`）排序。排序之后，对同一个 `Tablet` 的修改操作的日志记录就连续存放在了一起，因此，我们只要一次磁盘 `Seek` 操作、之后顺序读取就可以了。为了并行排序，我们先将日志分割成 `64MB` 的段，之后在不同的 `Tablet` 服务器对段进行并行排序。这个排序工作由 `Master` 服务器来协同处理，并且在一个 `Tablet` 服务器表明自己需要从 `Commit`日志文件恢复 `Tablet` 时开始执行。