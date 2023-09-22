# Hadoop入门

## 运行模式

- 本地模式

  数据存储在Linux本地

  **用途：**测试偶尔使用

- 伪分布式

  数据存储在HDFS，一台服务器模拟一个分布式环境

  **用途：**公司比较差钱，数据量很小，使用伪分布式

- 完全分布式

  数据存储在HDFS上/多台服务器工作

  **用途：**多用于企业开发，企业里大量使用

## 分布式环境搭建

1. 准备3台客户机（关闭防火墙、静态IP、主机名称）

   ```Linux
   //关闭防火墙
   systemctl stop firewalld
   systemctl disable firewalld.service
   //静态IP
   vim /etc/sysconfig/network-scripts/ifcfg-ens33
   //主机名称
   vim /etc/hostname
   vim /etc/hosts
   ```

1. 安装JDK
2. 配置环境变量
3. 配置集群
4. 单点启动
5. 配置ssh
6. 群起并测试集群

## rsync远程同步工具

rsync主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

rsync和scp区别：用rsync做文件的复制要比scp的速度快，rsync只对差异文件做更新。scp是把所有文件都复制过去。

```rsync   -av    $pdir/$fname    $user@$host:$pdir/$fname```

脚本文件

```
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi

#2. 遍历集群所有机器
for host in hadoop102 hadoop103 hadoop104
do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送

    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
            then
                #5. 获取父目录
                pdir=$(cd -P $(dirname $file); pwd)

                #6. 获取当前文件的名称
                fname=$(basename $file)
                ssh $host "mkdir -p $pdir"
                rsync -av $pdir/$fname $host:$pdir
            else
                echo $file does not exists!
        fi
    done
done

```

## ssh无密登录

![image-20230903194932698](..\图片\免密登陆原理.png)

1.生成公钥和私钥

```cd .ssh```

```ssh-keygen -t rsa```

2.将公钥拷贝到要免密登陆的目标机器上

```ssh-copy-id hadoop103```

**注意**

还需要在hadoop103上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

还需要在hadoop104上采用atguigu账号配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上。

还需要在hadoop102上采用root账号，配置一下无密登录到hadoop102、hadoop103、hadoop104服务器上

3.**.ssh文件夹下的文件功能解释**

| known_hosts     | 记录ssh访问过计算机的公钥(public key) |
| --------------- | ------------------------------------- |
| id_rsa          | 生成的私钥                            |
| is_rsa.pub      | 生成的公钥                            |
| authorized_keys | 存放授权过的无密登陆服务器公钥        |

## 集群配置

1）集群部署规划

- NameNode和SecondaryNameNode不要安装在同一台服务器
- ResourceManager也很消耗内存，不要和NameNode、SecondaryNameNode配置在同一台机器上。

|      | hadoop102         | hadoop103                   | hadoop104                  |
| ---- | ----------------- | --------------------------- | -------------------------- |
| HDFS | NameNode\DataNode | DataNode                    | SecondaryNameNode\DataNode |
| YARN | NodeManager       | ResourceManager\NodeManager | NodeManager                |

2)配置文件说明

Hadoop配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值。

（1）默认配置文件

| 要获取的默认文件     | 文件存放在Hadoop的jar包中的位置                           |
| -------------------- | --------------------------------------------------------- |
| [core-default.xml]   | hadoop-common-3.1.3.jar/core-default.xml                  |
| [hdfs-default.xml]   | hadoop-hdfs-3.1.3.jar/hdfs-default.xml                    |
| [yarn-default.xml]   | hadoop-yarn-common-3.1.3.jar/yarn-default.xml             |
| [mapred-default.xml] | hadoop-mapreduce-client-core-3.1.3.jar/mapred-default.xml |

(2)自定义配置文件

**core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml**四个配置文件存放在$HADOOP_HOME/etc/hadoop这个路径上，用户可以根据项目需求重新进行修改配置。

3）**配置集群**

