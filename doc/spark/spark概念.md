# spark 是什么?
spark的官方是这样定义的
> Apache Spark™ is a multi-language engine for executing data engineering, data science, and machine learning on single-node machines or clusters.

翻译过来就是 spark是 在单节点机器或集群上执行数据工程、数据科学和机器学习 的多语言引擎。

简单一句话就是Spark 是一种基于内存的快速、通用、可扩展的大数据分析计算引擎。

# spark 有哪些组件?
* master:管理集群和节点，不参与计算。
* worker:计算节点，进程本身不参与计算， 和 master 汇报。
* Driver:运行程序的 main 方法，创建 spark context 对象。
* spark context:控制整个 application 的生命周期，包括 dagsheduler 和 task scheduler 等组件。
* client:用户提交程序的入口。

## Driver
Spark 驱动器节点，用于执行 Spark 任务中的 main 方法，负责实际代码的执行工作。 Driver 在 Spark 作业执行时主要负责:
➢ 将用户程序转化为作业(job)
➢ 在Executor之间调度任务(task) SchedulerBackend
➢ 跟踪Executor的执行情况
➢ 通过UI展示查询运行情况
实际上，我们无法准确地描述 Driver 的定义，因为在整个的编程过程中没有看到任何有关 Driver 的字眼。所以简单理解，所谓的 Driver 就是驱使整个应用运行起来的程序，也称之为 Driver 类。

## Executor
Spark Executor 是集群中工作节点(Worker)中的一个 JVM 进程，负责在 Spark 作业中运行具体任务(Task)，任务彼此之间相互独立。Spark 应用启动时，Executor 节点被同时启动，并且始终伴随着整个 Spark 应用的生命周期而存在。如果有 Executor 节点发生了 故障或崩溃，Spark 应用也可以继续执行，会将出错节点上的任务调度到其他 Executor 节点 上继续运行。
Executor 有两个核心功能:
➢ 负责运行组成Spark应用的任务，并将结果返回给驱动器进程
➢ 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的 RDD 提供内存式存储。RDD 是直接缓存在 Executor 进程内的，因此任务可以在运行时充分利用缓存 数据加速运算。

## Master & Worker
Spark 集群的独立部署环境中，不需要依赖其他的资源调度框架，自身就实现了资源调 度的功能，所以环境中还有其他两个核心组件:Master 和 Worker，这里的 Master 是一个进 程，主要负责资源的调度和分配，并进行集群的监控等职责，类似于 Yarn 环境中的 RM, 而 Worker 呢，也是进程，一个 Worker 运行在集群中的一台服务器上，由 Master 分配资源对 数据进行并行的处理和计算，类似于 Yarn 环境中 NM。

## ApplicationMaster
Hadoop 用户向 YARN 集群提交应用程序时,提交程序中应该包含 ApplicationMaster，用 于向资源调度器申请执行任务的资源容器 Container，运行用户自己的程序任务 job，监控整 个任务的执行，跟踪整个任务的状态，处理任务失败等异常情况。
说的简单点就是，ResourceManager(资源)和 Driver(计算)之间的解耦合靠的就是 ApplicationMaster。

# spark的一些核心概念
## 

# 什么是 RDD
RDD(Resilient Distributed Dataset)叫做弹性分布式数据集，是 Spark 中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行 计算的集合。
➢ 弹性
⚫ 存储的弹性:内存与磁盘的自动切换; 
⚫ 容错的弹性:数据丢失可以自动恢复; 
⚫ 计算的弹性:计算出错重试机制;
⚫ 分片的弹性:可根据需要重新分片。
➢ 分布式:数据存储在大数据集群不同节点上
➢ 数据集:RDD封装了计算逻辑，并不保存数据
➢ 数据抽象:RDD是一个抽象类，需要子类具体实现
➢ 不可变:RDD封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的 RDD 里面封装计算逻辑
➢ 可分区、并行计算