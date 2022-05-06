# Hive 表关联查询，如何解决数据倾斜的问题?
1)倾斜原因:map 输出数据按 key Hash 的分配到 reduce 中，由于 key 分 布不均匀、业务数据本身的特、建表时考虑不周、等原因造成的 reduce 上的 数据量差异过大。
(1)key 分布不均匀; (2)业务数据本身的特性; (3)建表时考虑不周;
(4)某些 SQL 语句本身就有数据倾斜;
如何避免:对于 key 为空产生的数据倾斜，可以对其赋予一个随机值。 2)解决方案
(1)参数调节:
hive.map.aggr = true
hive.groupby.skewindata=true 有数据倾斜的时候进行负载均衡，当选项设定位 true,生成的查询计划会有两
个 MR Job。第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的; 第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中(这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中)， 最后完成最终的聚合操作。
(2)SQL 语句调节:
1 选用 join key 分布最均匀的表作为驱动表。做好列裁剪和 filter 操作，以 达到两表做 join 的时候，数据量相对变小的效果。
2 大小表 Join:
使用 map join 让小的维度表(1000 条以下的记录条数)先进内存。在
map 端完成 reduce。 3 大表 Join 大表:
把空值的 key 变成一个字符串加上随机数，把倾斜的数据分到不同的 reduce 上，由于 null 值关联不上，处理后并不影响最终结果。

4 count distinct 大量相同特殊值:
count distinct 时，将值为空的情况单独处理，如果是计算 count distinct，
可以不用处理，直接过滤，在最后结果中加 1。如果还有其他计算，需要进行 group by，可以先将值为空的记录单独处理，再和其他计算结果进行 union。

# Hive 的 HSQL 转换为 MapReduce 的过程?
HiveSQL ->AST(抽象语法树) -> QB(查询块) ->OperatorTree(操作树)-> 优化后的操作树->mapreduce 任务树->优化后的 mapreduce 任务树
过程描述如下:
SQL Parser:Antlr 定义 SQL 的语法规则，完成 SQL 词法，语法解析，将
SQL 转化为抽象语法树 AST Tree;
Semantic Analyzer:遍历 AST Tree，抽象出查询的基本组成单元
QueryBlock;
Logical plan:遍历 QueryBlock，翻译为执行操作树 OperatorTree; Logical plan optimizer: 逻辑层优化器进行 OperatorTree 变换，合并不必
要的 ReduceSinkOperator，减少 shuffle 数据量;
Physical plan:遍历 OperatorTree，翻译为 MapReduce 任务;
Logical plan optimizer:物理层优化器进行 MapReduce 任务的变换，生
成最终的执行计划。

# Hive 底层与数据库交互原理?
由于 Hive 的元数据可能要面临不断地更新、修改和读取操作，所以它显然不 适合使用 Hadoop 文件系统进行存储。目前 Hive 将元数据存储在 RDBMS 中， 比如存储在 MySQL、Derby 中。元数据信息包括:存在的表、表的列、权限 和更多的其他信息。

# Hive 的两张表关联，使用 MapReduce 怎么实现?
如果其中有一张表为小表，直接使用 map 端 join 的方式(map 端加载小表) 进行聚合。
如果两张都是大表，那么采用联合 key，联合 key 的第一个组成部分是 join on 中的公共字段，第二部分是一个 flag，0 代表表 A，1 代表表 B，由此让 Reduce 区分客户信息和订单信息;在 Mapper 中同时处理两张表的信息，将 join on 公共字段相同的数据划分到同一个分区中，进而传递到一个 Reduce 中，然后在 Reduce 中实现聚合。

# 请谈一下 Hive 的特点?
hive 是基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为 一张数据库表，并提供完整的 sql 查询功能，可以将 sql 语句转换为 MapReduce 任务进行运行。其优点是学习成本低，可以通过类 SQL 语句快速 实现简单的 MapReduce 统计，不必开发专门的 MapReduce 应用，十分适合 数据仓库的统计分析，但是 Hive 不支持实时查询。

# 请说明 hive 中 Sort By，Order By，Cluster By，Distrbute By 各代表什么意思?
order by:会对输入做全局排序，因此只有一个 reducer(多个 reducer 无 法保证全局有序)。只有一个 reducer，会导致当输入规模较大时，需要较长 的计算时间。
sort by:不是全局排序，其在数据进入 reducer 前完成排序。
distribute by:按照指定的字段对数据进行划分输出到不同的 reduce 中。 cluster by:除了具有 distribute by 的功能外还兼具 sort by 的功能。

