# sparkSQL简单整理

spark sql 
第一天:

出现原因:
	--hive工作:将SQL语句转化为MR程序(新版本已经不支持MR转为支持spark),降低编程难度
	--spark SQL转化为RDD,提交到集群执行,执行效率较高
特点:
	--易整合:无缝使用SQL
	--统一的数据访问方式:数据源(orc/json/hive/jdbc/avro)-->统一的方式访问
	--兼容hive:hive语句(MetaStore/HiveSQL/UDFs/SerSes)无需修改可直接给spark SQL使用
	--标准的数据连接(BI Tools)
	结构化数据(有一定范式的数据)处理的spark模块,sparkSQL会根据SQL逻辑自动进行优化转化为RDD,
编程风格:	
	--SQL:写SQL语句
	--DSL:调用底层API接口--DataFrame和DataSet--两种数据抽象
	两者结果是一样的,可以在SQL和DSL之间随意切换,底层实现是一样的并不依赖与API
定义:学会总结
	--处理的数据是结构化数据
	--sparkSQL会根据结构化信息(scheme)进行优化
	--可以使用SQL/提供的api(sparkcore数据抽象是RDD;sparkSQL数据抽象是dataframe和dataset)进行编程逻辑处理
	--底层使用统一的方式对所表示的逻辑进行分析优化,生成最后的执行代码(spark core代码)

spark sql程序入口:SparkSession(2.0之后)对象(类似于spark core中的sparkcontext) 
	
Dataset:
	--分布式数据集(强类型的(数据类型是任意的),可以使用兰布达),
	--可以从JVM对象/transformation类型的方法(类似于transfomation算子从RDD得到RDD)
	--Dataset的api有scala和java两种版本,暂不支持python版本
DataFrame:
	--是dataset的一个子集
	--数据类型只有一种:row类型
	--类似于关系型数据库中的一张表(表头,列,属性,类型,约束...)/python里面的dataframe
	--构建:从结构化数据文件/Hive表/外部数据库/RDD中生成
	--api有scala,java,python

RDD转换为DF --注意导入隐式类import sparkSession.implicit._
从RDD出发生成DF并处理的方式:
	--RDD对数据预处理(处理为tuple类型)--.toDF--DataFrame(表结构)-->SQL语句
反射方式:从对象反推类的定义中的属性--RDD变成对象的数据
	先得到RDD--变成对象的数据集--toDF(列名从样例类中取得,此处无需参数)--注册一张表,基于表去操作df中的数据--sparkSession.sql调用SQL语句操作
DataFrame:等价于RDD+结构化信息(scheme)
	RDD-->变成Row类型的数据(对象)--创建与RDD对应的scheme信息(StructType里面定义一个List(List(StructField(属性字段,类型,字段约束)))
		--将RDD和Scheme进行关联生成DF
使用SQL语句处理DF要先注册一张表生成一个视图,然后基于表写SQL语句
使用DSL则不需要注册表,直接使用DF的api(方法名和SQL语句较为相似)即可,取出表头所对应列的信息(即内容是变化的)需要再前面加$,很多功能都需要隐式转换


第二天:sparkday07--test0603
DataSet:惰性的,由action方法触发
	API操作分两类:transformation类型的方法和action类型的方法
		--transformation:输入为DS输出为DS
		--action:输入为DS,触发计算,返回结果	
	代表的是一个逻辑计划,当被触发时spark查询优化器将对其优化并产生为物理计划:处理过程可用explain方法查看	
	编码过程encoder 可用scheme方法查看过程	
生成方式:
	--由DF转化为DS:sparkSession.read.parquet().as... (Json/JDBC/CSV/requet) 外部文件系统的文件
	--通过DS的transformation的方法由DS生成新的DS
RDD与DF与DS之间的互相转化:
	--RDD:基于JVM对象的
	--DS:类型安全+快速
读取文件:底层默认parquet文件--按列存储(可按列取),压缩比 较大,效率比较高
Json/CSV/parquet/JDBC :读写都有两种方式
	--mode写模式 看官网 Save Mode "error"  "append" "overwrite" "ignore"
	
DS/DF下的api:
	
	
第三天:英文版资料 sparkday08--test0604
内置函数整理
网址: Databricks
Sparksql执行流程:
	--sql--Parse--analyzer(元数据)-->optimizer-->SparkPlan--RDDs
	--sql-->parsePlan-->ParseDriver.scala
		 -->ofRow--executePlan--?
	
Sparksql join :有可能发生shuffle
	--Broadcast join / shuffle hash join / sort merge join
	--区别:
		--一个表比较小(几百M几个G)以广播比变量的形式发送到每个Executor上,避免shuffle--只能广播右表--可在配置文件修改相应参数
		--join之前先做重分区(使用相同分区器--避免发生shuffle),避免大量的数据交换
			--表大小悬殊较大时使用
		--两表较大时:先shuffle(重分区--使用相同分区器--key相同分到同一个分区--可避免发生shuffle)--再sort(避免多次遍历)--join
	--join底层实现:
自定义函数(重点):一对一,一个输入对应一个输出
自定义聚合函数(重点):
	--强类型 :DS
	--弱类型 :DF
开窗函数:
	--原因:既需要聚合数据,又不会丢失本身的细节:
	--分组topN--最经典应用场景
broadcastTest多次迭代版本


​	
​	

理解:
1,对DataFrame的认识 
2,RDD如何转化成DataFrame

1，对Dataset数据集的认识 
	--创建,处理方式,定义概念(强类型--错误语法在编译阶段即可查出)
2，对比RDD,Dataset,Dataframe
	--数据抽象,并行处理,API(T/A)
	--RDD:数据集不是严格结构化的,DS/DF:结构化数据集
	--创建方式:
	--编程方式:
	--相互转换
	--DS的两个子集
	
1，窗口函数的使用场景以及常用的窗口函数 
2，spark sql 底层实现join的三种方式	
	


代码复习:项目名:scala111
test0603:broadcast12/13/14  --三个版本(DSL/SQL/Broadcast)
test0531:broadcastTest11  --spark-core版本
test0530:IPSum10 Address11  --spark-core习题  11疑问

sparkday08 UDAFDemo1/2/3  --3有问题


反射/样例类/自定义函数/迭代器/序列化
SparkSQL函数全集--时间窗口