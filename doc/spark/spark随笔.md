# spark streaming 的基本原理
Spark streaming 是 spark core API 的一种扩展，可以用于进行大规模、高吞吐量、 容错的实时数据流的处理。
它支持从多种数据源读取数据，比如 Kafka、Flume、Twitter 和 TCP Socket，并且 能够使用算子比如 map、reduce、join 和 window 等来处理数据，处理后的数据可 以保存到文件系统、数据库等存储中。
Spark streaming 内部的基本工作原理是:接受实时输入数据流，然后将数据拆分成 batch，比如每收集一秒的数据封装成一个 batch，然后将每个 batch 交给 spark 的 计算引擎进行处理，最后会生产处一个结果数据流，其中的数据也是一个一个的 batch 组成的。



# spark 工作机制?
用户在 client 端提交作业后，会由 Driver 运行 main 方法并创建 spark context 上下 文。执行 add 算子，形成 dag 图输入 dagscheduler，按照 add 之间的依赖关系划分
stage 输入 task scheduler。task scheduler 会将 stage 划分为 task set 分发到各个 节点的 executor 中执行。

# Spark 主备切换机制原理知道吗?
Master 实际上可以配置两个，Spark 原生的 standalone 模式是支持 Master 主备切 换的。当 Active Master 节点挂掉以后，我们可以将 Standby Master 切换为 Active Master。
Spark Master 主备切换可以基于两种机制，一种是基于文件系统的，一种是基于 ZooKeeper 的。
基于文件系统的主备切换机制，需要在 Active Master 挂掉之后手动切换到 Standby Master 上;
而基于 Zookeeper 的主备切换机制，可以实现自动切换 Master。




# 简单说一下 hadoop 和 spark 的 shuffle 相同和差异?
1)从 high-level 的角度来看，两者并没有大的差别。 都是将 mapper (Spark 里是 ShuffleMapTask)的输出进行 partition，不同的 partition 送 到不同的 reducer(Spark 里 reducer 可能是下一个 stage 里的 ShuffleMapTask，也可能是 ResultTask)。Reducer 以内存作缓冲区，边 shuffle 边 aggregate 数据，等到数据 aggregate 好以后进行 reduce() (Spark 里可能是后续的一系列操作)。

2)从 low-level 的角度来看，两者差别不小。 Hadoop MapReduce 是 sort-based，进入 combine() 和 reduce() 的 records 必须先 sort。这样的好 处在于 combine/reduce() 可以处理大规模的数据，因为其输入数据可以通过 外排得到(mapper 对每段数据先做排序，reducer 的 shuffle 对排好序的每 段数据做归并)。目前的 Spark 默认选择的是 hash-based，通常使用 HashMap 来对 shuffle 来的数据进行 aggregate，不会对数据进行提前排序。
如果用户需要经过排序的数据，那么需要自己调用类似 sortByKey() 的操作; 如果你是 Spark 1.1 的用户，可以将 spark.shuffle.manager 设置为 sort，则 会对数据进行排序。在 Spark 1.2 中，sort 将作为默认的 Shuffle 实现。

3)从实现角度来看，两者也有不少差别。 Hadoop MapReduce 将处理流 程划分出明显的几个阶段:map(), spill, merge, shuffle, sort, reduce() 等。 每个阶段各司其职，可以按照过程式的编程思想来逐一实现每个阶段的功能。 在 Spark 中，没有这样功能明确的阶段，只有不同的 stage 和一系列的 transformation()，所以 spill, merge, aggregate 等操作需要蕴含在 transformation() 中。
如果我们将 map 端划分数据、持久化数据的过程称为 shuffle write，而将 reducer 读入数据、aggregate 数据的过程称为 shuffle read。那么在 Spark 中，问题就变为怎么在 job 的逻辑或者物理执行图中加入 shuffle write 和 shuffle read 的处理逻辑?以及两个处理逻辑应该怎么高效实现?
Shuffle write 由于不要求数据有序，shuffle write 的任务很简单:将数据 partition 好，并持久化。之所以要持久化，一方面是要减少内存存储空间压力， 另一方面也是为了 fault-tolerance。

