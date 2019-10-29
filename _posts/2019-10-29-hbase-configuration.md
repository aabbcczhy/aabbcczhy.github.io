---
layout: post
title: HBase的安装与配置
date: 2019-10-29
author: aabbcczhy
categories: 大数据
tags: HBase 大数据
---
## HBase的安装与配置

>因为HBase的版本需要同Hadoop版本兼容，所以请慎重选择两者的版本，确保它们可以兼容。这里我使用的是Hadoop 3.2.0和HBase 2.2.1，如果你使用了其他版本，请参考[官方文档](https://hbase.apache.org/book.html)，确保版本可以兼容。

### 一、获取HBase

渠道有两个，一个是直接打开浏览器，到[官网](https://hbase.apache.org/downloads.html)下载，一个是直接使用终端利用wget命令下载，两者其实差不多，根据个人喜好，这里我使用wget方式。
```shell
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.2.1/hbase-2.2.1-bin.tar.gz
$ tar zxvf hbase-2.2.1-bin.tar.gz -C /usr/local
$ cd /usr/local
$ mv hbase-2.2.1 hbase
```
下载完，解压缩即可，最好改下名字方便操作。

### 二、配置HBase

1. 使用HBase的前提是JDK1.8，没错就是1.8，其他版本的JDK或多或少都有问题，请慎用，确保你的电脑已经下载了JDK并配置好了环境变量
2. 需要配置的地方有两个，一个是conf目录下的`hbase-site.xml`,另一个是`hbase-env.sh`
    ##### 配置 hbase-env.sh
    ```shell
    export JAVA_HOME=/usr/local/jdk1.8.0_221
    export HBASE_MANAGES_ZK=true
    export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"
    ```
    > 1.JAVA_HOME是JDK的目录所在，不多做解释；
    > 2.HBASE_MANAGES_ZK设置为true表示使用HBase自带的zookeeper，如果单独安装了zookeeper，那么将其设置为false即可；
    > 3.最后一项的意义是让HBase不要扫描Hadoop的library，因为HBase的某些jar包与Hadoop的jar包存在冲突，加上这一项可以避免报错和警告。
    ##### 配置 hbase-site.xml（伪分布式）
    ```xml
    <configuration>
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://localhost:9000/hbase</value>
        </property>
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
    </configuration>
    ```
    >注意：端口号要和你的Hadoop配置的HDFS端口号一致，我这里是9000
3. 配置HBase环境变量(可选)
> 我使用的是ubuntu，其他Linux发行版请按照相应的配置文件进行配置。
```shell
$ vim ~/.bashrc

#.bashrc
export PATH=$PATH:/usr/local/hbase/bin
```
4. 启动HBase
```shell
$ start-dfs.sh && start-hbase.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [aabbcczhy-Inspiron-5576]
localhost: running zookeeper, logging to /usr/local/hbase/bin/../logs/hbase-hadoop-zookeeper-aabbcczhy-Inspiron-5576.out
running master, logging to /usr/local/hbase/logs/hbase-hadoop-master-aabbcczhy-Inspiron-5576.out
: running regionserver, logging to /usr/local/hbase/logs/hbase-hadoop-regionserver-aabbcczhy-Inspiron-5576.out
```
5. 至此，HBase的安装与配置就完成了，可以打开hbase的shell进行测试
```shell
$ hbase shell
hbase(main):001:0> create 't1','info'
Created table t1
Took 3.0611 seconds                                                             
=> Hbase::Table - t1
hbase(main):002:0> put 't1','row1','info','aabbcczhy'
Took 0.2799 seconds
hbase(main):003:0> scan 't1'
ROW                   COLUMN+CELL                                               
 row1                 column=info:, timestamp=1572339984781, value=aabbcczhy    
1 row(s)
Took 0.0499 seconds  
hbase(main):004:0> disable 't1'
Took 1.3555 seconds                                                             
hbase(main):005:0> drop 't1'
Took 0.5080 seconds 
```
