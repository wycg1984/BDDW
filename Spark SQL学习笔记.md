# Spark SQL及Streaming学习笔记

spark sql 
jdk1.8
spark-3.0.0-bin-hadoop3.2.tgz
启动本地环境 运行 spark-shell.cmd
bin目录创建文件夹input
在文件夹input创建wordcount.txt
读取文件

```scala
sc.textFiles("input/word.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

读取json文件

```scala
spark.read.json("input/user.json")
```

读取json文件创建DataFrame

```scala
val df = spark.read.json("input/user.json")
```

展示

```scala
df.show
```

使用
对DataFrame创建一个临时表

```scala
df.createTempView("user")
//或
df.createOrReplaceTempView("user")
```

通过SQL语句实现查询全表

```scala
spark.sql("select * from user").show
spark.sql("select age from user").show
spark.sql("select avg(age) from user").show
```

对DataFrame创建一个全局表

```scala
df.createGlobalTempView("user")
df.createOrReplaceGlobalTempView("user")
```

通过SQL语句实现查询全表

```scala
spark.newSession.sql("select * from global_temp.user").show
```

## DSL语法

创建一个DataFrame
val df = spark.read.json("data/user.json")
查看DataFrame的Schema信息
df.printSchema
只查看username列数据
df.select("username").show 
查看username列数据以及age+1数据
df.select($"username",$"age"+1).show
df.select('username,'age+1).show
$表示引用数据
查看age大于30的数据
df.filter($"age">30).show
filter过滤数据
df.filter('age>30).show
按照age分组，查看数据条数
df.groupBy("age").count.show

RDD转换为DataFrame
val rdd = sc.makeRDD(List(1,2,3,4))
val df = rdd.toDF("id")
df.show
toDF(列名1，列名2)
DataFrame转换为RDD
df.rdd

DataSet是具有强类型的数据集合，需要提供对应的类型信息。
创建DataSet
1)通过样例类序列创建DataSet
case class Person(name:String,age:Long)  --构建一个样例类
val list = List(Person("zhangsan",30),Person("lisi",20))
val ds = list.toDS
ds.show 

DataFrame转换为DataSet  as[样例类]
case class Emp(age:Long,username:String)
val ds = df.as[Emp]
ds.show 

DataSet转换为DataFrame toDF 
ds.toDF

RDD转换为DataSet
val rdd = sc.makeRDD(List(Emp(30,"zhangsan"),Emp(20,"lisi")))
val ds = rdd.toDS

IDEA创建SparkSQL环境对象
1.创建SparkSQL的运行环境
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
val spark = new SparkSession.builder().config(sparkConf).getOrCreate()

2.执行逻辑操作
DataFrame基本操作

//DataFrame
val df:DataFrame = spark.read.json(path="datas/user.json")
df.show()
//DataFrame =>SQL
df.createOrReplaceTempView("user")
spark.sql("select * from user").show 
spark.sql("select age,username from user").show 
spark.sql("select avg(age) from user").show 
//DataFrame => DSL
//在使用DataFrame时，如果涉及到转换操作，需要引入转换规则
import spark.implicits._
df.select("username","age").show
df.select($"age"+1).show
df.select('age+1).show

//DataSet 
//DataFrame是特定泛型的DataSet
val seq = Seq(1,2,3,4)
val ds:DataSet[Int] = seq.toDS()
ds,show()

//RDD <=> DataFrame 
val rdd = spark.sparkContext.makeRDD(List((1,"zhangsan",30),(2,"lisi",20)))
val df:DataFrame = rdd.toDF("id","name","age")
val rowRDD:RDD[Row] = df,rdd 

//DataFrame  <=> DataSet 
val ds:DataSet[User] = df.as[User] 
val df1 = ds.toDF()

//RDD <=> DataSet 
val ds1:DataSet[User] = rdd.map{
	case(id,name,age) => {
		User(id,name,age)
	}
}.toDS()

val userRDD:RDD[User] = ds1.rdd
3.关闭环境
spark.close()


case class User(id:Int,name:String,age:Int)


UDF函数 用户自定义函数
用户可以通过spark.udf功能添加自定义函数

1.创建SparkSQL的运行环境
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL")
val spark = new SparkSession.builder().config(sparkConf).getOrCreate()

val df = spark.read.json("data/user.json")
df.createOrReplaceTempView("user")

spark.udf.register("prefixName",(name:String)=>{
	"Name:"+name
})
spark.sql("select age,prefixName(username) from user").show

spark.close()

UDAF函数
1)弱类型函数实现
1.创建SparkSQL的运行环境
2.执行业务逻辑

3.关闭环境

2)强类型函数实现
1.创建SparkSQL的运行环境
2.执行业务逻辑


3.关闭环境

/**
* 自定义聚合函数类：计算年龄的平均值
	1.继承UserDefinedAggregateFunction
	2.重写方法
	*/
	class MyAvgUDAF extends UserDefinedAggregateFunction{
	

}
3)早期强类型函数实现
1.创建SparkSQL的运行环境
2.执行业务逻辑


3.关闭环境


数据读取与保存
1.通用的加载和保存方式
SparkSQL 默认读取和保存的文件格式为 parquet(列式存储的格式)

1)加载数据
spark.read.load 是加载数据的通用方法
val df = spark.read.load("examples/src/main/resources/user.parquet")

val df = spark.read.format("json").load("data/user.json")
val df = spark.read.json("data/user.json")
2)保存数据
df.write.save 是保存数据的通用方法
df.write.format("json").save("outut") 

df.write.mode("append").json("/opt/module/data/output")
2.Pparquet
Parquet 是一种能够有效存储嵌套数据的列式
存储格式。
数据源为 Parquet 文件时，Spark SQL 可以方便的执行所有的操作，不需要使用 format。
修改配置项 spark.sql.sources.default，可修改默认数据源格式。

3.JSON

```scala
spark.sql("select * from json.`data/user.json`").show 
```

4.CSV

```scala
spark.read.format("csv").option("sep", ";").option("inferSchema",
"true").option("header", "true").load("data/user.csv")
```

5.MySQL

6.Hive







数据处理的方式角度
流式(Streaming)数据处理
批量(batch)数据处理

数据处理延迟的长短
实时数据处理：毫秒级别
离线数据处理：小时or天级别

SparkStreaming 准实时(秒，分钟)，微批次的数据处理框架。设定时间来处理数据


1.WordCount 解析



2.DStream 创建



3.DStream 转换



4.DStream 输出


5.优雅关闭

6.SparkStreaming 案例实操