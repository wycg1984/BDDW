# **StreamSets学习笔记**

# StreamSet操作教学-SqlServer同步到Hive

## 1、业务背景

公司上大数据，要把sqlserver里的业务数据实时同步到大数据平台上。几天调研后选择StreamSet作为ETL工具。技术选型的理由主要有几点：

sqlserver的坑太深，网上找了很多工具对sqlserver的支持力度都不是很大(微软全家桶的要哭了~)
自己开发ETL程序耗时太长，同时配套的ETL metrics工具也需要配备，劳民伤财。感觉有时间开发不如把精力放在业务数据研究和指标计算上。
sqlserver支持两种实时同步机制：CDC和Change Tracking，CDC使用起来比较重 果断放弃。

## streamset安装

我用的是CDH6.2版本集成化了StreamSet，方便管理。

1、[streamset下载地址](https://archives.streamsets.com/index.html)

![img](https://img-blog.csdnimg.cn/2020052815540358.png)

2、配置本地yum源
将下载的三剑客文件放在一个新建文件夹中，并移动到/var/www/html 目录中，做离线包的下载地址，用浏览器访问如下，表示成功

![img](https://img-blog.csdnimg.cn/20200528155708226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

3、配置CSD
将STREAMSETS-3.1.4.0.jar拷贝到/opt/cloudera/csd,并更改权限，然后重启cloudera-scm-server服务，没有csd文件夹就自己新建一个。
4.下载分发激活Parcel包
传送门：https://cloud.tencent.com/developer/article/1078852

## sqlserver之Change Tracking

sqlserver上开启Change Tracking，这里要做到表级别Change Tracking

开启库级别Change Tracking

```sql
ALTER DATABASE 数据库名 SET CHANGE_TRACKING = ON (CHANGE_RETENTION = 2 DAYS,AUTO_CLEANUP = ON)
```


开启表级别Change Tracking

```sql
ALTER TABLE [dbo].[Department] ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON)
```



## Streamset之sqlserver配置

1、右上角点击All Stages找到SQLxxx

![img](https://img-blog.csdnimg.cn/2020052816162016.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

2、配置分析

![img](https://img-blog.csdnimg.cn/20200528161900693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2020052816200747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2020052816211722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200528162202301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200528162251703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

## StreamSet之Hive配置

配置Hive要使用HiveMeData和HiveMeStore两个插件
重要参数详解：

Database Expression：Hive数据库选择，不写则是默认库
Partition Configuration：设置Hive分区的参数，这里可以设置成多级分区，只要选择“+”就行，其中Partition Value Expression是获取分区号数据的表达式，可以自己生成也可以根据表字段里的数据定义分区。因为我们数据原因，我这里表达式写的很蛋疼~有关表达式的写法建议大家直接去撸下官网(PS:Streamset的官网文档还是很不错的，个人感觉比flink强多了！！)。
官网functions传送门：https://streamsets.com/documentation/datacollector/latest/help/datacollector/UserGuide/Expression_Language/Functions.html#concept_lhz_pyp_1r

## StreamSet之HDFS配置

重要参数详解：
File System URI:hdfs的URI连接
Configuration Files Directory：hdfs的config文件位置，需要加载core-site.xml
Files Prefix:生成hdfs的文件前缀，建议大家改一下，不然生成的文件名巨长~
Files Suffix：生成hdfs的文件后缀，为用了txt格式，记住不用加点
Directory in Header：打钩后会直接落盘到hive的/user/hive/warehouse/中，如果不打钩，则可以自定义落盘路径
Compression Codec ：选择压缩算法
Use Roll Attribute ：回滚机制

![img](https://img-blog.csdnimg.cn/20200528163932355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200528164043848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

## StreamSet之踩坑实战

处理数据同步过程中避免不了加减字段，清洗数据等问题，业务越复杂的OLTP架构中数据迁移越是麻烦，在学习使用StreamSet中，建议大家先去撸清楚官网。我自己在学习使用中总结两个方面感觉很有必要：

函数表达式，这里的坑很多，要想使用数据，调用数据都要通过表达式来完成
Field开头的插件大多是对数据做处理加工的，大家使用前一定要对Field开头的插件挨个看一遍。我自己就使用了Field Type Converter(字段值类型转换)、Expression Evaluator(表达式执行器)

# StreamSet操作教学: JVM性能优化,让你秒支持上百个Job

## 一、JVM性能优化

1.进入启动脚本
代码如下（示例）：

```shell
vim /opt/streamsets/streamsets-datacollector-3.14.0/libexec/sdc-env.sh
```

2.修改JVM参数
优化1:

```shell
export SDC_JAVA_OPTS="-Xmx20480m -Xms20480m -Xmn4096m -server -XX:-OmitStackTraceInFastThrow ${SDC_JAVA_OPTS}"
```

 优化2：

```shell
export SDC_JAVA8_OPTS=${SDC_JAVA8_OPTS:-"-XX:+UseG1GC -XX:ConcGCThreads=4 -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=10m -Djdk.nio.maxCachedBufferSize=262144"}
```

 选择G1垃圾回收机制可以减少web页面卡顿问题,同时可以根据服务器配置设置G1每个Region大小等配置.


# StreamSets操作教学: 数据库同步HBase,支持rowkey反转

## 前言

在实时增量同步方案时 博主是通过Mysql -> Kafka -> HBase方案，其中加入Kafka缓存功能目的是在导入增量数据前 需要把历史全量数据先load进HBase，这里为了能保证数据一致性，才使用Kafka功能。

 1. 配置Kafka Consumer

    ![img](https://img-blog.csdnimg.cn/2021063021483148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)


这里填写Kafka的必要配置，可以设置Kafka每个Batch pull的数据量。![img](https://img-blog.csdnimg.cn/20210630214919313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)



选择Kafka消费数据格式为Json。

2. 配置JavaScript Evaluator

   ![img](https://img-blog.csdnimg.cn/20210630215026653.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)


编写JavaScript实现rowkey反转功能，并赋值给一个新的字段rowkey,因为测试后发现Streamsets不支持原生的reserve反转函数,自己就通过遍历字符串实现了一个反转功能。大家有更好的方法可以留言交流~

3. 配置HBase

   ![img](https://img-blog.csdnimg.cn/20210630215209534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

这里的rowkey就指定为前面JavaScript中生成的新字段就行了。

# 利用StreamSets的Kafka组件写入HBase后，读取HBase数据时Decimal类型数据异常

今天在实时读取HBase维度数据时发现取数据时有问题，Deicmal类型的数据异常，经过排查推理发现最后是Kafka数据通过StreamSets写入时存在问题：

问题原因：

Kafka里存的是Json数据，StreamSets通过解析JSON数据后把每个字段值写入HBase，因为JSON里字段是没有具体数据类型的，StreamSets可能会把字段转成字符串后解析成二进制写入HBase，这里就造成存储HBase的Decimal类型数据精度存在问题。

解决办法：

新增一个Field Type Converter组件模块，把JSON中为DECIMAL类型的字段由String类型转成DECIMAL类型，并标注精度和Round方式：


![img](https://img-blog.csdnimg.cn/20210817155755666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JsYWNrQXJtYW5k,size_16,color_FFFFFF,t_70)

# StreamSets集成Kerberos详解(Mysql实时同步数据到HBASE)

StreamSets集成Kerberos步骤如下：
StreamSets的安装部署省略
1、进入到安装kerberos的服务器生成指定keytab文件

```shell
#1、进入到Kerberos管理界面
kadmin.local
#2、在管理界面指定指定用户的Keytab文件生成
ktadd -k /opt/hbase.keytab -norandkey hbase@AUTO.COM
```


2、将生成的Keytab文件拷贝到安装StreamSets服务器节点

```shell
#将生成的keytab文件S拷贝到StreamSets的安装目录下
scp /opt/hbase.keytab root@streamset:/opt/streamsets/
```

3、配置StreamSets的sdc.properties
vi ./streamsets/etc/sdc.properties

```shell
kerberos.client.enabled=true
kerberos.client.principal=hbase@AUTO.COM
kerberos.client.keytab=/opt/streamsets/hbase.keytab
```

# StreamSets 从传统关系型数据库Mysql到Hbase的实时数据采集

最近在研究StreamSets，因为它官网的标题就是处理复杂数据流，就想试一下，做了几个简单Demo之后，发现从传统关系型数据库到Hbase貌似经过很简单的操作就可以做到实时的数据采集：

1、首先不管是什么环境，一定要先装好StreamSets,我用的的是CDH，直接在StreamSets的官网下载对应版本的parcel包，在CDH离线安装就可以了

![img](https://img-blog.csdn.net/20171114092207616?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjU1MTUyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2、直接进入StreamSets的web端，登陆，下图是一个简单测试实例，从Mysql到Hbase抽数据的例子，注意左上角的时间，这个任务已经跑了17个小时，这是他一直在做抽数据的操作，也就是这时候我在mysql相应表插入一条数据后，会被实时抽到hbase,并且注意下面的报表图，他会对你的抽入抽出有很详细的报表展现，包括Error的监控

![img](https://img-blog.csdn.net/20171114092714560?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjU1MTUyNA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 

3、接下来肯定就是要关心数据的抽取时的配置了，配置很简单，Mysql端配置好JDBC 连接，按格式写好sql就可以了，Hbase端配置好Zooker队列，配置好字段映射（注意格式就可以了）

# StreamSets 实时同步mysql数据到kudu

StreamSets 实时同步mysql数据到kudu
初始化数据
业务库下的表 wm.admin_user_app wm.department

1.在hive创建数据库;

```sql
CREATE DATABASE zxl_db;
CREATE DATABASE zxl_db_tmp;

```

2.在hive建kudu表,hive临时表;

```sql
# impala-shell

CREATE TABLE IF NOT EXISTS zxl_db_tmp.admin_user_app (
  `id` bigint ,
  `user_id` bigint ,
  `app_id` bigint ,
  `o_id` bigint ,
  `c_id` bigint ,
  `status` tinyint ,
  `update_time` string ,
  `create_time` string ) 
  row format delimited fields terminated by '\t'
  STORED AS TEXTFILE

# impala-shell

CREATE TABLE IF NOT EXISTS  zxl_db.admin_user_app (
  `id` bigint ,
  `user_id` bigint ,
  `app_id` bigint ,
  `o_id` bigint ,
  `c_id` bigint ,
  `status` tinyint ,
  `update_time` string ,
  `create_time` string ,
  PRIMARY KEY (`id`))
	STORED AS KUDU;
	

	CREATE TABLE zxl_db_tmp.department (

  `dept_id` bigint,
  `unit_code` string,
  `parent_id` bigint,
  `name` string,
  `status` tinyint,
  `sort` bigint,
  `ext` string,
  `update_time` string,
  `create_time` string )
  row format delimited fields terminated by '\t'
  STORED AS TEXTFILE;

  CREATE TABLE zxl_db.department (
  `dept_id` bigint,
  `unit_code` string,
  `parent_id` bigint,
  `name` string,
  `status` tinyint,
  `sort` bigint,
  `ext` string,
  `update_time` string,
  `create_time` string,
  PRIMARY KEY (`dept_id`))
  STORED AS KUDU;
```

Note:

 (1) 建kudu表时必须指明PRIMARY KEY

 (2)建kudu表时必须指明为kudu存储类型

 (3)建hive表最好指定分隔符 row format delimited fields terminated by '\t'，本次测试从mysql抽数到hive，默认的分隔符是逗号或空格，抽数时指定 \t 分隔符导致数据都为null。

3.从mysql抽数到hive临时表

```shell
# 抽数

sudo -u hive sqoop  import  \
--connect jdbc:mysql://10.234.7.73:3306/wm?tinyInt1isBit=false \
--username work \
--password phkAmwrF \
--hive-database zxl_db_tmp \
--hive-table admin_user_app \
--query "select id,user_id,app_id,o_id,c_id,status,date_format(update_time, '%Y-%m-%d %H:%i:%s') update_time,date_format(create_time, '%Y-%m-%d %H:%i:%s') create_time from admin_user_app where 1=1 and \$CONDITIONS" \
--hive-import \
--null-string '\\N' \
--null-non-string '\\N' \
--fields-terminated-by "\t" \
--lines-terminated-by "\n"  \
--delete-target-dir \
--target-dir /user/hive/import/admin_user_app \
--hive-drop-import-delims \
--hive-overwrite  \
-m 1;


sudo -u hive sqoop  import  \
--connect jdbc:mysql://10.234.7.73:3306/wm?tinyInt1isBit=false \
--username work \
--password phkAmwrF \
--hive-database zxl_db_tmp \
--hive-table department \
--query "select dept_id,unit_code,parent_id,name,status,sort,ext,date_format(update_time, '%Y-%m-%d %H:%i:%s') update_time,date_format(create_time, '%Y-%m-%d %H:%i:%s') create_time from department where 1=1 and \$CONDITIONS" \
--hive-import \
--null-string '\\N' \
--null-non-string '\\N' \
--fields-terminated-by "\t" \
--lines-terminated-by "\n"  \
--delete-target-dir \
--target-dir /user/hive/import/department \
--hive-drop-import-delims \
--hive-overwrite  \
-m 1;


# 修复分区(若有分区则需要修复) # beeline

msck repair table zxl_db_tmp.admin_user_app
```



Note:

 (a) time,date,datetime ,timestamp（非string类型）导入到hive时时间格式会有问题，如：“2018-07-17 10:01:54.0”；需要在导入时进行处理;

(b) tinyInt1isBit=false 是为了解决sqoop从mysql导入数据到hive时tinyint(1)格式自动变成Boolean;

© mysql到hive字段类型会发生改变，本例中mysql的int映射到hive变成了bigint，若要指定映射类型，需要在hive手动创建表指定数据类型;

4.从hive临时表抽数到kudu

```sql
# impala-shell

upsert into table zxl_db.admin_user_app select id,user_id,app_id,o_id,c_id,status,update_time,create_time from zxl_db_tmp.admin_user_app

upsert into table zxl_db.department select dept_id,unit_code,parent_id,name,status,sort,ext,,update_time,create_time from zxl_db_tmp.department


# 修复元数据  # impala-shell

invalidate metadata zxl_db.admin_user_app
invalidate metadata zxl_db.department

# 删除hive临时表  # beeline

drop table zxl_db_tmp.admin_user_app
```



SDC实时同步数据
1.创建管道

![img](https://img-blog.csdnimg.cn/20190705165100835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

2.添加和配置 binlog采集组件

![img](https://img-blog.csdnimg.cn/20190705165127991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190705165147782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2019070516521999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)


Initial offset 在数据库中使用 SHOW MASTER STATUS;获取；

Include Tables配置需要实时同步的表，多个使用逗号隔开；

3.添加和配置流选择器，可用于过滤数据库

![img](https://img-blog.csdnimg.cn/20190705165254101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

4.数据处理

![img](https://img-blog.csdnimg.cn/20190705165342221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

```python
for record in records:
  newRecord = sdcFunctions.createRecord(record.sourceId + ':newRecordId')
  try:
    if record.value['Type'] == 'DELETE':
      newRecord.attributes['sdc.operation.type']='2'
      newRecord.value = record.value['OldData']
    else:
      newRecord.attributes['sdc.operation.type']='4';
      newRecord.value = record.value['Data'];
    # Write record to processor output
    record.value['Type'] = record.value['Type']
    newRecord.value['Table'] = record.value['Table']
    

    output.write(newRecord)

  except Exception as e:

    # Send record to error

	error.write(newRecord, str(e))
```

5.写入kudu

![img](https://img-blog.csdnimg.cn/20190705165446674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190705165509861.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

Table Name：impala::zxl_db.${record:value(’/Table’)} 表示将表数据写入zxl_db的表中

6.启动管道

![img](https://img-blog.csdnimg.cn/2019070516553897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

7.验证是否实时同步

![img](https://img-blog.csdnimg.cn/20190705165556720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoeWxsaW4=,size_16,color_FFFFFF,t_70)

Shylin
