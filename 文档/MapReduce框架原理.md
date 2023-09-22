# MapReduce框架原理

![mapreduce框架原理](/图片/Mapreduce框架原理.png)

## 1.1 InputFormat 数据输入

### 1.1.1 切片与MapTask 并行度决定机制

**数据块与数据切片**

​	**数据块：**Block是HDFS物理上把数据分成一块一块。数据块是HDFS存储数据单位

​	**数据切片：**数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。数据切片是MapReduce 程序计算输入数据的单位，一个切片会对应启动一个MapTask。

![数据切片](/图片/切片与MapTask并行度决定机制.png)

### 1.1.2 Job提交流程源码和切片源码解析

- Job提交流程源码

  ```java
  waitForCompletion()
  submit();
  // 1 建立连接
  connect();
  // 1）创建提交 Job 的代理
  new Cluster(getConfiguration());
  // （1）判断是本地运行环境还是 yarn 集群运行环境
  initialize(jobTrackAddr, conf); 
  // 2 提交 job
  submitter.submitJobInternal(Job.this, cluster)
  // 1）创建给集群提交数据的 Stag 路径
  Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);
  // 2）获取 jobid ，并创建 Job 路径
  JobID jobId = submitClient.getNewJobID();
  // 3）拷贝 jar 包到集群
  copyAndConfigureFiles(job, submitJobDir);
  rUploader.uploadFiles(job, jobSubmitDir);
  // 4）计算切片，生成切片规划文件
  writeSplits(job, submitJobDir);
  maps = writeNewSplits(job, jobSubmitDir);
  input.getSplits(job);
  // 5）向 Stag 路径写 XML 配置文件
  writeConf(conf, submitJobFile);
  conf.writeXml(out);
  // 6）提交 Job,返回提交状态
  status = submitClient.submitJob(jobId, submitJobDir.toString(), 
  job.getCredentials());
  
  ```

  ![Job提交流程源码分析](/图片/Job提交流程源码解析.png)

### 1.1.3 FileInputFormat 切片源码分析

![](/图片/FileInputFormat*.png)

### 1.1.4 FileInputFormat 切片机制

![](/图片/FileInputFormat*.png)

### 1.1.5 FileInputFormat切片大小的参数配置

1. 源码中计算切片大小的公式

   **Math.max(miniSize,Math.min(maxSize,blockSize));**

   mapreduce.input.fileinputformat.split.minisize=1默认值为1

   mapreduce.input.fileinputformat.split.maxsize=Long.MaxValue 默认值Long.MaxValue

   > 默认情况下，切片大小=blocksize

2. 切片大小的设置

   maxsize（切片最大值）：参数如果调得比blockSize小，则会让切片变小，而且就等于配置的这个参数的值。 

   minsize（切片最小值）：参数调的比blockSize大，则可以让切片变得比blockSize还大。

3. 获取切片信息API

```java
//获取切片的文件名称
String name = inputSplit.getPath().getName();
//根据文件的类型获取切片信息
FileSplit inputSplit = (FileSplit) context.getInputSpit();
```

### 1.1.6 TextInputFormat

- TextInputFormat

TextInputFormat 是默认的 FileInputFormat 实现类。按行读取每条记录。键存储该行在整个文件中的起始字节偏移量LongWritable 类型。值是这行的内容，不包括任何行终止 符（换行符和回车符），Text 类型。

举例：

4行文本记录及其对应的每条记录的键值对

```
Rich learning form
Intelligent learning engine
Learning more convenient
From the real demand for more close to the enterprise
```

```
(0,Rich learning form)
(20,Intelligent learning engine)
(49,Learning more convenient)
(74,From the real demand for more close to the enterprise)
```

## 1.2 CombineTextInputFormat 切片机制

框架默认的 TextInputFormat 切片机制是对任务按文件规划切片，不管文件多小，都会 是一个单独的切片，都会交给一个 MapTask，这样如果有大量小文件，就会产生大量的 MapTask，处理效率极其低下。

- **应用场景**

  CombineTextInputFormat 用于小文件过多的场景，它可以将多个小文件从逻辑上规划到 一个切片中，这样，多个小文件就可以交给一个 MapTask 处理。

- **虚拟存储切片最大值设置**

  CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m 

  注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

- **切片机制**

生产切片过程主要包括：**虚拟存储过程**和**切片过程**两部分

![combineTextInputFormat](/图片/CombineTextInputFormat切片机制.png)

1. 虚拟存储过程

   将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不 大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍， 那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时 将文件均分成 2 个虚拟存储块（防止出现太小切片）。

   例如：setMaxInputSplitSize 值为 4M，输入文件大小为 8.02M，则先逻辑上分成一个 4M。剩余的大小为 4.02M，如果按照 4M 逻辑划分，就会出现 0.02M 的小的虚拟存储 文件，所以将剩余的 4.02M 文件切分成（2.01M 和 2.01M）两个文件。

2. 切片过程

   （a）判断虚拟存储的文件大小是否大于 setMaxInputSplitSize 值，大于等于则单独 形成一个切片。 

   （b）如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。 

   （c）测试举例：有 4 个小文件大小分别为 1.7M、5.1M、3.4M 以及 6.8M 这四个小 文件，则虚拟存储之后形成 6 个文件块，大小分别为： 1.7M，（2.55M、2.55M），3.4M 以及（3.4M、3.4M） 最终会形成 3 个切片，大小分别为： （1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M
