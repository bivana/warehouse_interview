# HBase 的特点是什么?
1)大:一个表可以有数十亿行，上百万列;
2)无模式:每行都有一个可排序的主键和任意多的列，列可以根据需要动态 的增加，同一张表中不同的行可以有截然不同的列;
3)面向列:面向列(族)的存储和权限控制，列(族)独立检索; 4)稀疏:空(null)列并不占用存储空间，表可以设计的非常稀疏; 5)数据多版本:每个单元中的数据可以有多个版本，默认情况下版本号自动
分配，是单元格插入时的时间戳;
6)数据类型单一:Hbase 中的数据都是字符串，没有类型。

# HBase 适用于怎样的情景?
1 半结构化或非结构化数据 对于数据结构字段不够确定或杂乱无章很难按 一个概念去进行抽取的数据适合用 HBase。以上面的例子为例，当业务发展需 要存储 author 的 email，phone， address 信息时 RDBMS 需要停机维护， 而 HBase 支持动态增加。
2 记录非常稀疏
RDBMS 的行有多少列是固定的，为 null 的列浪费了存储空间。而如上文提 到的，HBase 为 null 的 Column 不会被存储，这样既节省了空间又提高了读 性能。
3 多版本数据
如上文提到的根据 Row key 和 Column key 定位到的 Value 可以有任意数量 的版本值，因此对于需要存储变动历史记录的数据，用 HBase 就非常方便了。 比如上例中的 author 的 Address 是会变动的，业务上一般只需要最新的值， 但有时可能需要查询到历史值。
4 超大数据量
当数据量越来越大，RDBMS 数据库撑不住了，就出现了读写分离策略，通过 一个 Master 专门负责写操作，多个 Slave 负责读操作，服务器成本倍增。 随 着压力增加，Master 撑不住了，这时就要分库了，把关联不大的数据分开部署， 一些 join 查询不能用了，需要借助中间层。随着数据量的进一步增加， 一个表 的记录越来越大，查询就变得很慢，于是又得搞分表，比如按 ID 取模分成多个 表以减少单个表的记录数。经历过这些事的人都知道过程是多么的折腾。 采用 HBase 就简单了，只需要加机器即可，HBase 会自动水平切分扩展，跟 Hadoop 的无缝集成保障了其数据可靠性(HDFS)和海量数据分析的高性能 (MapReduce)。

# 描述 HBase 的 rowKey 的设计原则?
(1)Rowkey 长度原则
Rowkey 是一个二进制码流，Rowkey 的长度被很多开发者建议说设计在
10~100 个字节，不过建议是越短越好，不要超过 16 个字节。
原因如下:
1 数据的持久化文件 HFile 中是按照 KeyValue 存储的，如果 Rowkey 过长
比如 100 个字节，1000 万列数据光 Rowkey 就要占用 100*1000 万=10 亿个 字节， 将近 1G 数据，这会极大影响 HFile 的存储效率;
2 MemStore 将缓存部分数据到内存，如果 Rowkey 字段过长内存的有效利 用率会降低，系统将无法缓存更多的数据，这会降低检索效率。 因此 Rowkey 的字节长度越短越好。
3 目前操作系统是都是 64 位系统，内存 8 字节对齐。控制在 16 个字节，8 字节的整数倍利用操作系统的最佳特性。
(2)Rowkey 散列原则
如果 Rowkey 是按时间戳的方式递增，不要将时间放在二进制码的前面，建 议将 Rowkey 的高位作为散列字段，由程序循环生成，低位放时间字段， 这样 将提高数据均衡分布在每个 Regionserver 实现负载均衡的几率。如果没有散 列字段，首字段直接是时间信息将产生所有新数据都在一个 RegionServer 上 堆积的 热点现象，这样在做数据检索的时候负载将会集中在个别 RegionServer，降低查询效率。
(3)Rowkey 唯一原则
 必须在设计上保证其唯一性。

 # 描述 HBase 中 scan 和 get 的功能以及实现的异同?
 HBase 的查询实现只提供两种方式:
1)按指定 RowKey 获取唯一一条记录，get 方法
(org.apache.hadoop.hbase.client.Get) Get 的方法处理分两种 : 设置了 ClosestRowBefore 和 没有设置 ClosestRowBefore 的 rowlock。主要是用来 保证行的事务性，即每个 get 是以一个 row 来标记的。一个 row 中可以有很 多 family 和 column。
2)按指定的条件获取一批记录，scan 方法 (org.apache.Hadoop.hbase.client.Scan)实现条件查询功能使用的就是 scan 方式。
(1)scan 可以通过 setCaching 与 setBatch 方法提高速度(以空间换时间);
(2)scan 可以通过 setStartRow 与 setEndRow 来限定范围([start， end)start 是闭区间，end 是开区间)。范围越小，性能越高。
(3)scan 可以通过 setFilter 方法添加过滤器，这也是分页、多条件查询的 基础。

