---
layout: post
title: Hadoop的安装与配置
date: 2019-10-29
author: Hadoop的安装过程和伪分布式的配置方法
categories: 大数据
tags: Hadoop 大数据
---

## Hadoop的安装与配置

因为Hadoop对Linux操作系统原生支持，因而推荐在Linux上使用Hadoop，Windows当然也可以用，只不过有点麻烦，这里不介绍Windows的配置和使用。

> * 操作系统：Linux Ubuntu 16.04.6 LTS
> * Java：JDK 1.8.0_221
> * Hadoop：3.2.0

### 创建用户

我的建议是专门创建一个用户来管理hadoop，当然你也可以选择跳过这个步骤，直接在你原有的用户上安装和使用hadoop。

```shell
# 创建名为hadoop的用户并指定bash作为shell
$ sudo useradd -m hadoop -s /bin/bash
# 设置密码
$ sudo passwd hadoop
# 为hadoop用户添加管理员权限
$ sudo adduser hadoop sudo
# 切换到Hadoop
$ su -hadoop
```

### 安装JDK

你可以使用apt直接安装JDK，但我更建议使用压缩包方式，因为使用压缩包可以自由选择版本，这里强烈建议你使用**JDK1.8**（我从打代码开始到现在用的都是**JDK1.8**，不要问为什么，就是1.8）。

1. 从[Oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)上下载JDK，这里，我下载的是***jdk1.8.0_221.tar.gz***

2. 解压缩到一个目录下

   ```shell
   $ tar zxvf jdk1.8.0_221.tar.gz -C /usr/local
   ```

3. 配置环境变量，打开~/.bashrc（其他Linux发行版请修改相应的配置文件），添加如下配置

   ```sh
   export JAVA_HOME=/usr/local/jdk1.8.0_221
   export JRE_HOME=${JAVA_HOME}/jre
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
   export PATH=${JAVA_HOME}/bin:$PATH
   ```

4. 最后别忘了`source ~/.bashrc`让配置生效

### SSH登录权限设置

hadoop的名称节点Namenode为了能启动集群中的所有机器，需要配置好无密码登录ssh。

1. 由于ubuntu只有ssh客户端，并没有服务端，首先先安装ssh服务端

   ```shell
   $ sudo apt-get install ssh-server
   ```

2. 切换到~/.ssh目录下并生成SSH密钥

   ```shell
   cd ~/.ssh
   ssh-keygen -t rsa
   ```

   三次回车后，成功生成SSH密钥。

3. 加入授权

   ```shell
   cat ./id_rsa.pub >> ./authorized_keys 
   ```

此时，运行 `ssh localhost` 就无需密码了。

### 安装Hadoop

可以直接从[官网](https://hadoop.apache.org/releases.html)上下载(hadoop，对于hadoop版本其实没有很严格的要求，如果怕后面对于其他软件兼容性不好的话可以选择低一点版本的hadoop，我这边安装的是**hadoop3.2.0.tar.gz**

1. 解压缩到一个目录，为了方便将文件夹更名

   ```shell
   $ sudo tar -zxvf hadoop-3.2.0.tar.gz -C /usr/local/
   $ sudo mv hadoop-3.2.0 hadoop
   ```

2. 将文件夹授权给用户hadoop

   ```shell
   $ sudo chown -hR hadoop /usr/local/hadoop
   ```

3. 修改**hadoop-env.sh**文件，配置JAVA_HOME

   ```shell
   export JAVA_HOME=/usr/local/jdk1.8.0_221
   ```

4. 查看hadoop版本

   ```shell
   $ ./bin/hadoop version
   ```

### Hadoop伪分布式配置

1. 修改配置文件`core-site.xml`

   ```xml
   <configuration>
           <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/local/hadoop/tmp</value>
           </property>
           <property>
                <name>fs.defaultFS</name>
                <value>hdfs://localhost:9000</value>
           </property>
   </configuration>
   ```

2. 修改配置文件`hdfs-site.xml`

   ```xml
   <configuration>
           <property>
               <name>dfs.replication</name>
               <value>1</value>
           </property>
           <property>
               <name>dfs.namenode.name.dir</name>
               <value>file:/usr/local/hadoop/tmp/dfs/name</value>
           </property>
           <property>
               <name>dfs.datanode.data.dir</name>
               <value>file:/usr/local/hadoop/tmp/dfs/data</value>
           </property>
           <property>
               <name>dfs.permissions</name>
               <value>false</value>
           </property>
   </configuration>
   ```

3. 初始化文件系统

   ```shell
   $ ./bin/hadoop namenode -format
   ```

4. 启动集群，并查看所有Java进程

   ```shell
   $ ./sbin/start-dfs.sh && jps
   ```

5. 打开浏览器，输入 http://localhost:9870 访问Hadoop的Web页面

   > 注：在hadoop3.x版本，端口号已经变为9870，不再是2.x版本的50070

至此，Hadoop的安装与配置完成。