---
title: Hadoop-Hadoop体系架构及其部署模式
date: 2024-02-05 22:26:39
tags: 
    - 大数据
    - hadoop
category: 大数据
---
## 体系架构
Hadoop的体系架构分为三大块，分别是HDFS,YARN,HBase。这三个部分都是主从架构，而主从架构就面临单点故障的问题。

> 什么是单点故障？

单点故障（英语：single point of failure，缩写SPOF）是指系统中一点失效，就会让整个系统无法运作的部件，换句话说，单点故障即会整体故障。

在HDFS,Yarn或者HBase中，如果主节点宕机，（HDFS:NameNode,Yarn:ResourceManager,Hbase:HMaster），则整体就会宕机，所以他们都面临单点故障的问题。

HDFS分为：
1. NameNode:主节点
2. SecondaryNameNode:第二名称节点-日志
3. DataNode:从节点-数据节点

Yarn分为：
1. ResourceManager:主节点
2. NodeManager:从节点，用来执行任务

HBase分为：
1. HMaster:主节点
2. RegionServer:从节点
3. **ZooKeeper**:可以当成数据库

### 关于NameNode
NameNode主节点的功能：
1. 管理HDFS，接收客户端的请求（如数据上传和下载）
2. 负责维护HDFS：维护edits文件（客户端的操作日志）、维护fsimage文件（元信息文件）

edits文件和fsimage文件都存放在hadoop的tmp目录下。

edits保存操作日志，带有*edits_inprogress*的日志是当前正在操作的日志。

日志是二进制存储，需要使用指令查看：

```bash
hdfs oev -i edits-file -o path/to/name.xml, 
```

可以通过其中存放的信息来恢复文件。

fsimage文件记录的是数据块的位置信息和冗余信息。在hadoop1里，数据块默认为64M，hadoop2为128M，可以通过修改参数来进行配置。

若上传文件较大时，将会切分为多个数据块。HDFS也提供元信息的查看器，可以将元信息转换为文本。使用-p命令可以转换为更方便查看的XML。


```bash
hdfs oiv -i 文件 -o 输出名称带路径 -p xml（转换为XML） 
```

XML里的内容比较丰富，比如记录的数据块多少，冗余度等相关信息。

数据块的位置:data目录，数据块都以blk的方式保存。blk有一个带数字的，和带meta后缀的文件。meta是元信息文件，不带meta的是数据信息。数据不到一个数据块时，会以它本身的大小存储，但占用一个数据块，所以数据块是一个逻辑单位。

设置数据块冗余度的一般原则：冗余度和数据节点个数一致，最大不超过3.超过3可以，但会导致很大的浪费。

## 部署Hadoop的方式
### 本地模式
1. 特点
  a. 没有HDFS和Yarn	
  b. 只能测试MapReduce程序，作为一个普通的Java程序
  c. 处理的数据是本地Linux的文件
  d. 用于开发和测试
2. 配置
  ○ 只需要配置一个文件：hadoop-env.sh,设置它的JAVA_HOME环境变量即可。
  ○ 运行个MapReduce是没问题的。不过看上去一般人用不上。
### 伪分布模式
1. 特点：
  a. 在单机上模拟一个分布式环境
  b. 具有所有的功能
    ⅰ. HDFS：NameNode,DataNode,SecondaryNameNode
    ⅱ. Yarn:ResourceManager,NodeManager
  c. 用于开发和测试，本地模式需要HDFS。
  d. 现在强制需要使用无密登录。