- 配置core-site.xml

  ```cd $HADOOP_HOME/etc/hadoop```

  ```vim core-site.xml```

  内容如下

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
      <!-- 指定NameNode的地址 -->
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://hadoop102:8020</value>
      </property>
  
      <!-- 指定hadoop数据的存储目录 -->
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/opt/module/hadoop-3.1.3/data</value>
      </property>
  
      <!-- 配置HDFS网页登录使用的静态用户为atguigu -->
      <property>
          <name>hadoop.http.staticuser.user</name>
          <value>atguigu</value>
      </property>
  </configuration>
  
  ```

  

- 配置hdfs-site.xml

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
  	<!-- nn web端访问地址-->
  	<property>
          <name>dfs.namenode.http-address</name>
          <value>hadoop102:9870</value>
      </property>
  	<!-- 2nn web端访问地址-->
      <property>
          <name>dfs.namenode.secondary.http-address</name>
          <value>hadoop104:9868</value>
      </property>
  </configuration>
  
  ```

  

- 配置yarn-site.xml

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
      <!-- 指定MR走shuffle -->
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
      </property>
  
      <!-- 指定ResourceManager的地址-->
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>hadoop103</value>
      </property>
  
      <!-- 环境变量的继承 -->
      <property>
          <name>yarn.nodemanager.env-whitelist</name>
          <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
      </property>
   <!-- 开启日志聚集功能 -->
  <property>
      <name>yarn.log-aggregation-enable</name>
      <value>true</value>
  </property>
  <!-- 设置日志聚集服务器地址 -->
  <property>  
      <name>yarn.log.server.url</name>  
      <value>http://hadoop102:19888/jobhistory/logs</value>
  </property>
  <!-- 设置日志保留时间为7天 -->
  <property>
      <name>yarn.log-aggregation.retain-seconds</name>
      <value>604800</value>
  </property>
  
  </configuration>
  
  ```

  

- 配置mapred-site.xml

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
  	<!-- 指定MapReduce程序运行在Yarn上 -->
      <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
      </property>
    <!-- 历史服务器端地址 --> 
   <property>
      <name>mapreduce.jobhistory.address</name>
      <value>hadoop102:10020</value>
  </property>
  
  <!-- 历史服务器web端地址 -->
  <property>
      <name>mapreduce.jobhistory.webapp.address</name>
      <value>hadoop102:19888</value>
    
  </configuration>
  
  ```

  

4)**分发配置好的Hadoop文件**

## 启动集群

- **如果集群是第一次启动**，需要在hadoop102节点格式化NameNode（注意：格式化NameNode，会产生新的集群id，导致NameNode和DataNode的集群id不一致，集群找不到已往数据。如果集群在运行过程中报错，需要重新格式化NameNode的话，一定要先停止namenode和datanode进程，并且要删除所有机器的**data**和**logs**目录，然后再进行格式化。）

- 格式化namenode

  ```hdfs namenode -format```

- 启动hdfs ```sbin/start-dfs.sh```
- 启动yarn ```sbin/start-yarn.sh```

## 编写Hadoop集群常用脚本

**启动集群脚本**

```
#!/bin/bash

if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit ;
fi

case $1 in
"start")
        echo " =================== 启动 hadoop集群 ==================="

        echo " --------------- 启动 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
        echo " --------------- 启动 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
        echo " --------------- 启动 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
        echo " =================== 关闭 hadoop集群 ==================="

        echo " --------------- 关闭 historyserver ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
        echo " --------------- 关闭 yarn ---------------"
        ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
        echo " --------------- 关闭 hdfs ---------------"
        ssh hadoop102 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
    echo "Input Args Error..."
;;
esac

```

```chmod +x myhadoop.sh```赋予执行权限

**查看三台服务器进程脚本**

```
#!/bin/bash

for host in hadoop102 hadoop103 hadoop104
do
        echo =============== $host ===============
        ssh $host jps 
done

```

```chmod +x jpsall```赋予脚本执行权限

**分发脚本**

```xsync /home/atguigu/bin/```

**报错处理**

[分发文件报错]: https://qastack.cn/unix/12203/rsync-failed-to-set-permissions-on-error-with-rsync-a-or-p-option



```
rsync: failed to set permissions on "/ata/text/RCS/jvlc,v": Operation not permitted (1)
rsync: failed to set permissions on "/ata/text/RCS/jvm,v": Operation not permitted (1)
rsync: failed to set permissions on ...
```

```
rsync -ahv --no-o --no-g /home/atguigu/bin/
```

## 常见端口号及其Web页面

