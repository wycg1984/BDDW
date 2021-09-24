# 大数据查询利器Hive

## 第一部分 认识大数据

### 1.1 大数据来源（大数据是如何产生的）

典型的数据分析系统，要分析的数据种类比较丰富，依据来源可以分为以下几部分：

![image-20210923164636664](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923164636664.png)

#### 1.1.1 内部数据

![image-20210923165604865](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923165604865.png)

如何进行埋点

- ​	埋点原理

对基于用户行为的数据平台来说，发生在用户界面的，能够获取用户信息的触点就是用户数据的直接来源，而建立这些触点的方式就是埋点。当这些触点获取到用户行为、身份数据后，会通过网络传输到服务器端进行后续的处理

埋点分类

埋点从准确性角度考虑，分为客户端埋点和服务端埋点。客户端埋点，即客户操作界面中，在客户产生动作时对用户行为进行记录，这些行为只会在客户端发生，不会传输到服务器；而服务器端埋点则通常是在程序和数据库交互的界面进行埋点，这时的埋点会更准确地记录数据的改变，同时也会减少由于网络传输等原因而带来的不确定性风险。

![image-20210923170539940](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923170539940.png)

- 埋点采集工作流程

![image-20210923171038204](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923171038204.png)

- 埋点数据采集维度

![image-20210923171359815](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923171359815.png)

埋点文档输出（文档必备要素包含以下方面）

| 要素       | 备注                                                         |
| ---------- | ------------------------------------------------------------ |
| 事件名称   | 埋点的事件名称，如优惠券领取/优惠券使用                      |
| 事件定义   | 用户点击领取优惠券，则上报该事件                             |
| 包含属性   | 用户进行了该行为，上报事件中需要传输哪些内容，如用户ID、时间、应用版本、网络信息、手机型号、IP地址、内容ID等；如某些属性在所有事件中都需要上传，则可以整理公共属性进行管理 |
| 属性定义   | 说明属性的定义，如用户主动上传的地址；如没有可以用用户IP代替 |
| 属性值类型 | 说明传输属性的类型，字符串、数值、bool                       |
| 开发名称   | 对应的开发变量名 可以由开发进行补充 如userId                 |
| 当前状态   | 明确当前该变量的状态，如待开发、开发中、验收中、已上线、已下线 |
| 上线版本   | 说明该内容在哪个版本进行上线 如2.3.1                         |
| 备注       | 备注中可记录该属性的变动情况和常见值等                       |

- 案例

在整个埋点工作流程中，埋点设计环节是最关键的一环，在这个环节整体可以根据需求梳理和转化的路径，分为业务分解、分析指标、事件设计、属性设计的四个阶段。其中，业务分解和分析指标，是事件设计和属性设计的关键信息输入，通过对业务场景和分析需求的梳理，确认埋点的内容和范围，而事件和属性设计，则是将这些业务分析诉求，转换成面向开发的需求语言，让开发能看懂需要他在什么场景和规则下采集哪些数据。结下来，我们就以优惠券营销场景为例，讲述该如何进行埋点设计。

**案例：优惠券营销场景的事件设计**

![image-20210923172617231](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923172617231.png)

埋点采集中常见事件属性的类型举例，主要包括用户属性、事件属性、对象属性和环境属性四大类。

![image-20210923173813971](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923173813971.png)

![image-20210923173829645](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923173829645.png)

#### 1.1.2外部数据

- 竞争对手数据---爬取数据
- 国家统计局数据
- 友商提供

#### 1.1.3 大数据特点

![image-20210923174146333](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923174146333.png)

### 1.2 数据仓库（数据是如何存储的）

数据仓库规模大、周期长，一般规模比较小的企业用户难以承担。因此，作为快速解决企业当前存在的实际问题的一种有效方法，独立型数据集市成为一种即成事实。

数据集市（Data Mart），也叫数据市场，数据市场就是满足特定的部门或者用户的需求，按照多维的方式进行存储，包括定义维度、需要计算的指标、维度的层次等，生成面向决策分析需求的数据立方体。

数据集市主要是针对一组特定的某个主题域、部门或者特殊用户需求的数据集合。这些数据需要针对用户的快速访问和报表展示进行优化，优化的方式包括对数据进行轻量级汇总，在数据结构的基础上创建索引。

数据集市的目标分析过程包括对数据集市的需求进行拆分，按照不同的业务规则进行组织，将与业务主题相关的实体组织成主题域，并且对各类指标进行维度分析，从而形成数据集市目标说明书。内容包括详细的业务主题、业务主题域和各项指标及其分析维度。

1.2.1 什么是数据仓库

