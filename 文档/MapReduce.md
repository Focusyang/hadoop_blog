# MapReduce概述

## 1.1定义

MapReduce是一个**分布式运算程序的编程框架**，是用户开发“基于Hadoop的数据分析的应用”的核心框架

MapReduce核心功能是将用户编写的**业务逻辑代码**和**自带默认组件**组合成一个完整的分布式运算程序

## 1.2MapReduce 优缺点

> **优点**

1. MapReduce易于编程：只需要结合业务逻辑实现一些接口就可以完成一个分布式程序
2. 良好的扩展性：可以通过增加服务器来扩展运算能力
3. 高容错性：运行中的一个机器挂掉，它能够将计算任务转移到另外一个节点上运行
4. 适合PB级以上的海量数据的离线处理

> **缺点**

1. 不擅长实时计算 （MySQL）

2. 不擅长流式计算 ：MapReduce的输入数据是静态的

3. 不擅长DAG（有向无环图）计算

   多个程序之间存在依赖关系，会导致MapReduce作业的输出结果都要写到磁盘，导致性能低下。

## 1.3 MapReduce 核心思想

![mapreduce核心思想](/图片/MapReduce核心思想.png)

（1）分布式的运算程序往往需要分成至少 2 个阶段。 

（2）第一个阶段的 MapTask 并发实例，完全并行运行，互不相干。

 （3）第二个阶段的 ReduceTask 并发实例互不相干，但是他们的数据依赖于上一个阶段 的所有 MapTask 并发实例的输出。

 （4）MapReduce 编程模型只能包含一个 Map 阶段和一个 Reduce 阶段，如果用户的业 务逻辑非常复杂，那就只能多个 MapReduce 程序，串行运行。 总结：分析 WordCount 数据流走向深入理解 MapReduce 核心思想。

**MapReduce进程**

> 1.MrAppMaster : 负责整个程序的过程调度及状态协调
>
> 2.MapTask：负责Map阶段的整个数据处理流程。
>
> 3.Reduce：负责Reduce整个阶段数据处理流程

## 1.4 编程规范

**1.Mapper阶段**

- 用户自定义的Mapper要继承自己的父类
- Mapper的输入数据是KV对的形式（KV的类型可以自定义）
- Mapper中的业务逻辑写在map方法中
- Mapper的输出数据是KV对的形式（KV类型可以自定义）
- **map（）方法对每一个<K,V>调用一次**

**2.Reduce阶段**

- 用户自定义的Reducer要继承自己的父类
- Reducer的输入数据是KV对的形式（KV的类型可以自定义）
- Reducer中的业务逻辑写在reduce方法中
- Reducer的输出数据是KV对的形式（KV类型可以自定义）
- **ReduceTask进程对每一组相同K的<K,V>组调用一次reduce（）方法**

**3.Driver阶段**

相当于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是封装了MapReduce程序相关运行参数的JOB对象。q
