# Greenplum数据库

## 1、PostgreSQL

支持绝大部分的主流数据库特性，主要体现在如下几方面:
	（1）函数/存储过程

​	PostgreSQL对非常丰富的过程类语言提供支持，可以编写自定义函数/存储过程。

- ​	内置的plpgsql
- ​	支持的脚本语言有：PL/Lua、PL/LOLCODE、PL/Perl、PL/HP、PL/Python、PL/Ruby、PL/sh、PL/Tcl和PL/Scheme。
- ​	编译语言有C、C++和JAVA。
- ​	统计语言PL/R。

​	（2）索引
​	PostgreSQL支持用户定义的索引访问方法，并且内置了B-tree、哈希和GiST索引。PostgreSQL中的索引有下面几个特点。

- ​	可以从后向前扫描。
- ​	可以创建表达式索引。
- ​	部分索引

（3）触发器

触发器是由SQL查询的动作触发的事件。比如，一个INSERT查询可能激活一个检查输入值是否有效的触发器。大多数触发器都只对INSERT或者UPDATE查询有效。

PostgreSQL完全支持触发器，可以附着在表上，但是不能在视图上。不过视图可以有规则。多个触发器是按照字母顺序触发的。我们还可以用其他过程语言书写触发器函数，不仅仅PL/PgSQL。

（4）并发管理（MVCC）

PostgreSQL的并发管理使用的是一种叫做“MVCC”（多版本并发机制）的机制，这种机制实际上就是现在在众多所谓的编程语言中极其火爆的“LockFree”，其本质是通过类似科幻世界的时空穿梭的原理，给予每个用户一个自己的“时空”，然后通过原子的“时空”控制来控制时间基线，并以此控制并发更改的可见区域，从而实现近乎无锁的并发，而同时还能在很大程度上保证数据库的ACID特性。

（5）规则（RULE）

规则允许我们对由一个查询生成的查询树进行改写。

（6）数据类型

PostgreSQL支持非常广泛的数据类型，包括：

- 任意精度的数值类型；
- 无限长度的文本类型；
- 几何原语；
- IPv4和IPv6类型；
- CIDR块和MAC地址；
- 数组。

用户还可以创建自己的类型，并且可以利用GiST框架把这些类型做成完全可索引的，比如来自PostGIS的地理信息系统（GIS）的数据类型。

（7）用户定义对象

因为PostgreSQL使用一种基于系统表的可扩展的结构设计，所以PostgreSQL内部的几乎所有对象都可以由用户定义，这些对象包括：

- 索引；
- 操作符（内部操作符可以被覆盖）；
- 聚集函数；
- 域；
- 类型转换；
- 编码转换。

（8）继承

PostgreSQL的表是可以相互继承的。一个表可以有父表，父表的结构变化会导致子表的结构变化，而对子表的插入和数据更新等也会反映到父表中。

（9）其他特性与扩展PostgreSQL还支持大量其他的特性，比如：

- 二进制和文本大对象存储；
- 在线备份；
- TOAST（The Oversized-Attribute Storage Technique）用于透明地在独立的地方保存大的数据库属性，当数据超过一定大小的时候，会自动进行压缩以节省空间；
- 正则表达式。

此外PostgreSQL还有大量的附加模块和扩展版本，比如，多种不同的主从/主主复制方案：

- Slony-I；
- pgcluster；
- Mammoth replicator；
- Bucardo



## 2、Greenplum

Greenplum是面向对象的关系型数据库，通过标准的SQL可以对Greenplum中的数据进行访问存取。

本质上讲，Greenplum是一个关系型数据库集群，它实际上是由数个独立的数据库服务组合成的逻辑数据库

Greenplum采用Shared-Nothing架构，整个集群由很多个数据节点（SegmentHost）和控制节点（Master Host）组成，其中每个数据节点上可以运行多个数据库。即Shared-Nothing是一个分布式的架构，每个节点相对独立。

在Greenplum中，需要存储的数据在进入数据库时，将先进行数据分布的处理工作，将一个表中的数据平均分布到每个节点上，并为每个表指定一个分发列（distribute Column），之后便根据Hash来分布数据。

基于Shared-Nothing的原则，Greenplum这样处理可以充分发挥每个节点处I/O的处理能力。在这一过程中，控制节点（Master Host）将不再承担计算任务，而只负责必要的逻辑控制和客户端交互。

Greenplum是面向数据仓库应用的关系型数据库，它是基于目前流行的PosgreSQL开发的，跟PostgreSQL的兼容性非常好，大部分的PostgreSQL客户端工具及PostgreSQL应用都能运行在Greenplum平台上。

## 3、Greenplum特性及应用场景

### 3.1 Greenplum特性

（1）支持海量数据存储和处理

Greenplum使用MPP架构，同时使用多台机器并行计算，极大地提高了对海量数据的处理能力。采取MPP架构的数据库系统才能对海量数据进行管理。

（2）高性价比

Greenplum数据库可以搭建在业界各种开放式硬件平台上，在硬件选型上有很强的自由性。

Greenplum易于维护，可以节省大量的维护成本。

（3）支持Just In Time BI

Greenplum通过准实时、实时的数据加载方式，实现数据仓库的实时更新，进而实现动态数据仓库（ADW）。