数据仓库（Data Warehouse）简称DW。数据仓库顾名思义，是一个很大的数据存储集合，出于企业的分析性报告和决策支持目的而创建。对多样的业务数据进行筛选与整合。它为企业提供一定的BI能力，指导业务流程改进。

![image-20210923175032425](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923175032425.png)

#### 1.2.2 数据仓库解决什么问题

数据仓库从大的方向来说解决了三类问题：存储、快速提取、跨部门应用。

![image-20210923203037864](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923203037864.png)

#### 1.2.3 数据仓库的主要特征

- 面向主题的
- 集成的

![image-20210923203314813](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210923203314813.png)

- 稳定的（不易失的）
- 时变的（反应历史变化的）

#### 1.2.4 数据仓库与数据库的区别

1、数据库与数据仓库的区别实际讲的是OLTP与OLAP的区别

2、数据仓库的出现，并不是要取代数据库

| 差异项   | 数据库               | 数据仓库                |
| -------- | -------------------- | ----------------------- |
| 特征     | 操作处理             | 信息处理                |
| 面向     | 事务                 | 分析                    |
| 用户     | DBA、开发            | 经理、主管、分析人员    |
| 功能     | 日常操作             | 长期信息需求、决策支持  |
| DB设计   | 基于ER模型，面向应用 | 星形/雪花模型、面向主题 |
| 数据     | 当前的、最新的       | 历史的、跨时间维度      |
| 汇总     | 原始的、高度详细     | 汇总的、统一的          |
| 视图     | 详细、一般关系       | 汇总的、多维的          |
| 工作单元 | 短的、简单事务       | 复杂查询                |
| 访问     | 读/写                | 大多为读                |
| 关注     | 数据进入             | 信息输出                |
| 操作     | 主键索引操作         | 大量的磁盘扫描          |
| 用户数   | 数百到数亿           | 数百                    |
| DB规模   | GB到TB               | >=TB                    |
| 优先     | 高性能、高可用性     | 高灵活性                |
| 度量     | 事务吞吐量           | 查询吞吐量、响应时间    |



#### 1.2.5 数据仓库架构

![image-20210924083409988](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924083409988.png)

大数据系统需要数据模型方法来帮助更好地组织和存储数据，以便在性能、成本、效率和质量之间取得最佳平衡，主流的方法是分层架构。

![image-20210924083545838](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924083545838.png)

数据仓库的数据来源于不同的源数据，并根据多样的数据应用，数据自下层流入数据仓库后向上层开发应用，而数据仓库只是中间集成化数据管理的一个平台。

**知名企业的数仓架构**

![image-20210924083759128](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924083759128.png)

![image-20210924083824954](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924083824954.png)



#### 1.2.6 数据仓库元数据管理

源数据（Meta Data）主要记录数据仓库中模型的定义，各层级间的映射关系，监控数据仓库的数据状态及ETL的任务运行状态，一般会通过元数据质量库（Meta Data Repository）来统一地存储和管理元数据，其主要目的是使数据仓库的设计、部署、操作和管理能达成协同和一致，保证数据质量。

元数据是数据仓库挂历系统的重要组成部分，元数据管理是企业级数据仓库中的关键组件，贯穿数据仓库构建的整个过程，直接影响着数据仓库的构建、使用和维护。

构建数据仓库的主要步骤之一是ETL；这时元数据将要发挥重要的作用，它定义了源数据系统到数据仓库的映射、数据装换的规则、数据仓库的逻辑结构、数据更新规则、数据导入历史记录以及装载周期等相关内容。数据抽取和转换的专家以及数据仓库管理员通过元数据高效地构建数据仓库。

用户在使用数据仓库时，通过元数据访问数据，明确数据项的含义以及定制报表数据仓库的规模及其复杂性离不开正确的元数据管理，包括增加或移除外部数据源，改变数据清洗方法，控制出错的查询以及安排备份等。

![image-20210924090011721](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924090011721.png)

元数据分为技术元数据和业务元数据

1、技术元数据为开发和管理数据仓库的IT人员使用，描述了与数据仓库开发、管理和维护相关的数据，包括数据源信息、数据转换描述、数据仓库模型、数据清洗与更新规则、数据映射和访问权限等。

2、业务元数据为管理层和业务分析人员服务，从业务角度描述数据包括商务术语、数据仓库中有什么数据、数据的位置和数据的可用性等。

元数据不仅定义了数据仓库中数据的模式、来源、抽取和转换规则等，而且是整个数据仓库系统运行的基础，它把数据仓库系统中各个松散的组件联系起来，组成了一个有机的整体。

#### 1.2.7 数据治理

