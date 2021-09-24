# HDFS的架构演进之路

**目标**

1. HDFS是如何实现有状态的高可用架构的
2. HDFS是如何从架构上解决单机内存受限问题的
3. 揭秘HDFS能支撑亿级流量的核心源码设计

## 第一章 HDFS架构演变

### Hadoop简介

Hadoop到目前为止发展已经有10余年，版本经过了无数次的更新迭代，目前业内大家把Hadoop的版本分为hadoop1,hadoop2,hadoop3三个版本。

### Hadoop1简介

Hadoop1版本刚出来的时候是为了解决两个问题：一个是海量数据如何存储的问题，一个是海量数据如何计算的问题。Hadoop1的核心设计就是HDFS和MapReduce。HDFS解决了海量数据存储的问题，MapReduce解决了海量数据计算的问题。

HDFS的全称：Hadoop Distributed File System

书籍：《深度剖析Hadoop HDFS》《Hadoop2.x HDFS源码剖析》

### HDFS的架构演进之路

![image-20210725194454934](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725194454934.png)

分布式文件系统

![image-20210725194704517](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725194704517.png)

5台服务器就形成了超级大型的电脑=分布式文件系统

**HDFS1的架构**

HDFS1是一个$\textcolor{red}{主从式} $的架构，主节点只有一个叫$\textcolor{red}{NameNode} $ ，从节点有多个叫$\textcolor{red}{DataNode} $

![image-20210725195941624](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725195941624.png)

**NameNode** 

1) 管理元数据信息(文件目录树)：文件与Block块，Block块与DataNode主机的关系

2) NameNode为了快速响应用户的操作请求，所以把元数据加载到内存里

**DataNode**

1) 存储数据，把上传的数据划分成为固定大小的文件块(Hadoop1，默认64M)

2) 为了保证数据安全，每个文件块默认都有三个副本

![image-20210725205505758](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725205505758.png)

**HDFS1架构的缺陷**

- 单点故障问题

- 内存受限问题

  方法一：能保证NameNode之间的元数据一致

![image-20210725211448398](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725211448398.png)

![image-20210725213024210](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725213024210.png)

**单点故障问题解决方案**

![image-20210725220156262](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725220156262.png)

**内存受限解决方案**

![image-20210725220307969](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210725220307969.png)

**HDFS2架构设计**

HA方案(High Available)

​	解决HDFS1 NameNode单点故障问题

联邦方案

​	解决了HDFS1内存受限问题

**HDFS3架构设计**

HA方案支持多个NameNode

引入纠删码技术



## 第二章 HDFS支持亿级流量的秘密

因为NameNode管理了元数据，用户所有的操作请求都要操作NameNode，大一点的平台一天需要运行几十万上百万的任务。一个任务就会有很多个请求，这些所有的请求都打到NameNode这(更新目录树)，对于NameNode来说这就是亿级流量，NameNode是如何支撑亿级流量的呢？

### HDFS如何管理元数据

![image-20210726085714334](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210726085714334.png)

### 分段加锁和双缓冲方案

![image-20210726090201533](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210726090201533.png)

### 方案实现

hdfs-nx FSEditLog.java

```java
/**
* 使用了面向对象的思想，把一条日志看成一个对象
*/
class EditLog{
  //日志的编号
  long txid;
  //日志的内容
  String content;
  //构造函数
  public EditLog(long txid,String content){
  	this.txid=txid;
  	this.content=content
  }
  //打印日志
}
/**
*双缓冲方案
*/
class DoubleBuffer{
  //内存1

  //内存2

  //把数据写到内存
  write(EditLog log)

  //两个内存交换数据
  setReadyToSync()

  //获取当前正在刷磁盘的内存里的ID最大的值
  getSyncMaxTxid()

  //把数据刷写到磁盘上面
  flush()
}

class FSEditLog{
	//写日志 这个肯定是一个高并发的请求
	//1秒有可能过来几千个请求
	logEdit(String content)

}
```



https://segmentfault.com/a/1190000039179555