# 请详细描述 HBase 中一个 cell 的结构?
HBase 中通过 row 和 columns 确定的为一个存贮单元称为 cell。
Cell:由{row key, column(= + ), version}唯一确定的单元。cell 中的数据是 没有类型的，全部是字节码形式存贮。

# 简述 HBase 中 compact 用途是什么，什么时候触发，分为哪两种， 有什么区别，有哪些相关配置参数?
在 hbase 中每当有 memstore 数据 flush 到磁盘之后，就形成一个 storefile， 当 storeFile 的数量达到一定程度后，就需要将 storefile 文件来 进行 compaction 操作。
Compact 的作用:
1 合并文件
2 清除过期，多余版本的数据
3 提高读写数据的效率
HBase 中实现了两种 compaction 的方式:minor and major. 这两种
compaction 方式的区别是:
1)Minor 操作只用来做部分文件的合并操作以及包括 minVersion=0 并且
设置 ttl 的过期版本清理，不做任何删除数据、多版本数据的清理工作。 2)Major 操作是对 Region 下的 HStore 下的所有 StoreFile 执行合并操作，
最终的结果是整理合并出一个文件。

# 每天百亿数据存入 HBase，如何保证数据的存储正确和在规定的时 间里全部录入完毕，不残留数据?

需求分析:
1)百亿数据:证明数据量非常大;
2)存入 HBase:证明是跟 HBase 的写入数据有关; 3)保证数据的正确:要设计正确的数据结构保证正确性; 4)在规定时间内完成:对存入速度是有要求的。
解决思路:
1)数据量百亿条，什么概念呢?假设一整天 60x60x24 = 86400 秒都在写入
数据，那么每秒的写入条数高达 100 万条，HBase 当然是支持不了每秒百万条 数据的， 所以这百亿条数据可能不是通过实时地写入，而是批量地导入。批量 导入推荐使用 BulkLoad 方式(推荐阅读:Spark 之读写 HBase)，性能是普 通写入方式几倍以上;
2)存入 HBase:普通写入是用 JavaAPI put 来实现，批量导入推荐使用 BulkLoad;
3)保证数据的正确:这里需要考虑 RowKey 的设计、预建分区和列族设计等 问题;
4)在规定时间内完成也就是存入速度不能过慢，并且当然是越快越好，使用 BulkLoad。

# 请列举几个 HBase 优化方法?
1)减少调整
减少调整这个如何理解呢?HBase 中有几个内容会动态调整，如 region(分 区)、HFile，所以通过一些方法来减少这些会带来 I/O 开销的调整。
1 Region
如果没有预建分区的话，那么随着 region 中条数的增加，region 会进行分
裂，这将增加 I/O 开销，所以解决方法就是根据你的 RowKey 设计来进行预建 分区， 减少 region 的动态分裂。
2 HFile
HFile 是数据底层存储文件，在每个 memstore 进行刷新时会生成一个
HFile，当 HFile 增加到一定程度时，会将属于一个 region 的 HFile 进行合并， 这个步骤会带来开销但不可避免，但是合并后 HFile 大小如果大于设定的值， 那么 HFile 会重新分裂。为了减少这样的无谓的 I/O 开销，建议估计项目数据 量大小， 给 HFile 设定一个合适的值。
2)减少启停 数据库事务机制就是为了更好地实现批量写入，较少数据库的开启关闭带来
的开销，那么 HBase 中也存在频繁开启关闭带来的问题。 1 关闭 Compaction，在闲时进行手动 Compaction。
因为 HBase 中存在 Minor Compaction 和 Major Compaction，也就是对 HFile 进行合并，所谓合并就是 I/O 读写，大量的 HFile 进行肯定会带来 I/O 开销， 甚至是 I/O 风暴，所以为了避免这种不受控制的意外发生，建议关闭自 动 Compaction，在闲时进行 compaction。
2 批量数据写入时采用 BulkLoad。

如果通过 HBase-Shell 或者 JavaAPI 的 put 来实现大量数据的写入，那么性 能差是肯定并且还可能带来一些意想不到的问题，所以当需要写入大量离线数 据时 建议使用 BulkLoad。
3)减少数据量
 虽然我们是在进行大数据开发，但是如果可以通过某些方式在保证数据准确
