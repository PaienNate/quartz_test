---
title: HDFS集群和如何解决单点故障问题
date: 2024-02-15 10:21:29
tags:
    - hadoop
    - 大数据
category: 大数据
---
# 集群
HADOOP的集群两个特点：负载均衡（Federation联盟），失败迁移（就是实现HA）

Federation联盟只有HDFS有，它的思路是使用一个viewFS部署在代理服务器上，然后在多个Active的NameNode中选择一个使用。

![image.png](https://pic.firehomework.top/2025/03/b1e2bd33ef5beb1936dc1f0728b0547c.webp)

实现失败迁移HA的方案是使用ZooKeeper，多个NameNode里只有一个处于Active状态，其余的都是Standby状态，若Active的宕机，可以调用另一个StandBy的，将它“选举”成为Active的。

只有HDFS有联盟，HA是谁都有。