# Spark 为什么比 mapreduce 快?  
1、任务模型的优化
   * mapreduce框架中，一个程序只能拥有一个map一个reduce的过程，如果运算逻辑很复杂，一个map+一个reduce是表述不出来的，可能就需要多个map-reduce的过程；mapreduce框架想要做到这个事情，就需要把第一个map-reduce过程产生的结果，写入HDFS，然后由第二个map-reduce过程去hdfs读取后计算，完成后又将结果写入HDFS，再交由第三个map-reduce过程去计算！ 重点！！！--这样一来，一个复杂的运算，在mapreduce框架中可能就会发生很多次写入并读取HDFS的操作，而读写HDFS是很慢的事情

   * spark框架，采用的是以rdd为核心，dag为调度，把上面的mapreduce-mapreduce-mapreduce的过程，连续执行，不需要反复落地到HDFS，这样就会比mapreduce快很多啦

2、spark支持在内存中缓存结果
   比如一个复杂逻辑中 ，一个map-reduce产生的结果A，如果在后续的map-reduce过程中需要反复用到，spark可以把A缓存到内存中，这样后续的map-reduce过程就只需要从内存中读取A即可，也会加快速度


# spark 对比 mapreduce的优势有哪些
* 计算模型优势，spark的核心技术是弹性分布式数据集(Resilient Distributed Datasets)，提供了比 MapReduce 丰富的模型，可以快速在内存中对数据集 进行多次迭代，来支持复杂的数据挖掘算法和图形计算算法。。
* Spark 和 Hadoop 的根本差异是多个作业之间的数据通信问题 : Spark 多个作业之间数据 通信是基于内存，而 Hadoop 是基于磁盘。
* Spark Task的启动时间快。Spark采用fork线程的方式，而Hadoop采用创建新的进程 的方式。
* Spark只有在shuffle的时候将数据写入磁盘，而Hadoop中多个MR作业之间的数据交 互都要依赖于磁盘交互
* Spark的缓存机制比HDFS的缓存机制高效。 
