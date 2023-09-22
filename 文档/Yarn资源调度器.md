# Yarn 资源调度器

> Yarn 是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce 等运算程序相当于运行于操作系统之上的应用程序

## 1.1 基础结构

**YARN 主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件构成**

![](/图片/Yarn组成.png)

## 1.2 Yarn 工作机制

![](/图片/Yarn工作机制.png)

- MR 程序提交到客户端所在的节点
- YarnRunner 向 ResourceManager 申请一个Application。
- RM 将该应用程序的资源路径返回给 YarnRunner。
- 该程序将运行所需资源提交到HDFS 上。
- 程序资源提交完毕后，申请运行 mrAppMaster。
- RM 将用户的请求初始化成一个 Task。
- 其中一个 NodeManager 领取到Task任务。
- 该NodeManager 创建容器 Container，并产生MRAppmaster。
- Container 从HDFS上拷贝资源到本地
- MRAppmaster 向 RM申请运行MapTask 资源。
- RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager 分别领取任务并创建容器。
- MR 向两个接收到任务的NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动 MapTask，MapTask 对数据分区排序。
- MrAppMaster 等待所有 MapTask 运行完毕之后，向 RM 申请容器，运行 ReduceTask 。
- ReduceTask 向 MapTask 获取相应分区的数据
- 程序运行完毕之后，MR 会向 RM 申请注销自己。

![](/图片/Yarn提交过程.png)

作业提交全过程详解 

（1）作业提交 

第 1 步：Client 调用 job.waitForCompletion 方法，向整个集群提交 MapReduce 作业。 

第 2 步：Client 向 RM 申请一个作业 id。 

第 3 步：RM 给 Client 返回该 job 资源的提交路径和作业 id。

 第 4 步：Client 提交 jar 包、切片信息和配置文件到指定的资源提交路径。

 第 5 步：Client 提交完资源后，向 RM 申请运行 MrAppMaster。 

（2）作业初始化 

第 6 步：当 RM 收到 Client 的请求后，将该 job 添加到容量调度器

第 7 步：某一个空闲的 NM 领取到该 Job。 

第 8 步：该 NM 创建 Container，并产生 MRAppmaster。

 第 9 步：下载 Client 提交的资源到本地。

（3）任务分配 

第 10 步：MrAppMaster 向 RM 申请运行多个 MapTask 任务资源。

 第 11 步：RM 将运行 MapTask 任务分配给另外两个 NodeManager，另两个 NodeManager 分别领取任务并创建容器。 

（4）任务运行 

第 12 步：MR 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动 MapTask，MapTask 对数据分区排序。 

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。 

第 14 步：ReduceTask 向 MapTask 获取相应分区的数据。

 第 15 步：程序运行完毕后，MR 会向 RM 申请注销自己。 

（5）进度和状态更新 YARN 中的任务将其进度和状态(包括 counter)返回给应用管理器, 客户端每秒(通过 mapreduce.client.progressmonitor.pollinterval 设置)向应用管理器请求进度更新, 展示给用户。 

（6）作业完成 除了向应用管理器请求作业进度外, 客户端每 5 秒都会通过调用 waitForCompletion()来 检查作业是否完成。时间间隔可以通过 mapreduce.client.completion.pollinterval 来设置。作业 完成之后, 应用管理器和 Container 会清理工作状态。作业的信息会被作业历史服务器存储 以备之后用户核查。