性同时减少数据量，何乐而不为呢?
1 开启过滤，提高查询速度
开启 BloomFilter，BloomFilter 是列族级别的过滤，在生成一个 StoreFile
同时会生成一个 MetaBlock，用于查询时过滤数据 2 使用压缩
一般推荐使用 Snappy 和 LZO 压缩 4)合理设计
在一张 HBase 表格中 RowKey 和 ColumnFamily 的设计是非常重要，好的 设计能够提高性能和保证数据的准确性
1 RowKey 设计:应该具备以下几个属性 散列性:散列性能够保证相同相似的 rowkey 聚合，相异的 rowkey 分散，
有利于查询。
简短性:rowkey 作为 key 的一部分存储在 HFile 中，如果为了可读性将
rowKey 设计得过长，那么将会增加存储压力。 唯一性:rowKey 必须具备明显的区别性。 业务性:举例来说:

假如我的查询条件比较多，而且不是针对列的条件，那么 rowKey 的设计就 应该支持多条件查询。
如果我的查询要求是最近插入的数据优先，那么 rowKey 则可以采用叫上 Long.Max-时间戳的方式，这样 rowKey 就是递减排列。
2 列族的设计:列族的设计需要看应用场景
优势:HBase 中数据时按列进行存储的，那么查询某一列族的某一列时就不
需要全盘扫描，只需要扫描某一列族，减少了读 I/O; 其实多列族设计对减少 的作用不是很明显，适用于读多写少的场景
劣势:降低了写的 I/O 性能。原因如下:数据写到 store 以后是先缓存在 memstore 中，同一个 region 中存在多个列族则存在多个 store， 每个 store 都一个 memstore，当其实 memstore 进行 flush 时，属于同一个 region 的 store 中的 memstore 都会进行 flush，增加 I/O 开销。

# Region 如何预建分区?
预分区的目的主要是在创建表的时候指定分区数，提前规划表有多个分区， 以及每个分区的区间范围，这样在存储的时候 rowkey 按照分区的区间存储， 可以避免 region 热点问题。
通常有两种方案: 方案 1:shell 方法
create 'tb_splits', {NAME => 'cf',VERSIONS=> 3},{SPLITS => ['10','20','30']}
方案 2:JAVA 程序控制
1 取样，先随机生成一定数量的 rowkey,将取样数据按升序排序放到一个集
合里;
2 根据预分区的 region 个数，对整个集合平均分割，即是相关的 splitKeys; 3 HBaseAdmin.createTable(HTableDescriptor
tableDescriptor,byte[][]splitkeys)可以指定预分区的 splitKey， 即是指定 region 间的 rowkey 临界值。

# HRegionServer 宕机如何处理?
1)ZooKeeper 会监控 HRegionServer 的上下线情况，当 ZK 发现某个 HRegionServer 宕机之后会通知 HMaster 进行失效备援;
2)该 HRegionServer 会停止对外提供服务，就是它所负责的 region 暂时停 止对外提供服务;
3)HMaster 会将该 HRegionServer 所负责的 region 转移到其他 HRegionServer 上，并且会对 HRegionServer 上存在 memstore 中还未持久 化到磁盘中的数据进行恢复;
4)这个恢复的工作是由 WAL 重播来完成，这个过程如下:
1 wal 实际上就是一个文件，存在/hbase/WAL/对应 RegionServer 路径下。
2 宕机发生时，读取该 RegionServer 所对应的路径下的 wal 文件，然后根 据不同的 region 切分成不同的临时文件 recover.edits。
3 当 region 被分配到新的 RegionServer 中，RegionServer 读取 region 时会进行是否存在 recover.edits，如果有则进行恢复。

# HBase 读写流程?
读:
1 HRegionServer 保存着 meta 表以及表数据，要访问表数据，首先 Client 先去访问 zookeeper，从 zookeeper 里面获取 meta 表所在的位置信息， 即 找到这个 meta 表在哪个 HRegionServer 上保存着。
2 接着 Client 通过刚才获取到的 HRegionServer 的 IP 来访问 Meta 表所在
的 HRegionServer，从而读取到 Meta，进而获取到 Meta 表中存放的元数据。
3 Client 通过元数据中存储的信息，访问对应的 HRegionServer，然后扫描 所在 HRegionServer 的 Memstore 和 Storefile 来查询数据。
4 最后 HRegionServer 把查询到的数据响应给 Client。 写:
1 Client 先访问 zookeeper，找到 Meta 表，并获取 Meta 表元数据。
2 确定当前将要写入的数据所对应的 HRegion 和 HRegionServer 服务器。 3 Client 向该 HRegionServer 服务器发起写入数据请求，然后
HRegionServer 收到请求并响应。
4 Client 先把数据写入到 HLog，以防止数据丢失。
5 然后将数据写入到 Memstore。
6 如果 HLog 和 Memstore 均写入成功，则这条数据写入成功。
7 如果 Memstore 达到阈值，会把 Memstore 中的数据 flush 到 Storefile
中。
8 当 Storefile 越来越多，会触发 Compact 合并操作，把过多的 Storefile
合并成一个大的 Storefile。
9 当 Storefile 越来越大，Region 也会越来越大，达到阈值后，会触发 Split
操作，将 Region 一分为二。