数据是企业的资产，数据治理能成就（特别是银行）的未来。它涉及数据质量、数据管理、数据政策、商业过程管理、风险管理等多个领域。

- **脏数据的种类**

![image-20210924090801566](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924090801566.png)

- **数据治理原则**

![image-20210924091039737](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924091039737.png)



### 1.3 Hadoop概述

#### 1.3.1 Hadoop简介

Hadoop是什么？简单来说，Hadoop就是解决大数据时代下海量数据的存储和分布式计算问题

Hadoop不是指具体的一个框架或者组件，它是Apache软件基金会下用java语言开发的一个开源分布式计算平台，实现在大量计算机组成的集群中对海量数据进行分布式计算，适合大数据的分布式存储和计算，从而弥补了传统数据库在海量数据下的不足。

#### 1.3.2 Hadoop优点

- 高可靠性：Hadoop按位存储和处理数据的能力值得人们信赖
- 高扩展性：Hadoop是在可用的计算机集群间分配数据并完成计算任务，这些集群可用方便地扩展到数以千计的节点中
- 高效性：Hadoop能够在节点之间动态地移动数据，并保持各个节点的动态平衡，因此处理速度非常快
- 高容错性：Hadoop能够自动保存数据的多个副本，并且能够自动将失败的任务重新分配
- 低成本：Hadoop是开源的，项目的软件成本因而得以大大降低



#### 1.3.3 Hadoop生态圈

![image-20210924092154653](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924092154653.png)

![image-20210924092228326](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924092228326.png)

#### 1.3.4 分布式存储（HDFS）

HDFS就是将文件切分成固定大小的数据块block（文件严格按照字节来切分，所以若是最后切的剩一点点，也算单独一块，Hadoop2.x默认的固定大小是128MB，不同版本默认值不同，可以通过Client端上传文件设置）

![image-20210924092630968](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924092630968.png)

HDFS的优点：

- ​	分布式存储
- ​	支持分布式和并行计算
- ​	水平可伸缩性



#### 1.3.5 分布式计算（MapReduce）

MapReduce为海量的数据提供了计算

MapReduce从它名字上来看就大致可以看出个缘由，两个动词Map和Reduce，Map就是将一个任务分解成多个任务，Reduce就是将分解后多任务处理的结果汇总起来，得出最后的分析结果，MapReduce采用分而治之的思想，简单地说，MapReduce就是任务的分解和结果的汇总。



## 第二部分 Apache Hive

### 2.1 Hive简介

#### 2.1.1 什么是Hive

Hive是Facebook为了解决海量日志数据的统计分析而开发的基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供类SQL查询功能。

本质上：将HQL语句转换为MapReduce任务进行运行（转化流程如下）

![image-20210924094343679](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924094343679.png)

主要用途：做离线数据分析，比直接使用MapReduce开发效率高



#### 2.1.2 为什么使用Hive

直接使用Hadoop MapReduce处理数据面临的问题：

- ​	人员学习成本太高
- ​	MapReduce实现复杂查询逻辑开发难度太大

使用Hive：

- ​	操作接口采用类SQL语法，提供快速开发的能力
- ​	避免了去写MapReduce减少开发人员的学习成本
- ​	功能扩展很方便



#### 2.1.3 Hive的优缺点

1）优点

1.操作接口采用类SQL语法，避免了写MapReduce程序，简单易上手，减少开发人员学习成本

2.在数据处理方面，Hive语句最终会生成MapReduce任务去计算，常用于离线数据分析，对数据实时性要求不高的场景

3.在数据存储方面，它能够存储很大的数据集，并且对数据完整性、格式要求并不严格

4.在延展性方面，Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数

2）缺点

1.Hive的HQL本身表达能力有限，不能够进行迭代式计算，在数据挖掘方面也不擅长

2.Hive操作默认基于MapReduce引擎，延迟高，不适用于交互式查询，因此智能化程度低，并且基于SQL调优困难，粒度较粗



#### 2.1.4 Hive架构

![image-20210924095812916](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924095812916.png)

1、用户接口：Client CLI(Hive shell)、JDBC/ODBC(JAVA访问hive)、WEBUI(浏览器访问hive)

2、元数据：Metastore元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列分区字段、表的类型（是否是外部表）、表的数据所在的目录等

3、Hadoop使用HDFS进行存储，使用MapReduce进行计算

4、驱动器（Driver）

1）解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr;对于AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误

2）编译器（Physical Plan）：将AST编译生成逻辑执行计划

3）优化器（Query Optimizer）：对逻辑执行计划进行优化

4）执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于hive来说就是MR/Spark

Hive运行机制

![image-20210924100545637](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924100545637.png)

