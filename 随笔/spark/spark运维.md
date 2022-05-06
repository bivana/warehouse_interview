# spark 的有几种部署模式，每种模式特点?
## 1) 本地模式
Spark 不一定非要跑在 hadoop 集群，可以在本地，起多个线程的方式来指
定。将 Spark 应用以多线程的方式直接运行在本地，一般都是为了方便调试， 本地模式分三类
local:只启动一个 executor
local[k]:启动 k 个 executor
local[*]:启动跟 cpu 数目相同的 executor 

## 2) standalone 独立部署模式
分布式部署集群，自带完整的服务，资源管理和任务监控是 Spark 自己监控， 这个模式也是其他模式的基础。

## 3) Spark on yarn 模式
分布式部署集群，资源和任务监控交给 yarn 管理，但是目前仅支持粗粒度资 源分配方式，包含 cluster 和 client 运行模式，cluster 适合生产，driver 运行 在集群子节点，具有容错功能，client 适合调试，dirver 运行在客户端。 

## 4) Spark On K8S & Mesos 模式。
官方推荐这种模式(当然，原因之一是血缘关系)。正是由于 Spark 开发之 初就考虑到支持 Mesos，因此，目前而言，Spark 运行在 Mesos 上会比运行 在 YARN 上更加灵活，更加自然。用户可选择两种调度模式之一运行自己的应 用程序:

(1)粗粒度模式(Coarse-grained Mode):每个应用程序的运行环境由 一个 Dirver 和若干个 Executor 组成，其中，每个 Executor 占用若干资源， 内部可运行多个 Task(对应多少个“slot”)。应用程序的各个任务正式运行 之前，需要将运行环境中的资源全部申请好，且运行过程中要一直占用这些资 源，即使不用，最后程序运行结束后，回收这些资源。

(2)细粒度模式(Fine-grained Mode):鉴于粗粒度模式会造成大量资源 浪费，Spark On Mesos 还提供了另外一种调度模式:细粒度模式，这种模式 类似于现在的云计算，思想是按需分配。


