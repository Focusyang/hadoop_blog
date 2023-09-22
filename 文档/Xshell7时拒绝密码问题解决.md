#### Xshell7时拒绝密码问题解决

> xshell远程连接时：SSH服务器拒绝了密码，请再输入一次

###### 1.首先，安装(更新)并开启sshd服务

   Ubuntu中默认关闭sshd服务。

```
sudo apt-get install openssh-server
service sshd restart
```

###### 2.在虚拟机终端中打开sshd_config文件

`  sudo vim /etc/ssh/sshd_config`

###### 3.修改sshd_config配置文件

按i进入编辑模式，修改打开的配置文件，找到#Authentication:(注意：此行不做修改) 做修改如下所示。

![image-20230624160726192](..\图片\修改sshd_config配置信息.png)

```
an#取消这几行的注释
LoginGraceTime 2m          #登陆时间
PermitRootLogin yes        #允许root登录
StrictModes yes            #严格模式
```

​    在这三行下增加一条：

```
PasswordAuthentication yes
```

###### 4.按Esc+：键入wq! 保存并退出

###### 5.重启ssh服务

```
sudo service ssh restart
```

###### 6.注意

![image-20230624161541626](..\图片\master节点.png)

使用Xshell7与虚拟机建立连接时应注意：

- 用户名就是虚拟机终端中@master前的名称

- 建立的会话名称可以任意

- 默认端口号22

  ![image-20230624162335645](..\图片\shell接连界面.png)