---
title: Hadoop 项目（2）--基于Pig Latin实验
date: 2021-10-25
updated: 2021-10-25
author: "sofm"
layout: post
tags: 
- Hadoop
- 大数据
category: 大数据
---

## **项目（2）--基于Pig Latin实验**
注：该实验主笔为：@Sofm，感谢他对博客的支持。

#### 1、任务目标

![image-20211025204145227](https://pic.firehomework.top/2025/03/ab7ed385b09262b501920e726e75e623.webp)

#### 2、目标实现

##### （1）将航班数据上传至HDFS

使用start-all.sh启动hadoop

可以先创建一个pig目录

```bash
hadoop fs -mkdir /user/pig
```

![image-20211025204945719](https://pic.firehomework.top/2025/03/90b0ca3cc41371464ed59185dc0ff542.webp)

使用hadoop的put命令将源文件上传到HDFS，也就是我们刚刚创建的文件夹

```bash
hadoop fs -put ~/MapReduceData/1987_600.csv /user/pig/1987_600.csv
```

![image-20211025204819897](https://pic.firehomework.top/2025/03/70d54da234a68835901c70a94dd36d0f.webp)

如果遇到这样的报错，说明hadoop的NameNode安全模式在开启状态，只需要输入以下代码即可关闭安全模式

![image-20211025210048960](https://pic.firehomework.top/2025/03/a7c31eb8bc019e5e18caba1b1eea4dd0.webp)

```bash
hadoop dfsadmin -safemode leave
```

![image-20211025210148022](https://pic.firehomework.top/2025/03/d1b856981ba1f1dbe559a32c0f16015c.webp)

这样，第一个小目标就顺利完成啦！



##### （2）编写Pig程序flight-week-dist.pig

创建一个flight-week-dist.pig文件

```bash
vi flight-week-dist.pig 
```

![image-20211025210508173](https://pic.firehomework.top/2025/03/f1f0e6e9409ea9282294a338f2eddd64.webp)

###### 统计周航班次数

任务目标

![image-20211025210859918](https://pic.firehomework.top/2025/03/f0eb447939b3f4d050c708487a0f98c1.webp)

脚本编写：

```bash
 data = load 'hdfs://zbt:9000/user/pig/1987_600.csv' using PigStorage(',') as (Year,Month,DayofMonth,DayOfWeek,DepTime,CRSDepTime,ArrTime,CRSArrTime,UniqueCarrier,FlightNum,TailNum,ActualElapsedTime,CRSElapsedTime,AirTime,ArrDelay,DepDelay,Origin,Dest,Distance,TaxiIn,TaxiOut,Cancelled,CancellationCode,Diverted,CarrierDelay,WeatherDelay,NASDelay,SecurityDelay,LateAircraftDelay);
```

![image-20211025211125429](https://pic.firehomework.top/2025/03/ae97b2fb5589e914ac8800f166f5b649.webp)

解释：

此load语句是加载"hdfs://zbt:9000/user/pig/1987_600.csv"路径下的文件，你需要根据你自己上传的路径来修改，主机名也不要忘记改哟！

从句using PigStorage(',')是根据逗号来将数据进行分隔

从句as....是为此数据设置表头

```bash
filter_data = filter data by DayOfWeek>=1;
```

![image-20211025212711561](https://pic.firehomework.top/2025/03/42ff201fb6f82e063b29e5274c35893d.webp)

解释：

此filter语句的目的是为了去除这样的表头，以免影响最终结果

![image-20211025212913901](https://pic.firehomework.top/2025/03/f56977ce875f484606b70f06cb0b2af6.webp)

```bash
group_week1 = group filter_data by DayOfWeek;
foreach_group_week1 = foreach group_week1 generate group, COUNT($1);
dump foreach_group_week1
```

![image-20211025213034060](https://pic.firehomework.top/2025/03/ac50baa0412fb8a431d9fde1b9af26cc.webp)

解释：

此group语句是通过周数将data数据进行分组

不必多说，foreach语句是将分好组的数据进行遍历，后面的COUNT($1)是统计每周有多少条数据，统计出来的结果真是我们想要的答案

dump语句是将结果展示出来

![image-20211025231404893](https://pic.firehomework.top/2025/03/63b933de121415cb8e8ed3d6ea1969f0.webp)

解释：

store..into...语句是将结果输出至文本文件中，文件名可以依照自己的喜好修改

输出结果如下：

![image-20211025232034167](https://pic.firehomework.top/2025/03/7abc2640f626082a35bc3af611f9db04.webp)

至此，统计周航班次数的目标就已达成！

###### 统计各航班飞行总里程

任务目标

![image-20211025232242971](https://pic.firehomework.top/2025/03/fff819579cc4c1b824f23ad9e27f9db5.webp)

任务分析：

相信在看这篇文章的同学们，之前一定看过MapReduce的文章。上篇文章已经提到航班号是由UniqueCarrier和FlightNum两列数据组成。而飞行公里数，我们可以从Distance这一列得到。

![image-20211025232527915](https://pic.firehomework.top/2025/03/03cc722a910cdeefcdd7f03049d5d732.webp)

现在摆在我们面前的问题有两个

​	1、我们如何将UniqueCarrier和FlightNum两列数据组成为一列。

​	2、如何将好多好多的飞行公里数加在一起然后呈现出来。

脚本编写：

![image-20211025233027939](https://pic.firehomework.top/2025/03/f3d59bdd716c62482494d007ccde044a.webp)

刚刚我们已经了解了这两行代码的用法，我就不多赘述了。

下面这行代码正是解决问题一的关键

```bash
week_concat = foreach filter_data generate CONCAT (UniqueCarrier, FlightNum) as hangBanHao, Distance;
```



![image-20211025233238067](https://pic.firehomework.top/2025/03/9690c6f517ac833194d77dcbcac56882.webp)

解释：

我们使用了CONCAT()函数，这个函数的作用可以将两列或者多列数据组合在一起

如果你想具体的了解这个函数，你可以去W3Cschool进行更详细的学习，网址如下：

[Apache Pig CONCAT()函数_w3cschool](https://www.w3cschool.cn/apache_pig/apache_pig_concat.html)

从句as的作用是给已经合并为一列的数据列起一个新的名字，以便我们后续对他其他进行操作

```bash
group_week2 = group week_concat by hangBanHao;
foreach_group_week2 = foreach group_week2 generate group, SUM(week_concat.Distance);
dump foreach_group_week2
```



![image-20211025234014699](https://pic.firehomework.top/2025/03/c2184a5d448d4478273fb3b1d68dcc4c.webp)

解释：

后面我们用group语句通过hangBanHao对他进行分组

SUM()可以解决问题二，他和foreach语句搭配在一起，可以将每组中的公里数加在一起，以实现我们的目标

老规矩，学习网址如下：

[Apache Pig SUM()函数_w3cschool](https://www.w3cschool.cn/apache_pig/apache_pig_sum.html)

dump语句用于展示数据

```bash
store foreach_group_week2 into 'flight2.dat';
```



![image-20211025234407384](https://pic.firehomework.top/2025/03/6044735b6a1e585bbaeddf1f8847da4e.webp)

解释：

store..into...语句是将结果输出至文本文件中，文件名可以依照自己的喜好修改

输出结果如下：

![image-20211025234621118](https://pic.firehomework.top/2025/03/228b7365956b457ba04f579f6512e69f.webp)

后面其实还有很多数据，因为太长就不便全部展示了

至此，大功告成！



###### 总代码一览

```bash
data = load '1987_600.csv' using PigStorage(',') as (Year,Month,DayofMonth,DayOfWeek,DepTime,CRSDepTime,ArrTime,CRSArrTime,UniqueCarrier,FlightNum,TailNum,ActualElapsedTime,CRSElapsedTime,AirTime,ArrDelay,DepDelay,Origin,Dest,Distance,TaxiIn,TaxiOut,Cancelled,CancellationCode,Diverted,CarrierDelay,WeatherDelay,NASDelay,SecurityDelay,LateAircraftDelay);

filter_data = filter data by DayOfWeek>=1;

group_week1 = group filter_data by DayOfWeek;
foreach_group_week1 = foreach group_week1 generate group, COUNT($1);
dump foreach_group_week1

week_concat = foreach filter_data generate CONCAT (UniqueCarrier, FlightNum) as hangBanHao, Distance;
group_week2 = group week_concat by hangBanHao;
foreach_group_week2 = foreach group_week2 generate group, SUM(week_concat.Distance);
dump foreach_group_week2

store foreach_group_week1 into 'flight1.dat';
store foreach_group_week2 into 'flight2.dat';
```



###### 运行脚本

在该脚本所在文件夹中输入：

```bash
pig -x local flight-week-dist.pig
```

运行成功后，刷新一下文件夹，相信你就可以看到你的新文件啦！

如果你找不到你的新文件，你大可以看看运行的日志信息，根据日志来寻找你的新文件。

第一次写这玩意，如果写的不好，请各位看官多多包涵！0.0