基于动态数据仓库，业务用户能对当前业务数据进行BI实时分析（Just In Time BI），能够让企业敏锐感知市场的变化，加快决策支持反应速度。

（4）系统易用性

Greenplum使用通用的PostgreSQL连接包即可与数据库连接，支持绝大部分开发语言。

Greenplum的易用性具体表现如下。

- 支持主流的SQL语法，使用起来十分方便，学习成本低。
- 扩展性好，支持多语言的自定义函数和自定义类型等。
- 提供了大量的维护工具，使用维护起来很方便。
- 在Internet上有着丰富的PostgreSQL资源供用户参考。

（5）支持线性扩展

Greenplum采用MPP并行处理架构。在MPP架构中增加节点就可以线性提高系统的存储容量和处理能力。	

（6）较好的并发支持及高可用性支持

Greenplum是高可用的系统，在已有案例中最多使用了96台机器的集群MPP环境。

除了硬件级的Raid技术外，Greenplum还提供数据库层Mirror机制保护，也就是将每个节点的数据在另外的节点中同步镜像，单个节点的错误不影响整个系统的使用。

对于主节点，Greenplum提供Master/Stand by机制进行主节点容错，当主节点发生错误时，可以切换到Stand by节点继续服务。

（7）支持MapReduce

MapReduce已经被谷歌和雅虎等互联网领先企业证明是一种大规模数据分析技术，Greenplum将这种能力提供给企业。

（8）数据库内部压缩

Greenplum支持对数据库表进行压缩处理，从而提升数据库的性能。

### 3.2 Greenplum应用场景

Greenplum数据引擎是为新一代数据仓库和大规模分析处理而建立的软件解决方案，其最大的特点是不需要高端的硬件支持仍然可以支撑大规模的高性能数据仓库和商业智能查询。在数据仓库、商业智能的应用上，尤其在海量数据的处理方面Greenplum表现出极其优异的性能。

Greenplum主要适用于面向分析的应用，比如构建企业级ODS/EDW、数据集市等。

## 4、Greenplum快速入门

### 4.1 软件安装及数据库初始化

在搭建环境之前，我们必须对Greenplum的架构有一定的了解，并且准备好安装部署的机器，机器硬件、操作系统的安装配置读者可自行完成。

#### 4.1.1 Greenplum架构

![Greenplum架构](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917170010767.png)

​									Greenplum总体架构

每个部件的主要功能

![image-20210917170133826](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917170133826.png)

Master与Segment的关系

![image-20210917170253397](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917170253397.png)

Master和Segment其实都是一个单独的PostgreSQL数据库。每一个都有自己单独的一套元数据字典，在这里，Master节点一般也叫主节点，Segment也叫做数据节点。

Segment节点与Master节点的通信，通过千兆（或万兆）网卡组成的内部连接，在同一台数据节点机器上可以放多个Segment，不同的Segment节点会被赋予不同的端口，同时，Segment之间也不断地进行着交互。为了实现高可用，每个Segment都有对应的备节点（Mirror Segment），分别存在于不同的机器上。

Client一般只能与Master节点进行交互，Client将SQL发给Master，然后Master对SQL进行分析后，再将其分配给所有的Segment进行操作，并且将汇总结果返回给客户端。

### 4.1.2 环境搭建

1. Greenplum集群介绍

在这里的Greenplum集群中，有4台机器，IP分别是：

- 10.20.151.7；  Master节点	两个Primary Segment和两个Mirror Segment
- 10.20.151.8；  Segment节点  两个Primary Segment和两个Mirror Segment
- 10.20.151.9；  Segment节点  两个Primary Segment和两个Mirror Segment
- 10.20.151.10。Segment节点   Master Standy节点  两个Primary Segment和两个Mirror Segment

![image-20210917170645588](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210917170645588.png)

2. 安装Linux

Greenplum没有Windows版本，只能安装在类UNIX的操作系统上。

3. 数据库存储

Greenplum对存储的要求比较高。在文件系统的选择上，在Linux下建议使用XFS。

4. 网络（hosts）

在确定机器配置的时候，要保证所有机器的网络都是通的，并且每台机器的防火墙都是关闭的，避免存在网络不通的问题。

在配置/etc/hosts时，习惯将Master机器叫做mdw，将Segment机器叫做sdw，配置好后，使用ping命令确定所有hostname都是通的。

```shell
#cat /etc/hosts
#BEGIN_GROUP_CUSTOMER
10.20.151.7	 dw-greenplum-1 mdw
10.20.151.8  dw-greenplum-2 sdw1
10.20.151.9  dw-greenplum-3 sdw2
10.20.151.10 dw-greenplum-4 sdw3
#END_GROP_CUSTOMER
```

5. 创建用户及用户组

创建gpadmin用户及用户组，将其作为安装Greenplum的操作系统用户。

将原有用户删除

```shell
#groupdel gpadmin
#userdel gpadmin
```

创建新的用户和用户组：

```shell
#groupadd -g 530 gpadmin
#useradd -g 530-u 530-m -d /home/gpadmin -s /bin/bash gpadmin
```

对文件夹进行赋权，为新用户创建密码：

```shell
```

