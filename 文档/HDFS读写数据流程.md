# HDFS读写数据流程

## 写数据

![写数据](/图片/HDFS写数据流程.png)

1. 客户端通过Distribute FileSystem 模块向NameNode请求上传文件，NameNode检查目标文件是否已经存在，父目录是否存在
2. NameNode 返回是否可以上传
3. 客户端请求第一个Block（0-128MB）上传到哪几个DataNode服务器上
4. NameNode返回3个DataNode节点，分别为dn1、dn2、dn3
5. 客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求后会继续调用dn2.然后dn2调用dn3，将这个通信管道建立完成
6. dn1、dn2、dn3逐级应答客户端
7. 客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个pocket就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答
8. 当一个Block传输完成后，客户端再次请求NameNode上传第二个Block的服务器（重复3-7）

## 读数据

![读数据](/图片/HDFS读数据流程.png)

1. 客户端通过 DistributedFileSystem 向NameNode请求下载文件，NameNode通过查询元数据，找到文件块儿所在的DataNode地址。
2. FSDataInputStream挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据
3. DataNode开始传输数据给客户端(从磁盘里面读取数据输入流，以Packet为单位来校验数据)
4. 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。