# 写出 hive 中 split、coalesce 及 collect_list 函数的用法(可举例)?
split 将字符串转化为数组，即:split('a,b,c,d' , ',') ==> ["a","b","c","d"]。
coalesce(T v1, T v2, ...) 返回参数中的第一个非空值;如果所有值都为 NULL， 那么返回 NULL。
collect_list 列出该字段所有的值，不去重 => select collect_list(id) from table。

# Hive 有哪些方式保存元数据，各有哪些特点?
Hive 支持三种不同的元存储服务器，分别为:内嵌式元存储服务器、本地元 存储服务器、远程元存储服务器，每种存储方式使用不同的配置参数。
内嵌式元存储主要用于单元测试，在该模式下每次只有一个进程可以连接到 元存储，Derby 是内嵌式元存储的默认数据库。
在本地模式下，每个 Hive 客户端都会打开到数据存储的连接并在该连接上请 求 SQL 查询。
在远程模式下，所有的 Hive 客户端都将打开一个到元数据服务器的连接，该 服务器依次查询元数据，元数据服务器和客户端之间使用 Thrift 协议通信。

# Hive 内部表和外部表的区别?
创建表时:创建内部表时，会将数据移动到数据仓库指向的路径;若创建外
部表，仅记录数据所在的路径，不对数据的位置做任何改变。
删除表时:在删除表的时候，内部表的元数据和数据会被一起删除， 而外部 表只删除元数据，不删除数据。这样外部表相对来说更加安全些，数据组织也 更加灵活，方便共享源数据。

# Hive 中的压缩格式 TextFile、SequenceFile、RCfile 、ORCfile 各有什么区别?
1、TextFile
默认格式，存储方式为行存储，数据不做压缩，磁盘开销大，数据解析开销 大。可结合 Gzip、Bzip2 使用(系统自动检查，执行查询时自动解压)，但使用 这种方式，压缩后的文件不支持 split，Hive 不会对数据进行切分，从而无法 对数据进行并行操作。并且在反序列化过程中，必须逐个字符判断是不是分隔 符和行结束符，因此反序列化开销会比 SequenceFile 高几十倍。
2、SequenceFile
SequenceFile 是 Hadoop API 提供的一种二进制文件支持，存储方式为行存
储，其具有使用方便、可分割、可压缩的特点。
SequenceFile 支持三种压缩选择:NONE，RECORD，BLOCK。Record 压
缩率低，一般建议使用 BLOCK 压缩。
优势是文件和 hadoop api 中的 MapFile 是相互兼容的
3、RCFile 存储方式:数据按行分块，每块按列存储。结合了行存储和列存储的优点:
首先，RCFile 保证同一行的数据位于同一节点，因此元组重构的开销很低;
其次，像列存储一样，RCFile 能够利用列维度的数据压缩，并且能跳过不必 要的列读取;
4、ORCFile
存储方式:数据按行分块 每块按照列存储。
压缩快、快速列存取。
效率比 rcfile 高，是 rcfile 的改良版本。
总结:相比 TEXTFILE 和 SEQUENCEFILE，RCFILE 由于列式存储方式，数
据加载时性能消耗较大，但是具有较好的压缩比和查询响应。 数据仓库的特点是一次写入、多次读取，因此，整体来看，RCFILE 相比其
余两种格式具有较明显的优势。

# 所有的 Hive 任务都会有 MapReduce 的执行吗?
不是，从 Hive0.10.0 版本开始，对于简单的不需要聚合的类似 SELECT from
LIMIT n 语句，不需要起 MapReduce job，直接通过 Fetch task 获取数据。

# Hive 的函数:UDF、UDAF、UDTF 的区别?
UDF:单行进入，单行输出
UDAF:多行进入，单行输出 UDTF:单行输入，多行输出

# 说说对 Hive 桶表的理解?
桶表是对数据进行哈希取值，然后放到不同文件中存储。
数据加载到桶表时，会对字段取 hash 值，然后与桶的数量取模。把数据放到 对应的文件中。物理上，每个桶就是表(或分区)目录里的一个文件，一个作业 产生的桶(输出文件)和 reduce 任务个数相同。
 桶表专门用于抽样查询，是很专业性的，不是日常用来存储数据的表，需要
抽样查询时，才创建和使用桶表。