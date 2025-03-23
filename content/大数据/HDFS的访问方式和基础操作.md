---
title: HDFS的访问方式和基础操作
date: 2024-02-07 10:40:41
tags: 
    - Hadoop
    - 大数据
category: 大数据
---
## 访问方式
### WebConsole方式

Hadoop3的默认端口：
Namenode:9870
SecondaryNameNode:9868

需要重点关注的是该处的启动描述：

![image.png](https://pic.firehomework.top/2025/03/f99f11cf54a42c590545b1bdcae19cfe.webp)

启动描述中显示了HADOOP的四个阶段：

- 加载元信息文件（fsimage）
- 读取edits日志文件（体现最新的状态）
- 第三阶段：触发检查点，SecondaryNameNode合并edits到fsimage中
- 第四阶段：进入安全模式，检查数据块完整性（此时不能操作hdfs）

### 命令行方式

使用`hdfs dfs/dfsadmin `来进行查询。

DFS常用命令如下：

-mkdir 在hdfs中创建目录，如果父目录不存在，需要使用-p的参数先创建父目录再创建子目录

-ls 直接查看某个目录

-ls R 查看某个目录，包含子目录，简写的方式为-lsr（注意：hadoop3中已经标记为废弃，最好使用ls R）

-copyFromLocal,-moveFromLocal,-put：put和copyFromLocal是一样的，moveFromLocal相当于剪切。

-copyToLocal, -get ：下载数据

-rm 删除数据，实际上是删除一个空目录，

-rmr 删除目录，包含子目录（注意，同样被废弃，现在使用-rm -r）提示：“Deleted /bbb”，如果启动了回收站，打印的日志会有一些不同，会提示移动到回收站。

-getmerge 合并，先把某个目录下的文件合并（直接拼在一起），然后再下载（提升效率）

（hdfs dfs -merge /students ./allstudents.txt）

-cp:拷贝

-mv:剪切

-count: 显示目录下文件个数和大小。它列出的不够详细

-du:列出的更加详细

-text -cat:查看文本文件的内容

-balancer:平衡操作（这个不需要添加dfs），在全分布模式下，需要让hdfs文件均衡。注意系统空闲的时候使用。

DFSADMIN管理命令如下：

-report: 打印一个报告，观察整体状态，和Web Console下的信息类似。

-safemode:安全模式有关，可以用它查看或者改变。

## 开发形式

需要包含：hadoop common的jar包，hdfs的jar包。可以从lib下取到，或者直接使用maven取得。

开发形式下默认以用户的主机名作为访问用户名，若权限为打开状态，将会报错：

```java
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.security.AccessControlException): Permission denied: user=Administrator, access=WRITE, inode="/":root:supergroup:drwxr-xr-x
```

提示其他用户没有权限。

### 关于权限

Hadoop的权限和Linux权限十分类似。Linux权限的说明如下：

第一个字符表示文件类别：
-：普通文件
d：目录文件
b：块设备文件
c：[字符设备](https://www.baidu.com/s?wd=字符设备&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)文件
l：符号链接文件

如上面drwxr提示现在正在操作目录文件。

后面每三个一组，提示三种用户：文件所有者，同组，其他用户

如上面的：rwx r-x r-x

每一组的3个字符一次表示读、写、执行权限，其中：
r：表示有读权限
w：表示有写权限
x：表示有执行权限
-：表示没有相应的权限

即：文件所有者有读写执行权限，其他两个组有读和执行的权限。另外的一种描述是给rwx分别赋值4 2 1的二进制，故chmod 777代表所有组提供所有权限。

像上面的这种就是755权限。

### 解决权限问题的四种方案

四种不同的方式能解决方案

1. 设置执行用户为root用户（权限管理比较弱），环境变量：HADOOP_USER_NAME可以用于设置。
2. 使用java的-D参数设置环境变量

3. 1. java -D参数：用main arg方法传参，或者使用-D参数设置环境变量。
   1. 用System.getProperty("xxx")可以读取变量
   2. javac xxx.java进行编译，并使用java -Dp1=aaa 主类 来启动。

4. 使用chmod命令先修改权限，比如修改成777.
5. 修改XML配置，直接关闭掉权限参数。（生产环境不推荐）
