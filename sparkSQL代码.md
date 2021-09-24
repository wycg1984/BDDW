# sparkSQL代码

```scala
package sparkday06

import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, SparkSession}

object DataFrameDemo1 {
  // RDD => DataFrame(直接指定)
  def main(args: Array[String]): Unit = {
    //创建sparksql的入口，2.0 sparksession 1.6 sqlContext hiveContext
    //创建一个sparksession1
    val sparkSession = SparkSession.builder().appName("sparksqlDemo")
      .master("local").getOrCreate()
    //创建一个sparksession2
    //    val conf = new SparkConf().setAppName("sparksqlDemo2").setMaster("local")
    //    val sparkSession1 = SparkSession.builder().config(conf).getOrCreate()
    //    //兼容hive
    //    val sparkSessionH = SparkSession.builder().appName("sparksqlDemo3")
    //      .master("local").enableHiveSupport().getOrCreate()

    //创建一个RDD
    val sc = sparkSession.sparkContext
    val peopleRDD: RDD[String] = sc.textFile("D://people.txt")
    //创建一个DataFrame
    val tupleRDD = peopleRDD.map(line=>{
      val arr = line.split(",")
      (arr(0),arr(1).trim.toInt)
    })
    import sparkSession.implicits._
    val dataFrame: DataFrame = tupleRDD.toDF("name","age")
    dataFrame.show()
    sc.stop()
    sparkSession.stop()

  }

}


package sparkday06

import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, SparkSession}

//通过反射方式RDD=>dataFrame
object DFDemo2 {

  def main(args: Array[String]): Unit = {
    //创建一个sparksession1
    val sparkSession = SparkSession.builder().appName("sparksqlDemo")
      .master("local").getOrCreate()

    val sc = sparkSession.sparkContext
    val rdd = sc.textFile("D://people.txt")
    //由字符串类型的数据集，变成对象的数据集
    val peopleRDD:RDD[People]=rdd.map(line=>{
      val arr = line.split(",")
      val name = arr(0)
      val age = arr(1).toInt
      People(name,age)
    })
    
    //rdd=>dataframe
    import sparkSession.implicits._
    val dF: DataFrame = peopleRDD.toDF
    //注册一张表,基于表操作对应dataframe中的数据
    dF.createOrReplaceTempView("t_people")
    
    //对df数据进行排序
    val sql ="select * from t_people order by age desc"
    val dataFrame: DataFrame = sparkSession.sql(sql)
    dataFrame.show()
    sc.stop()

  }

}

case class People(name:String,age:Int)



package sparkday06

import org.apache.spark.rdd.RDD
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}

//RDD=>dATAFRAME
//dataframe = rdd + scheme
object DFDemo3 {

  def main(args: Array[String]): Unit = {
    //创建一个sparksession1
    val sparkSession = SparkSession.builder().appName("sparksqlDemo")
      .master("local").getOrCreate()

    val sc = sparkSession.sparkContext
    val rdd = sc.textFile("D://people.txt")
    //由字符串类型的数据集，变成Row类型的数据集
    val rowRDD:RDD[Row]=rdd.map(line=>{
      val arr = line.split(",")
      val name = arr(0)
      val age = arr(1).toInt
      Row(name,age)
    })
    //创建给RDD对应的scheme信息
    val scheme:StructType = StructType(List(
      StructField("name",StringType,true),
      StructField("age",IntegerType,true)
    ))
    
    //rdd scheme进行关联生成dataframe
    val dataFrame: DataFrame = sparkSession.createDataFrame(rowRDD,scheme)
    //dataframe两种编程风格 sql 和DSL（基于API编程）
    //sql 先生成一个视图表，基于表写sql语句
    //dsl 直接使用dataframe的API
    val frame: DataFrame = dataFrame.select("name","age")
    //排序
    import sparkSession.implicits._
    val ordered: Dataset[Row] = frame.orderBy($"age" desc)
    ordered.write.json("jsonres")
    ordered.show()
    sc.stop()

  }

}



package sparkday07

import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}
import org.apache.spark.sql.types.{IntegerType, StringType, StructField, StructType}

object TransformDemo1 {
  def main(args: Array[String]): Unit = {

    //RDD DF DS 互相转换
    //RDD -->DF(自己指定/反射方式/通过编码定义scheme信息+RDD)
    
    //DF -->RDD
    //创建一个sparksession1
    val sparkSession = SparkSession.builder().appName("sparksqlDemo")
      .master("local").getOrCreate()
    
    val sc = sparkSession.sparkContext
    val rdd = sc.textFile("D://people.txt")
    //由字符串类型的数据集，变成Row类型的数据集
    val rowRDD:RDD[Row]=rdd.map(line=>{
      val arr = line.split(",")
      val name = arr(0)
      val age = arr(1).toInt
      Row(name,age)
    })
    //创建给RDD对应的scheme信息
    val scheme:StructType = StructType(List(
      StructField("name",StringType,true),
      StructField("age",IntegerType,true)
    ))
    
    import sparkSession.implicits._
    //rdd scheme进行关联生成dataframe
    val dataFrame: DataFrame = sparkSession.createDataFrame(rowRDD,scheme)
    //DF -->RDD ,使用RDD算子对当前数据集进行计算
    val dfToRDD = dataFrame.rdd
    //DF -->DS

   // val unit :Dataset[People]= dataFrame.as[Pepple]
    //DS-->DF
//    val dsToDF:DataFrame = dfToDs.toDF
//    //DS --> RDD
//    val dsToRDD = dfToDs.rdd

    sc.stop()
    sparkSession.stop()

  }
}



package sparkday07

import java.util.Properties

import org.apache.spark.sql.{DataFrame, SparkSession}

object JsonFileDemo2 {
  def main(args: Array[String]): Unit = {
    val sparkSession = SparkSession.builder().appName("JSon").master("local").getOrCreate()

    val frame:DataFrame = sparkSession.read.json(args(0))
    import sparkSession.implicits._
    val resdf = frame.where($"age">20)

//    //write--action算子--保存文件
//    frame.write.csv("csvres")
//    val frame1:DataFrame = sparkSession.read.csv("csvres")
//    frame.write.format("parquet").save("pathInfo")
//    frame.write.parquet("parquetres")
//    sparkSession.read.parquet("parquet")
//
//    //通过jdbc读数据库某张表的数据
//    val prop = new Properties()
//    prop.put("user","root")
//    prop.put("password","123456")
//    prop.put("driver","com.mysql.jdbc.Driver")
//
//    //写数据库
//    //mode写模式 看官网 Save Mode "error"  "append" "overwrite" "ignore"
//    resdf.write.mode("append").jdbc("","t_people",prop)
//    //读数据库表中的数据
//    sparkSession.read.jdbc("url","t_people",prop)
//    sparkSession.read.format("jdbc").options(
//      Map("url"->"urlInfo"
//        ,"driver"->"com.mysql.jdbc.Driver"
//        ,"user"->"root"
//        ,"password"->"123456"
//        ,"dbtable"->"t_people")
//    ).load()


    //打印scheme信息(结构化信息--字段,类型,约束...)
    resdf.printSchema()
    //打印SQL,api=RDD 从逻辑计划到物理计划的执行流程,where调用的是filter
    resdf.explain()
    resdf.show()
    sparkSession.stop()


  }
}



package sparkday07

import org.apache.spark.sql.{DataFrame, Dataset, SparkSession}
import java.util.Properties

object DSWordCount3 {
  def main(args: Array[String]): Unit = {
    val sparkSession =SparkSession.builder().appName("").master("").getOrCreate()

    val lines:Dataset[String] = sparkSession.read.textFile("")
    import sparkSession.implicits._
    val words:Dataset[String] = lines.flatMap(_.split(" "))
    //SQL版本,t_word里面只有一列:单词
    words.createOrReplaceTempView("t_words")
    
    //写一个SQL,统计t_word里的各个单词数
    words.show()
    val sql = "select value,count(*) counts from t_words group by value order by counts desc"
    val frame: DataFrame = sparkSession.sql(sql)
    
    //DSL 版本
    //使用sparkSQL的内置函数(上百个),例如agg聚合函数
    import org.apache.spark.sql.functions._
    val res = words.groupBy($"value" as "word").count().sort($"count" desc)
    val res1 = words.groupBy($"value" as "word").agg(count("*") as "count").orderBy($"count" desc)


  }
}




package spark.sparkday07

import org.apache.spark.sql.{DataFrame, RelationalGroupedDataset, Row, SparkSession}

object DSApiDemo4 {
  def main(args: Array[String]): Unit = {
    //创建一个df ds 数据集
    val sparkSession = SparkSession.builder().appName("JSon").master("local").getOrCreate()

    val frame:DataFrame = sparkSession.read.json(args(0))
    
    //action
    //show 默认显示20行,可指定行数 --truncate是否截取,
    frame.show()
    frame.show(10)
    //是否对显示的内容进行截取,默认取前20个字符 ...
    frame.show(true)
    
    //collect 返回的是一个数组类型的集合对象
    frame.collect()
    //返回list集合对象
    frame.collectAsList()
    //返回具体某一列数据的统计信息:总数,均值,方差,最大值,最小值
    frame.describe("age").show()
    //取第一行
    frame.first()
    //取前n行  如何自动补全
    frame.take(3)
    frame.head()
    frame.head(5)
    frame.takeAsList(3)
    frame.limit(3)
    
    //常用的与条件相关的过滤
    import sparkSession.implicits._
    frame.where($"age" > 20).show() //where是transformation类型的
    frame.filter($"age" > 20).show()
    //选某一列
    frame.select("age").show() //对列进行过滤 前两者是对行进行过滤
    frame.col("age") //选某一列
    frame.apply("name")
    //drop相关的
    //删除某些列之后的结果
    frame.drop("name")
    //按列去重
    frame.dropDuplicates("name")
    //排序分组相关
    frame.sort("age").show() //会将DF转换为DS
    frame.orderBy("age")
    //根据分区,在分区内排序
    val value = frame.sortWithinPartitions()
    val res: RelationalGroupedDataset = frame.groupBy("name")
    val res1: RelationalGroupedDataset = frame.groupBy(frame.col("age"))
    //按行去重
    frame.distinct()
    //集合操作
    frame.union(frame)
    frame.join(frame)
    frame.crossJoin(frame) //笛卡尔积
    frame.intersect(frame)
    //增加一列
    frame.withColumn("gender",frame.col("age"))
    //行转列
    //frame.explode_
    
    sparkSession.stop()

  }
}


package spark.sparkday07

import org.apache.spark.sql.SparkSession

case class Student(subject:String,name:String)

object WindowDemo5 {
  //分组topN
  //row_number--不并列,顺次排序  rank--有并列,有占位   dense_rank--有并列,不占位
  //avg sum ...
  //over  partition by
  //学科 姓名 rank
  def main(args: Array[String]): Unit = {

    val sparkSession = SparkSession.builder().appName("win").master("local[*]").getOrCreate()
    //
    val lines = sparkSession.sparkContext.textFile("C:\\Users\\acer\\Desktop\\win.txt")
    val studentRDD = lines.map(line=> {
      val arr = line.split(" ")
      //返回对象,需要先定义样例类
      Student(arr(0),arr(1))
    })
    import sparkSession.implicits._
    val dataFram = studentRDD.toDF()
    
    //注册一个临时表
    dataFram.createOrReplaceTempView("t_student")
    //根据科目分组,统计每个人在每个科目的票数
    val dataFrame1 = sparkSession.sql("select subject,name,count(*) counts from t_student group by subject,name")
    //根据票数排序,显示每个人的排名结果,最后取每组的top1
    dataFrame1.createOrReplaceTempView("t_student_count")
    sparkSession.sql("select * from (select *,row_number() over(partition by subject order by counts desc) rank from t_student_count) t_temp where rank < 2").show()
    
    sparkSession.stop()

  }
}


package spark.sparkday08

import org.apache.spark.sql.{DataFrame, SparkSession}


//使用场景:一对一,一个输入,传入自定义函数,得到一个计算函数
object UDFDemo1 {
  def main(args: Array[String]): Unit = {
    val sparkSession = SparkSession.builder().appName("UDFDemo").master("local[*]").getOrCreate()
    val frame: DataFrame = sparkSession.read.json(args(0))
    //自定义函数,处理姓名,注册自定义函数
    sparkSession.udf.register("nameProcess",(x:String)=>x.charAt(0).toUpper+x.substring(1,x.length))
    frame.createOrReplaceTempView("t_people")
    sparkSession.sql("select nameProcess(name),age from t_people ").show()

    sparkSession.stop()

  }
}


package spark.sparkday08

import org.apache.spark.sql.{DataFrame, Row, SparkSession}
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types._

//自定义弱类型,使用DF
//多个输入对应一个输出,实现计算年龄的均值
class UDAF1 extends UserDefinedAggregateFunction{
  //定义输入参数的数据的scheme信息
  override def inputSchema: StructType = {
    StructType(List(StructField("age",IntegerType,true))) //参数要求为集合类型
  }

  //分区中聚合时产生的中间结果的scheme
  //(age+age...,1+1+...)
  override def bufferSchema: StructType = {
    StructType(StructField("sum",IntegerType)::StructField("count",IntegerType)::Nil)
    //StructType(List(StructField("")))
    //new StructType().add("sum",IntegerType).add("count",IntegerType)
  }

  //定义最后聚合返回结果的数据类型
  override def dataType: DataType = DoubleType

  //多个确定的输入输出的结果是否是确定的
  override def deterministic: Boolean = true

  //初始化中间结果对象
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    //初始化年龄聚合
    buffer(0) = 0
    //初始化人的个数
    buffer(1) = 0
  }

  //处理分区里的每一条数据,聚合到中间结果
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    if (!input.isNullAt(0)) {
      buffer(0) = buffer.getInt(0) + input.getInt(0)
      buffer(1) = buffer.getInt(1) + 1
    }
  }

  //分区之间的聚合,实现每个分区聚合之后的再汇总
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0) = buffer1.getInt(0) + buffer2.getInt(0)
    buffer1(1) = buffer1.getInt(1) + buffer2.getInt(1)
  }

  //获取返回值-聚合之后的结果
  override def evaluate(buffer: Row): Double =buffer.getInt(0)/buffer.getInt(1).toDouble

}

object UDAFDemo2 {
  def main(args: Array[String]): Unit = {
    val sparkSession = SparkSession.builder().appName("UDFDemo").master("local[*]").getOrCreate()
    val frame: DataFrame = sparkSession.read.json(args(0))
    //先注册
    sparkSession.udf.register("myAvg",new UDAF1)
    frame.createOrReplaceTempView("t_people")
    sparkSession.sql("select myAvg(age) from t_people").show()

    sparkSession.stop()

  }
}


package spark.sparkday08

import org.apache.spark.sql.{DataFrame, Encoder, Encoders, SparkSession}
import org.apache.spark.sql.expressions.Aggregator
import sparkday06.People

//输入结果类型(表中的每条数据),中间结果类型,返回结果类型
case class Average(var sum:Int,var count:Int)
class UDAF2 extends Aggregator[People,Average,Double]{
  //初始化中间结果
  override def zero: Average = Average(0,0)

  //分区内聚合,把每一条数据聚合到中间结果对象
  override def reduce(b: Average, a: People): Average = {
    b.sum = b.sum + a.age
    b.count = b.count + 1
    b
  }

  //分区结果汇总
  override def merge(b1: Average, b2: Average): Average = {
    b1.sum = b1.sum + b2.sum
    b1.count = b1.count + b2.count
    b1
  }

  //返回聚合结果
  override def finish(reduction: Average): Double = reduction.sum/reduction.count.toDouble

  override def bufferEncoder: Encoder[Average] = Encoders.product

  override def outputEncoder: Encoder[Double] = Encoders.scalaDouble
}

object UDAFDemo3 {
  def main(args: Array[String]): Unit = {
    val sparkSession = SparkSession.builder().appName("UDFDemo").master("local[*]").getOrCreate()
    val frame: DataFrame = sparkSession.read.json(args(0))
    frame.as("Average")
    //先注册
//    sparkSession.udf.register("Average",new UDAF2)
    frame.createOrReplaceTempView("t_people")
    sparkSession.sql("select myAvg(age) from t_people").show()

    sparkSession.stop()

  }
}
```