Hive通过给用户提供的一系列交互接口，接收到用户的指令（SQL），使用自己的Driver，结合元数据，将这些指令翻译成MR，提交到Hadoop中执行，将执行返回的结果输出到用户交互接口上。



#### 2.1.5 Hive与传统数据模型

Hive具有SQL数据库的外表，但应用场景完全不同，Hive只适用于批量数据统计分析

![image-20210924101427464](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924101427464.png)



#### 2.1.6 Hive数据模型

Hive的数据模型主要有以下四种

![image-20210924101625521](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924101625521.png)



#### 2.1.7 Hive安装

![image-20210924101805513](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210924101805513.png)

#### 2.1.8 远程连接Hive



### 2.2 Hive DDL数据定义语言

**Hive语句注意事项**

- ​	SQL语言大小写不敏感
- ​	SQL可以写在一行或多行
- ​	关键字不能被缩写也不能分行
- ​	各子句一般要分行写
- ​	使用缩进提高语句的可读性
- ​	--为注释符号



#### 2.2.1 创建和删除数据库

1）创建数据库

```SQL
#CREATE DATABASE [IF NOT EXISTS] 数据库名;
create database lagou;
```



2）查看数据库

```sql
SHOW DATABASES;
```



3）删除数据库

- 删除空数据库

```SQL
-- drop database 数据库名;
drop database lagou;
-- 查看所有数据库
show databases;
```

- 如果删除的数据库不存在 最好采用if exists判断数据库是否存在

```sql
-- drop database if exists 数据库名;
drop database if exists lagou;
```

- 如果数据库不为空，可以采用cascade命令 强制删除

```sql
-- drop database 数据库名 cascade;
drop database lagou cascade;
```

4）使用进入数据库

```sql
-- use 数据库名;
use lagou;
```

#### 2.2.2 创建表--数据类型

1）Hive数据类型

数字类型

| 类型     | 长度  | 备注               |
| -------- | ----- | ------------------ |
| TINYINT  | 1字节 | 有符号整型         |
| SMALLINT | 2字节 | 有符号整型         |
| INT      | 4字节 | 有符号整型         |
| BIGINT   | 8字节 | 有符号整型         |
| FLOAT    | 4字节 | 有符号单精度浮点型 |
| DOUBLE   | 8字节 | 有符号双精度浮点型 |
| DECIMAL  | --    | 可带小数的精确数字 |

日期时间类型

| 类型      | 长度 | 备注                                |
| --------- | ---- | ----------------------------------- |
| TIMESTAMP | --   | 时间戳 内容格式 yyyy-mm-dd hh:mm:ss |
| DATE      | --   | 日期 内容格式YYYYMMDD               |
| INTERVAL  | --   | --                                  |

字符串类型

| 类型    | 长度              | 备注           |
| ------- | ----------------- | -------------- |
| STRING  | --                | 字符串         |
| VARCHAR | 字符数范围1~65535 | 长度不定字符串 |
| CHAR    | 最大字符数255     | 长度固定字符串 |

Misc类型

| 类型    | 长度 | 备注                |
| ------- | ---- | ------------------- |
| BOOLEAN | --   | 布尔类型 TRUE/FALSE |
| BINARY  | --   | 字节序列            |

复合类型

| 类型      | 长度 | 备注                             |
| --------- | ---- | -------------------------------- |
| ARRAY     | --   | 包含同类型元素的数组 索引从0开始 |
| MAP       | --   | 字典 MAP<>                       |
| STRUCT    | --   | 结构体                           |
| UNIONTYPE | --   | 联合体                           |

2）Hive建表

分为内部表和外部表

|              | 内部表                   | 外部表                                                       |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| 数据管理     | Hive自身管理             | HDFS管理                                                     |
| 数据存储位置 | /user/hive/warehouse     | LOCATION指定或hive将在HDFS上的/user/hive/warehouse下外部表的表名文件夹 |
| 删除表       | 直接删除元数据及存储数据 | 仅仅会删除元数据                                             |
| 修改表       | 将修改直接同步给元数据   | 需要修复（MSCK REPAIR TABLE table_name）                     |

1.直接建表法

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
	[(col_name data_type [COMMENT col_comment]...)]
	[COMMENT table_comment]
	[PARTITIONED BY (col_name,data_type[COMMENT col_comment])]
	[CLUSTERED BY (col_name,col_name,...)]
	[SORTED BY (col_name [ASC|DESC],...)] INTO num_buckets BUCKETS]
	[ROW FORMAT row_format]
	[STORED AS file_format]
	[LOCATION hdfs_path]
```

#### 2.2.3 创建表---分隔符



## 第三部分  案例实战



## 第四部分 总结