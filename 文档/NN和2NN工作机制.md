## NameNode和SecondaryNameNode

![](/图片/NameNode工作机制.png)

**1）第一阶段：NameNode启动**

1. 第一次启动NameNode 格式化以后，创建Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
2. 客户端对元数据进行增删改查的请求
3. NameNode记录操作日志，更新滚动日志
4. NameNode在内存中对元数据进行增删改查

**2）第二阶段：Secondary NameNode工作**

1. Secondary NameNode 询问 NameNode 是否需要CheckPoint。直接带回NamNode是否检查结果
2. Secondary NameNode 请求执行CheckPoint。
3. NameNode滚动正在写的Edits日志。
4. 将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode
5. Secondary NameNode 加载编辑日志和镜像文件到内存，并合并
6. 生成新的镜像文件fsimage.chkpoint
7. 拷贝fsimage.chkpoint 到 NameNode
8. NameNode 将 fsimage.chkpoint 重命名为fsimage