# HBase 内部机制是什么?
Hbase 是一个能适应联机业务的数据库系统
物理存储:hbase 的持久化数据是将数据存储在 HDFS 上。 存储管理:一个表是划分为很多 region 的，这些 region 分布式地存放在很
多 regionserver 上 Region 内部还可以划分为 store， store 内部有 memstore 和 storefile。
版本管理:hbase 中的数据更新本质上是不断追加新的版本，通过 compact 操作来做版本间的文件合并 Region 的 split。
集群管理:ZooKeeper + HMaster + HRegionServer。

# Hbase 中的 memstore 是用来做什么的?

hbase 为了保证随机读取的性能，所以 hfile 里面的 rowkey 是有序的。当客 户端的请求在到达 regionserver 之后，为了保证写入 rowkey 的有序性， 所 以不能将数据立刻写入到 hfile 中，而是将每个变更操作保存在内存中，也就是 memstore 中。memstore 能够很方便的支持操作的随机插入， 并保证所有的 操作在内存中是有序的。当 memstore 达到一定的量之后，会将 memstore 里面的数据 flush 到 hfile 中，这样能充分利用 hadoop 写入大文件的性能优 势， 提高写入性能。
由于 memstore 是存放在内存中，如果 regionserver 因为某种原因死了，会 导致内存中数据丢失。所有为了保证数据不丢失， hbase 将更新操作在写入 memstore 之前会写入到一个 write ahead log(WAL)中。WAL 文件是追加、 顺序写入的，WAL 每个 regionserver 只有一个， 同一个 regionserver 上所 有 region 写入同一个的 WAL 文件。这样当某个 regionserver 失败时，可以 通过 WAL 文件，将所有的操作顺序重新加载到 memstore 中。

# HBase 在进行模型设计时重点在什么地方?一张表中定义多少个 Column Family 最合适?为什么?
Column Family 的个数具体看表的数据，一般来说划分标准是根据数据访问 频度，如一张表里有些列访问相对频繁，而另一些列访问很少， 这时可以把这 张表划分成两个列族，分开存储，提高访问效率。


# 如何提高 HBase 客户端的读写性能?请举例说明
1 开启 bloomfilter 过滤器，开启 bloomfilter 比没开启要快 3、4 倍
2 Hbase 对于内存有特别的需求，在硬件允许的情况下配足够多的内存给它 3 通过修改 hbase-env.sh 中的 export HBASE_HEAPSIZE=3000 #这里默
认为 1000m
4 增大 RPC 数量
通过修改 hbase-site.xml 中的 hbase.regionserver.handler.count 属性， 可以适当的放大 RPC 数量，默认值为 10 有点小。

# HBase 集群安装注意事项?
1 HBase 需要 HDFS 的支持，因此安装 HBase 前确保 Hadoop 集群安装完 成;
2 HBase 需要 ZooKeeper 集群的支持，因此安装 HBase 前确保 ZooKeeper 集群安装完成;
3 注意 HBase 与 Hadoop 的版本兼容性;
4 注意 hbase-env.sh 配置文件和 hbase-site.xml 配置文件的正确配置;
5 注意 regionservers 配置文件的修改;
6 注意集群中的各个节点的时间必须同步，否则启动 HBase 集群将会报错。

# 直接将时间戳作为行健，在写入单个 region 时候会发生热点问题， 为什么呢?
region 中的 rowkey 是有序存储，若时间比较集中。就会存储到一个 region 中，这样一个 region 的数据变多，其它的 region 数据很少，加载数据就会很 慢， 直到 region 分裂，此问题才会得到缓解。

# 请描述如何解决 HBase 中 region 太小和 region 太大带来的冲突?
Region 过大会发生多次 compaction，将数据读一遍并重写一遍到 hdfs 上， 占用 io，region 过小会造成多次 split，region 会下线，影响访问服务， 最佳 的解决方法是调整 hbase.hregion. max.filesize 为 256m。