# spark 工作机制
1 构建 Application 的运行环境，Driver 创建一个 SparkContext
2 SparkContext 向资源管理器(Standalone、Mesos、Yarn)申请 Executor 资源，资源管理器启动 StandaloneExecutorbackend(Executor) 3 Executor 向 SparkContext 申请 Task 4 SparkContext 将应用程序分发给 Executor 5 SparkContext 就建成 DAG 图，DAGScheduler 将 DAG 图解析 成 Stage，每个 Stage 有多个 task，形成 taskset 发送给 task Scheduler，由 task Scheduler 将 Task 发送给 Executor 运行 6 Task 在 Executor 上运行， 运行完释放所有资源

# spark 的优化怎么做? 
spark 调优比较复杂，但是大体可以分为三个方面来进行
1)平台层面的调优:防止不必要的 jar 包分发，提高数据的本地性，选择高 效的存储格式如 parquet
2)应用程序层面的调优:过滤操作符的优化降低过多小任务，降低单条记录 的资源开销，处理数据倾斜，复用 RDD 进行缓存，作业并行化执行等等
3)JVM 层面的调优:设置合适的资源量，设置合理的 JVM，启用高效的序 列化方法如 kyro，增大 off head 内存等等

# 数据本地性是在哪个环节确定的? 
具体的 task 运行在那他机器上，dag 划分 stage 的时候确定的

# RDD 的弹性表现在哪几点?
1)自动的进行内存和磁盘的存储切换; 2)基于 Lineage 的高效容错; 3)task 如果失败会自动进行特定次数的重试;
4)stage 如果失败会自动进行特定次数的重试，而且只会计算失败的分片; 5)checkpoint 和 persist，数据计算之后持久化缓存; 6)数据调度弹性，DAG TASK 调度和资源无关; 7)数据分片的高度弹性。

# RDD 有哪些缺陷?
1)不支持细粒度的写和更新操作(如网络爬虫)，spark 写数据是粗粒度的。 所谓粗粒度，就是批量写入数据，为了提高效率。但是读数据是细粒度的也就 是说可以一条条的读。
2)不支持增量迭代计算，Flink 支持

# Spark 的 shuffle 过程?
从下面三点去展开
1)shuffle 过程的划分 2)shuffle 的中间结果如何存储 3)shuffle 的数据如何拉取过来

# Spark 的数据本地性有哪几种?
Spark 中的数据本地性有三种:
1)PROCESS_LOCAL 是指读取缓存在本地节点的数据 2)NODE_LOCAL 是指读取本地节点硬盘数据
3)ANY 是指读取非本地节点数据
通常读取数据 PROCESS_LOCAL>NODE_LOCAL>ANY，尽量使数据以
PROCESS_LOCAL 或 NODE_LOCAL 方式读取。其中 PROCESS_LOCAL 还和 cache 有关，如果 RDD 经常用的话将该 RDD cache 到内存中，注意，由于 cache 是 lazy 的，所以必须通过一个 action 的触发，才能真正的将该 RDD cache 到内存中。

# Spark 为什么要持久化，一般什么场景下要进行 persist 操作?
为什么要进行持久化?
spark 所有复杂一点的算法都会有 persist 身影，spark 默认数据放在内存， spark 很多内容都是放在内存的，非常适合高速迭代，1000 个步骤只有第一个 输入数据，中间不产生临时数据，但分布式系统风险很高，所以容易出错，就 要容错，rdd 出错或者分片可以根据血统算出来，如果没有对父 rdd 进行 persist 或者 cache 的化，就需要重头做。 以下场景会使用 persist
1)某个步骤计算非常耗时，需要进行 persist 持久化 2)计算链条非常长，重新恢复要算很多步骤，很好使，persist 3)checkpoint 所在的 rdd 要持久化 persist。checkpoint 前，要持久化，
写个 rdd.cache 或者 rdd.persist，将结果保存起来，再写 checkpoint 操作， 这样执行起来会非常快，不需要重新计算 rdd 链条了。checkpoint 之前一定 会进行 persist。
4)shuffle 之后要 persist，shuffle 要进性网络传输，风险很大，数据丢失重 来，恢复代价很大
5)shuffle 之前进行 persist，框架默认将数据持久化到磁盘，这个是框架自 动做的。