| 端口名称                   | Hadoop2.x | Hadoop3.X      |
| -------------------------- | --------- | -------------- |
| NameNode内部通信端口       | 8020/9000 | 8020/9000/9820 |
| NameNode HTTP UI           | 50070     | 9870           |
| MapReduce 查看执行任务端口 | 8088      | 8088           |
| 历史服务器通信端口         | 19888     | 19888          |

## 集群时间同步

如果服务器在公网环境（能连接外网），可以不采用集群时间同步，因为服务器会定期和公网时间进行校准；

如果服务器在内网环境，必须要配置集群时间同步，否则时间久了，会产生时间偏差，导致集群执行任务时间不同步。

**1**）需求

找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，生产环境根据任务对时间的准确程度要求周期同步。测试环境为了尽快看到效果，采用1分钟同步一次。

  ![image-20230904112757787](..\图片\集群时间同步.png)                               

**2**）时间服务器配置（必须root用户）

**（1）查看所有节点ntpd服务状态和开机自启动状态**

[atguigu@hadoop102 ~]$ sudo systemctl status ntpd

[atguigu@hadoop102 ~]$ sudo systemctl start ntpd

[atguigu@hadoop102 ~]$ sudo systemctl is-enabled ntpd

**（2）修改hadoop102的ntp.conf配置文件**

[atguigu@hadoop102 ~]$ sudo vim /etc/ntp.conf

修改内容如下

（a）修改1（授权192.168.10.0-192.168.10.255网段上的所有机器可以从这台机器上查询和同步时间）

\#restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap

为restrict 192.168.10.0 mask 255.255.255.0 nomodify notrap

  b）修改2（集群在局域网中，不使用其他互联网上的时间）

server 0.centos.pool.ntp.org iburst

server 1.centos.pool.ntp.org iburst

server 2.centos.pool.ntp.org iburst

server 3.centos.pool.ntp.org iburst

为

**#**server 0.centos.pool.ntp.org iburst

**#**server 1.centos.pool.ntp.org iburst

**#**server 2.centos.pool.ntp.org iburst

**#**server 3.centos.pool.ntp.org iburst

（c）添加3（当该节点丢失网络连接，依然可以采用本地时间作为时间服务器为集群中的其他节点提供时间同步）

server 127.127.1.0

fudge 127.127.1.0 stratum 10

**（3）修改hadoop102的/etc/sysconfig/ntpd 文件**

[atguigu@hadoop102 ~]$ sudo vim /etc/sysconfig/ntpd

增加内容如下（让硬件时间与系统时间一起同步）

SYNC_HWCLOCK=yes

**（4）重新启动ntpd服务**

[atguigu@hadoop102 ~]$ sudo systemctl start ntpd

**（5）设置ntpd服务开机启动**

[atguigu@hadoop102 ~]$ sudo systemctl enable ntpd

**3**）其他机器配置（必须root用户）

（1）关闭所有节点上ntp服务和自启动

[atguigu@hadoop103 ~]$ sudo systemctl stop ntpd

[atguigu@hadoop103 ~]$ sudo systemctl disable ntpd

[atguigu@hadoop104 ~]$ sudo systemctl stop ntpd

[atguigu@hadoop104 ~]$ sudo systemctl disable ntpd

（2）在其他机器配置1分钟与时间服务器同步一次

[atguigu@hadoop103 ~]$ sudo crontab -e

编写定时任务如下：

*/1 * * * * /usr/sbin/ntpdate hadoop102

（3）修改任意机器时间

[atguigu@hadoop103 ~]$ sudo date -s "2021-9-11 11:11:11"

（4）1分钟后查看机器是否与时间服务器同步

[atguigu@hadoop103 ~]$ sudo date

## 入门面试题

### 常用端口号

一、**hadoop3.X**

> HDFS NameNode 内部通常端口：**8020/9000/9820**
>
> HDFS NameNode 对用户的查询端口：**9870**
>
> Yarn查看任务运行的情况端口：**8088**

二、**hadop.2x**

> HDFS NameNode内部通常端口：**8020/9000**
>
> HDFS NameNode对用户的查询端口：**50070**
>
> Yarn查看任务运行情况的：**19888**

### 常用配置文件

3.X **core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml workers**

2.X **core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml slaves**



