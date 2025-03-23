---
title: HDFS数据上传和下载的原理
date: 2024-02-14 10:14:06
tags: 
  - 大数据
  - hadoop
category: 大数据
---

## 下载文件

![【Hadoop】HDFS操作、数据上传与下载原理解析、高级特性及底层原理_hadoop_32](https://pic.firehomework.top/2025/03/a72318a5c39263e9e57910b070b998ab.webp)

下载流程：

1. 客户端创建一个DistributedFileSystem作为HDFS客户端对象
2. 客户端创建一个DFSClient对象，与HDFS建立RPC通讯
3. DFSClient对象借助RPC通讯，获取服务器上NameNode的代理对象。
4. 通过代理对象，试图获取文件的元信息。此时元信息将会在HDFS端进行处理
5. HDFS通过IO读取fsimage文件，从中获取到正确的元信息。
6. 元信息返回给客户端对象，根据元信息，其创建一个数据流，开始下载第一个数据块
7. 最后循环下载所有的数据块完成。

注意：**元信息一个150字节，尽量将文件压缩而不是上传大量小文件，尤其是适合压缩成128M的文件**

## 上传文件