# 介绍一下 join 操作优化经验?
join 其实常见的就分为两类: map-side join 和 reduce-side join。当大表 和小表 join 时，用 map-side join 能显著提高效率。将多份数据进行关联是数
据处理过程中非常普遍的用法，不过在分布式计算系统中，这个问题往往会变 的非常麻烦，因为框架提供的 join 操作一般会将所有数据根据 key 发送到所 有的 reduce 分区中去，也就是 shuffle 的过程。造成大量的网络以及磁盘 IO 消耗，运行效率极其低下，这个过程一般被称为 reduce-side-join。如果其中 有张表较小的话，我们则可以自己实现在 map 端实现数据关联，跳过大量数 据进行 shuffle 的过程，运行时间得到大量缩短，根据不同数据可能会有几倍 到数十倍的性能提升。
 备注:这个题目面试中非常非常大概率见到，务必搜索相关资料掌握，这里
抛砖引玉。

# 描述 Yarn 执行一个任务的过程?
1)客户端 client 向 ResouceManager 提交 Application， ResouceManager 接受 Application 并根据集群资源状况选取一个 node 来启 动 Application 的任务调度器 driver(ApplicationMaster)。
2)ResouceManager 找到那个 node，命令其该 node 上的 nodeManager 来启动一个新的 JVM 进程运行程序的 driver(ApplicationMaster)部分， driver(ApplicationMaster)启动时会首先向 ResourceManager 注册，说 明由自己来负责当前程序的运行。
3)driver(ApplicationMaster)开始下载相关 jar 包等各种资源，基于下载 的 jar 等信息决定向 ResourceManager 申请具体的资源内容。
4)ResouceManager 接受到 driver(ApplicationMaster)提出的申请后， 会最大化的满足 资源分配请求，并发送资源的元数据信息给 driver (ApplicationMaster)。
5)driver(ApplicationMaster)收到发过来的资源元数据信息后会根据元 数据信息发指令给具体机器上的 NodeManager，让其启动具体的 container。
6)NodeManager 收到 driver 发来的指令，启动 container，container 启 动后必须向 driver(ApplicationMaster)注册。
7)driver(ApplicationMaster)收到 container 的注册，开始进行任务的 调度和计算，直到 任务完成。
注意:如果 ResourceManager 第一次没有能够满足 driver (ApplicationMaster)的资源请求 ，后续发现有空闲的资源，会主动向 driver(ApplicationMaster)发送可用资源的元数据信息以提供更多的资源用 于当前程序的运行。

# Spark on Yarn 模式有哪些优点?
1)与其他计算框架共享集群资源(Spark 框架与 MapReduce 框架同时运行， 如果不用 Yarn 进行资源分配，MapReduce 分到的内存资源会很少，效率低 下);资源按需分配，进而提高集群资源利用等。
2)相较于 Spark 自带的 Standalone 模式，Yarn 的资源分配更加细致。
3)Application 部署简化，例如 Spark，Storm 等多种框架的应用由客户端 提交后，由 Yarn 负责资源的管理和调度，利用 Container 作为资源隔离的单 位，以它为单位去使用内存,cpu 等。
4)Yarn 通过队列的方式，管理同时运行在 Yarn 集群中的多个服务，可根据 不同类型的应用程序负载情况，调整对应的资源使用量，实现资源弹性管理。

# 谈谈你对 container 的理解?
1)Container 作为资源分配和调度的基本单位，其中封装了的资源如内存， CPU，磁盘，网络带宽等。 目前 yarn 仅仅封装内存和 CPU
2)Container 由 ApplicationMaster 向 ResourceManager 申请的，由 ResouceManager 中的资源调度器异步分配给 ApplicationMaster
3)Container 的运行是由 ApplicationMaster 向资源所在的 NodeManager 发起的，Container 运行时需提供内部执行的任务命令