2. 配置
需要配置多个文件。
core-site.xml:
```xml
<configuration>
<property>
<!-- 配置NameNode的地址，即主节点的地址 -->
<!-- 9000是RPC通信的端口 -->
	<name>fs.defaultFS</name>
	<value>hdfs://hadoop1.master:9000</value>
</property>
<property>
<!-- HDFS对应的操作系统目录，生产系统一定要修改，
默认是Linux的tmp目录，Linux重新启动会导致它被删除 -->
	<name>hadoop.tmp.dir</name>
	<value>/usr/local/hadoop/hadoop_temp/tmp</value>
</property>
</configuration>
```
yarn-site.xml
```xml
<!-- 配置resourcemanager的地址，即yarn的主节点，以供提交任务 -->
	<name>yarn.resourcemanager.hostname</name>
	<value>hadoop1.master</value>
</property>

<property>
<!-- 配置mapreduce，以洗牌的方式进行任务执行 -->
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
```
mapred-site.xml:
```xml
<property>
<!-- 配置mapreduce使用yarn作为框架，默认好像是local. -->
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>
```
hdfs-site.xml:
```xml
<property>
<!-- 数据块的冗余度，默认为3，就是一份数据保三份-->
<!-- 数据块的冗余度，有几个数据节点就配置几个，但最大不超过3 -->
	<name>dfs.replication</name>
	<value>1</value>
</property>
<!-- 还可以通过dfs.permissions配置关闭权限检查，但生产环境中，可能有需要。-->
```
### 全分布模式
1. 特点
  a. 真正的分布式环境
  b. 具备hadoop所有功能
  c. 至少需要三台机器
构成一个主从式架构，一个主节点，两个从节点。
需要做准备工作：关闭防火墙，设置主机名，配置免密码登录（两两之间），安装JDK，同步时间（使用NTP服务器）。
需要修改的部分：
1. 伪分布模式修改数据块的冗余度为数据节点个数（如三台机器，设置为2）
2. 配置slaves（hadoop3为workers文件），填入从节点的主机名或IP
### 特殊问题和分析
启动hadoop时见到如下的报错：
> ERROR: Attempting to operate on hdfs namenode as root
> ERROR: but there is no HDFS_NAMENODE_USER defined. 
> Aborting operation.

根据StackOverFlow的说法：
> Hadoop 安装是为不同的用户进行的，而您启动 YARN 服务却使用了不同的用户。或者在 Hadoop 配置的 hadoop-env.sh 中指定的 HDFS_NAMENODE_USER 和 HDFS_DATANODE_USER 用户是其他用户。因此，我们需要在每个地方进行更正并保持一致。因此，解决这个问题的一个简单方法是编辑您的 hadoop-env.sh 文件，并添加您想要启动 YARN 服务的用户名。

解决的方案：添加：
```bash
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```

但这个解释并不太让我满意。我没有配置过HDFS_NAMENODE_USER，一直使用的也是root账户，似乎并不是stackoverflow上解释的说法。

所以我找到了官网配置的地址：https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/UnixShellGuide.html

官网的描述是这样的：

>   Apache Hadoop提供了一种按子命令进行用户检查的方法。虽然这种方法很容易被规避，不应被视为安全特性，但它确实提供了一种防止意外发生的机制。例如，设置HDFS_NAMENODE_USER=hdfs将使hdfs namenode和hdfs --daemon start namenode命令验证运行命令的用户是否为hdfs用户，通过检查USER环境变量来实现。这也适用于非守护进程。设置HADOOP_DISTCP_USER=jane将在允许执行hadoop distcp命令之前验证USER是否设置为jane。
>   如果_USER环境变量存在，并且使用特权运行命令（例如，作为root；参见API文档中的hadoop_privilege_check），执行将首先切换到指定的用户。对于支持出于安全原因进行用户帐户切换的命令，并因此具有SECURE_USER变量的命令（请参见下文），基本_USER变量需要是预期用于切换到SECURE_USER帐户的用户。例如：
> HDFS_DATANODE_USER=root
> HDFS_DATANODE_SECURE_USER=hdfs
> 将强制‘hdfs –daemon start datanode’为root，但在特权工作完成后最终会切换到hdfs用户。

根据上面的内容，我的猜测是：如果_USER环境变量存在，并且使用特权运行命令（例如，作为root；参见API文档中的hadoop_privilege_check），执行将首先切换到指定的用户。由于测试使用了特权命令，所以hadoop试图找到一个非特权的用户（读取环境变量）来运行程序（或许和MySQL的不使用root运行差不多），所以我们要强制设置为root(测试环境)，所以网络上搜到的大多数都是要把USER设置为root，因为大部分人遇到这个问题都是在“测试环境”，使用特权账户才会发生。