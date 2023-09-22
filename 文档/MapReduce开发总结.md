# MapReduce 开发总结

**输入数据接口：InputFormat**

1. 默认使用的实现类是：TextInputFormat
2. TextInputFormat 的功能逻辑是：一次读一行文本，然后将该行的起始偏移量作为key，行内容作为 value 返回
3. CombineTextInputFormat 可以把多个小文件合并成一个切片处理，提高处理效率。

**逻辑处理接口：Mapper**

1. 用户根据业务需求实现其中的三个方法：map() setup() cleanup()

**Partitioner 分区**

1. 有默认实现 HashPartitioner，逻辑是根据key的哈希值和numReduces 来返回一个分区号：key.hashCode( )&Integer.MAXVALUE%numReduces
2. 如果业务上有特别的需求，可以自定义分区。

**Comparable 排序**

1. 当我们用自定义对象作为key来输出时，就必须要实现WritableComparable 接口，重写其中的 compareTo( )方法。
2. 部分排序：对最终输出的每一个文件进行内部排序。
3. 全排序：对所有数据进行排序，通常只有一个Reducde
4. 二次排序：重写的compareTo( )方法中排序条件有两个

**Combiner 合并**

>  Combiner 合并可以提高程序执行效率 ，减少IO传输，但是不能影响原有的业务处理结果(**通常是在map的输出阶段进行Combinder**)

**逻辑处理接口：Reducer**

> 用户根据业务需求实现其中的三个方法：reduce( ) setup( ) cleanup( )

**输出数据接口：OutputFormat**

1. 默认实现类是 TextOutputFormat，功能逻辑是：将每一个KV对，向目标文本文件输出一行
2. 用户还可以自定义 OutPutFormat				