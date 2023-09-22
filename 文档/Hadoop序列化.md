## 序列化

## 1.1定义

**序列化：**就是把内存中的对象，转换成字节序列以便存储到磁盘和网络传输

**反序列化：**就是将收到的字节序列或者是磁盘的持久化数据，转换成内存中的对象

**与Java序列化的区别：**

Java 的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带 很多额外的信息（各种校验信息，Header，继承体系等），不便于在网络中高效传输。所以， Hadoop 自己开发了一套序列化机制（Writable）。

## 1.2 自定义bean对象实现序列化接口(Writable)

> 实现过程

（1）必须实现Writable接口

（2）反序列化时，需要反射调用空参构造函数，所以必须要有空参构造

`public FlowBean() { super(); }`

（3）重写序列化方法

```java
@Override
public void write(DataOutput out) throws IOException {
out.writeLong(upFlow);
out.writeLong(downFlow);
out.writeLong(sumFlow);
}

```

（4）重写反序列化方法

```java
@Override
public void readFields(DataInput in) throws IOException {
upFlow = in.readLong();
downFlow = in.readLong();
sumFlow = in.readLong();
}

```

（5）**反序列化的顺序和序列化的顺序完全一致，且变量需要初始化**：避免出现空指针异常

（6）要想把结果显示在文件中，需要重写toString() 方法

（7）如果需要将自定义的bean放在key中传输，还要实现Comparable接口，因为MapReduce框架中的Shuffle 过程要求对Key能排序

```java
@Override
public int compareTo(FlowBean o) {
// 倒序排列，从大到小
return this.sumFlow > o.getSumFlow() ? -1 : 1;
}

```