# Spark 使用 parquet 文件存储格式能带来哪些好处?
1)如果说 HDFS 是大数据时代分布式文件系统首选标准，那么 parquet 则 是整个大数据时代文件存储格式实时首选标准。
2)速度更快:从使用 spark sql 操作普通文件 CSV 和 parquet 文件速度对 比上看，绝大多数情况会比使用 csv 等普通文件速度提升 10 倍左右，在一些 普通文件系统无法在 spark 上成功运行的情况下，使用 parquet 很多时候可以 成功运行。
3)parquet 的压缩技术非常稳定出色，在 spark sql 中对压缩技术的处理可 能无法正常的完成工作(例如会导致 lost task，lost executor)但是此时如果 使用 parquet 就可以正常的完成。
4)极大的减少磁盘 I/o,通常情况下能够减少 75%的存储空间，由此可以极大 的减少 spark sql 处理数据的时候的数据输入内容，尤其是在 spark1.6x 中有 个下推过滤器在一些情况下可以极大的减少磁盘的 IO 和内存的占用，(下推 过滤器)。
5)spark 1.6x parquet 方式极大的提升了扫描的吞吐量，极大提高了数据的 查找速度 spark1.6 和 spark1.5x 相比而言，提升了大约 1 倍的速度，在 spark1.6X 中，操作 parquet 时候 cpu 也进行了极大的优化，有效的降低了 cpu 消耗。
6)采用 parquet 可以极大的优化 spark 的调度和执行。我们测试 spark 如 果用 parquet 可以有效的减少 stage 的执行消耗，同时可以优化执行路径

# 介绍 parition 和 block 有什么关联关系?
1)hdfs 中的 block 是分布式存储的最小单元，等分，可设置冗余，这样设计 有一部分磁盘空间的浪费，但是整齐的 block 大小，便于快速找到、读取对应 的内容;
2)Spark 中的 partion 是弹性分布式数据集 RDD 的最小单元，RDD 是由分 布在各个节点上的 partion 组成的。partion 是指的 spark 在计算过程中，生 成的数据在计算空间内最小单元，同一份数据(RDD)的 partion 大小不一， 数量不定，是根据 application 里的算子和最初读入的数据分块数量决定;
3)block 位于存储空间、partion 位于计算空间，block 的大小是固定的、 partion 大小是不固定的，是从 2 个不同的角度去看数据。

# Spark 应用程序的执行过程是什么?
1)构建 Spark Application 的运行环境(启动 SparkContext)， SparkContext 向资源管理器(可以是 Standalone、Mesos 或 YARN)注册 并申请运行 Executor 资源;
2)资源管理器分配 Executor 资源并启动 StandaloneExecutorBackend， Executor 运行情况将随着心跳发送到资源管理器上;
3)SparkContext 构建成 DAG 图，将 DAG 图分解成 Stage，并把 Taskset 发送给 Task Scheduler。Executor 向 SparkContext 申请 Task，Task Scheduler 将 Task 发放给 Executor 运行同时 SparkContext 将应用程序代码 发放给 Executor;
4)Task 在 Executor 上运行，运行完毕释放所有资源。

# 不需要排序的 hash shuffle 是否一定比需要排序的 sort shuffle 速 度快?
不一定，当数据规模小，Hash shuffle 快于 Sorted Shuffle 数据规模大的时 候;当数据量大，sorted Shuffle 会比 Hash shuffle 快很多，因为数量大的有 很多小文件，不均匀，甚至出现数据倾斜，消耗内存大，1.x 之前 spark 使用 hash，适合处理中小规模，1.x 之后，增加了 Sorted shuffle，Spark 更能胜 任大规模处理了。

# Sort-based shuffle 的缺陷?
1)如果 mapper 中 task 的数量过大，依旧会产生很多小文件，此时在 shuffle 传递数据的过程中 reducer 段，reduce 会需要同时大量的记录进行反 序列化，导致大量的内存消耗和 GC 的巨大负担，造成系统缓慢甚至崩溃。
2)如果需要在分片内也进行排序，此时需要进行 mapper 段和 reducer 段的 两次排序。

# spark.storage.memoryFraction 参数的含义,实际生产中如何调优?
1)用于设置 RDD 持久化数据在 Executor 内存中能占的比例，默认是 0.6,， 默认 Executor 60%的内存，可以用来保存持久化的 RDD 数据。根据你选择的 不同的持久化策略，如果内存不够时，可能数据就不会持久化，或者数据会写 入磁盘;
2)如果持久化操作比较多，可以提高 spark.storage.memoryFraction 参数， 使得更多的持久化数据保存在内存中，提高数据的读取性能，如果 shuffle 的 操作比较多，有很多的数据读写操作到 JVM 中，那么应该调小一点，节约出更 多的内存给 JVM，避免过多的 JVM gc 发生。在 web ui 中观察如果发现 gc 时间很长，可以设置 spark.storage.memoryFraction 更小一点。