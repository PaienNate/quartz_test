---
title: HDFS的高级特性
date: 2024-02-15 10:13:38
tags:
    - hadoop
    - 大数据
category: 大数据
---
## 回收站
默认回收站是关闭的，可以通过在core-site.xml中添加fs.trash.interval来打开配置时间阈值。
删除文件时，是放入回收站/trash
回收站文件可以快速恢复，可以设置一个时间，超过该时间将会彻底删除文件，并且释放占用的数据块。
（阈值是分钟）

```xml
<property>
<name>fs.trash.interval</name>
<value>1440</value>
</property>
```

提示从Delete修改为： Moved: 'hdfs://hadoop1.master:9000/output' to trash at: hdfs://hadoop1.master:9000/user/root/.Trash/Current/output
如果需要恢复，使用mv或者cp恢复即可。没有正经的恢复方式。
Oracle使用闪回的方式返回。
## 配额
名称配额规定文件个数，设置N个，该目录下最多存放N-1个文件或者目录；
空间配额规定的是某个HDFS目录下文件的大小，设置HDFS目录的空间配额是200M就只能存放200M以下的文件
```bash
hdfs dfsadmin -setQuota N <directory>...<directory>
hdfs dfsadmin clrQuota <directory>...<directory>
# 设置空间配额
hdfs dfsadmin -setSpaceQuota <quota> <dirname>...<dirname>
hdfs dfsadmin -clrSpaceQuota <dirname>...<dirname>
```
默认**没有设置，需要用管理员命令设置**
置空间配额是设置的一个逻辑单位，并不是物理大小
如设置空间配额为1M，但上传一个小于1M的文件，虽然它存放时按照自己的大小存储，但它至少占用一个数据块，一个数据块128M，故配额超出
## 快照
快照的本质就是拷贝，防止用户错误操作，备份，试验或测试，以及灾难恢复。默认是关闭的。

```bash
# 开启快照
hdfs dfsadmin -allowSnapshot /input
# 创建快照
hdfs dfs -createSnapshot /input backup_input_01
# 查看快照
hdfs lsSnapshottableDir
# 对比快照
hdfs snapshotDiff /input backup input ol backup input 02
# 恢复快照
hdfs dfs -cp /input/.snapshot/backup input 01/data.txt /input
```

会被拷贝到隐藏路径`.snapshot`下。web ui可以看到，同时也有对比和恢复

## 安全模式和权限
安全模式是hadoop的一种保护机制，用于保证集群中的数据块的安全性。如果HDFS处于安全模式，则表示HDFS是只读状态。

当集群启动的时候，会首先进入安全模式。当系统处于安全模式时会检查数据块的完整性。

假设我们设置的副本数(即参数dfsreplication) 是5，那么在datanode上就应该有5个副本存在。

假设只存在3个副本，那么比例就是3/5=0.6。在配置文件hdfs-default.xml中定义了一个最小的副本的副本率0.999。我们的副本率0.6明显小于0.99，因此系统会自动的复制副本到其他的dataNode.使得副本率不小于0.999.如果系统中有8个副本，超过我们设定的5个副本，那么系统也会删除多余的3个副本。
多退少补，这个配置在hdfs-default.xml
权限和linux的权限设计完全一致，rwx对应421，完全一致。