# spark_core程序

```java
package sparkday01
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object SparkWordCounts {
  def main(args: Array[String]): Unit = {
    //创建一个配置信息对象
    //设置的应用程序名称
    val conf:SparkConf = new SparkConf()
      .setAppName("scala_wordcount")
      .setMaster("local[*]")
    //local 使用本地一个线程模拟机群执行任务
    //local[num]:使用num个线程去模拟集群执行任务
    //local[*] 使用所有空闲的线程去模拟集群执行任务
    //spark上下文对象,执行spark程序的入口
    val sc:SparkContext = new SparkContext(conf)
    //读取数据源
    //通过参数设置源文件路径,HDFS文件/本地文件
    //RDD集合里的每一个元素即为文件里的一行
    //textFile对于一个或者多个文件
    val lines: RDD[String] =  sc.textFile(args(0))
    //单词的rdd集合
    val words: RDD[String]=  lines.flatMap(_.split(" "))
    val tupleWords: RDD[(String,Int)] =  words.map((_,1))
    //先根据Word进行分组,然后对每组的value集合元素进行reduce聚合
    val reduced:RDD[(String,Int)] = tupleWords.reduceByKey(_ + _)
    //根据每一个单词出现的次数进行排序
    val res: RDD[(String,Int)] = reduced.sortBy(_._2,false) //逆序 ,默认是升序
    //结果打印,将结果从executor汇聚到driver端
    println(res.collect().toBuffer)
    //结果保存到文件系统
    res.saveAsTextFile(args(1))
    //资源释放
    sc.stop()
  }
}



package sparkday02

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

object CombineByKeyDemo1 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("CombineByKey").setMaster("local[*]")
    val sc:SparkContext = new SparkContext(conf)
    //求各科平均成绩
    val rdd = sc.parallelize(List(("english",60),("math",74),("chinese",90),("math",86),("english",90),("math",78)))
    //使用combineByKey
    val reducedres:RDD[(String,(Int,Int))] = rdd.combineByKey(
      (score:Int) => {(score,1)},
      (accres:(Int,Int),score:Int)=>{(accres._1+score,accres._2+1)},
      (accres1:(Int,Int),accres2:(Int,Int))=>{(accres1._1+accres2._1,accres1._2+accres2._2)}
    )
    //计算平均成绩
    val res:RDD[(String,Double)] = reducedres.mapValues((x:(Int,Int))=>x._1/x._2.toDouble)
    println(res.collect().toBuffer)
  }
}


package sparkday03
import org.apache.spark.{SparkConf, SparkContext}
object TakeDemo {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("CombineByKey").setMaster("local[*]")
    val sc:SparkContext = new SparkContext(conf)
    //求各科平均成绩
    val rdd = sc.parallelize(List(10,20,60,50,40))
    val res1:Array[Int] = rdd.take(3)
    //数据量小时使用,数据会传输到driver端
    val res2:Array[Int] = rdd.takeOrdered(3)
    //排序后取topN
    val res3 = rdd.top(3)
    //等同于take(1)
    val res4 = rdd.first()
    val res5 = rdd.takeSample(false,3,100L)
    println(res1.toBuffer+":"+res2.toBuffer+":"+res3.toBuffer +":"+res4+":"+res5.toBuffer
//    println(rdd.count()) //元素个数
//    //foreach与map的对比
//    rdd.foreach(println)
//    rdd.foreachPartition(_.foreach(println))
//    rdd.saveAsTextFile("")
    //rdd.iterator().toList
  }
}



package sparkday05
import org.apache.spark.{SparkConf, SparkContext}
object BroadcastDemo1 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("broadcast").setMaster("local")
    val sc:SparkContext = new SparkContext(conf)
    val rdd = sc.parallelize(List(10,20,30,40,50,60))
    val arr = Array(1,2,3,4,5,6)
    //定义一个变量,广播变量,将数据集进行广播
    //如果要广播的是RDD数据集,则需要先收集到Driver端,然后再广播
    //广播变量在Executor端只能读取
    val broadcast = sc.broadcast(arr)

    val res = rdd.map(x=>{
      val sum = broadcast.value.sum
      x + sum
    })
    res.foreach(println)
    sc.stop()

  }
}



package sparkday05
import org.apache.spark.util.AccumulatorV2
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable
//使用自定义累加器实现WordCount功能
class myAccumulator extends AccumulatorV2[String,mutable.HashMap[String,Int]] {
  private  val hashmapAcc = new mutable.HashMap[String,Int]()
  //当前累加器初始值是否为0或者Nil
  //检测累加器是否为空
  override def isZero: Boolean = {
    hashmapAcc.isEmpty
  }
  //做一个累加器的拷贝,返回一个新的累加器
  override def copy(): AccumulatorV2[String, mutable.HashMap[String, Int]] = {
    val newAcc = new myAccumulator()
    hashmapAcc.synchronized {
      newAcc.hashmapAcc ++= hashmapAcc
    }
    newAcc
  }
  //重置累加器,清空数据
  //重置之后,调用isZero方法,保证返回True
  override def reset(): Unit = {
    hashmapAcc.clear()
  }
  //
  override def add(word: String): Unit = {
    hashmapAcc.get(word) match {
      case Some(v1) => hashmapAcc += ((word, v1 + 1))
      case None => hashmapAcc += ((word, 1))
    }
  }
  //
  override def merge(other: AccumulatorV2[String, mutable.HashMap[String, Int]]): Unit = {
    other match {
      case acc:AccumulatorV2[String,mutable.HashMap[String,Int]] =>{
        for ((k,v)<-acc.value){
          hashmapAcc.get(k) match {
            case Some(newv) => hashmapAcc += ((k,v+newv))
            case None => hashmapAcc += ((k,v))
          }
        }
      }
    }
  }
  //
  override def value: mutable.HashMap[String, Int] = hashmapAcc
}
object DefineAccumulatorDemo3 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("accumulate").setMaster("local")
    val sc:SparkContext = new SparkContext(conf)
    val rdd = sc.parallelize(List("scala","java","c","scala","scala"))
    //创建累加器,然后进行注册
    val acc = new myAccumulator()
    sc.register(acc,"wordCount")
    rdd.foreach(x=>acc.add(x))
    //通过value访问最后累加的结果
    println(acc.value)
  }
}


package sparkday05
import java.util.concurrent.atomic.LongAccumulator

import org.apache.spark.util.CollectionAccumulator
import org.apache.spark.{SparkConf, SparkContext}
object AccumulateDemo2 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("accumulate").setMaster("local")
    val sc:SparkContext = new SparkContext(conf)
    val rdd = sc.parallelize(List(10,20,30,40,50,60))
    //long类型的累加器
//    def longAccumulator(name:String):LongAccumulator ={
//      val acc = new Lo
//      sc.register(acc,name)
//      acc
//    }
//    val acc1 = longAccumulator("longAcc")
//    //在Executor端执行
//    rdd.foreach(x=>acc1.add(x))
//    //在driver端执行
//    println(acc1.value)
    //collect类型的累加器
    def collectAccumulator(name:String):CollectionAccumulator[Int] ={
      val acc = new CollectionAccumulator[Int]
      sc.register(acc,name)
      acc
    }
    val acc2 = collectAccumulator("collectAccumulator")
    rdd.foreach(x=>acc2.add(x))
    println(acc2.value)
  }
}



package sparkday05
import java.sql.DriverManager
import org.apache.spark.rdd.JdbcRDD
import org.apache.spark.{SparkConf, SparkContext}
object JDBCRDD5 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("accumulate").setMaster("local[4]")
    val sc:SparkContext = new SparkContext(conf)
    val jdbcurl = "jdbc:mysql://10.0.16.58:3306/mydb1?useUnicode=true&characterEncoding=utf8"
    val user ="root"
    val password = "20143650"
    //sql语句,要有一个范围
    val sql = "select * from  careers where cid >? and cid <? "
    //创建数据库连接
    val conn = ()=>{
      Class.forName("com.mysql.jdbc.Driver").newInstance()
      DriverManager.getConnection(jdbcurl,user,password)
    }
    val JRDD:JdbcRDD[(Int,String)] = new JdbcRDD(sc,conn,sql,1L,6L,1,resultSet=>{
      val cid = resultSet.getInt("cid")
      val cname = resultSet.getString("cname")
      (cid,cname)
    })
    JRDD.foreach(println)
    sc.stop()
  }
}


package sparkday05
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
object SortDemo6 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("accumulate").setMaster("local")
    val sc:SparkContext = new SparkContext(conf)
    //解决自定义类型的数据对象的排序
    //Java方法中会重写方法或定义对象
    val arr = Array(("kate",20,98),("mary",18,95),("rose",23,65),("candy",20,99))
    val rdd = sc.parallelize(arr)
    //对RDD数据进行排序,先按年龄,再按颜值排序
    val userRDD:RDD[User] = rdd.map(x=>{
      new User(x._1,x._2,x._3)
    })
    val sortedRDD = userRDD.sortBy(u=>u)
    println(sortedRDD.collect().toBuffer)
    val sortedRDD1 = rdd.sortBy(tuple=>new User(tuple._1,tuple._2,tuple._3))
    println(sortedRDD1.collect().toBuffer)
    sc.stop()
  }
}
//通过定义类,在类中实现对象的比较逻辑--模仿Java
//要加val
class User(val name:String,val age:Int,val faceValue:Int) extends Ordered[User] with Serializable {
  override def compare(that: User): Int = {
    if (this.age == that.age)
      //颜值高的在前面
      -(this.faceValue - that.faceValue)
    else
    //年龄小的在前面
      this.age - that.age
  }
  override def toString: String = s"name:$name,age:$age,faceValue:$faceValue"
}


package sparkday05
import org.apache.spark.{SparkConf, SparkContext}
object SortDemo7 {
  def main(args: Array[String]): Unit = {
    val conf:SparkConf = new SparkConf()
      .setAppName("accumulate").setMaster("local")
    val sc:SparkContext = new SparkContext(conf)
    //解决自定义类型的数据对象的排序
    //隐式转换
    val arr = Array(("kate",20,98),("mary",18,95),("rose",23,65),("candy",20,99))
    val rdd = sc.parallelize(arr)
    import SortRules.OrderUser
//    val sorted = rdd.sortBy(tuple=>User1(tuple._2,tuple._3))
//    println(sorted.collect().toBuffer)
  }
}
case class User1(age:Int,faceValue:Int)
object SortRules {
  implicit object OrderUser extends Ordering[User1] {
    override def compare(x: User1, y: User1): Int = {
      if (x.age == y.age)
        -(x.faceValue - y.faceValue)
      else
        x.age - y.age
    }
  }
}


package test0528

import java.net.URL

import org.apache.spark.{Partitioner, SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

import scala.collection.mutable

object AccessSortTest12 {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf()
      .setAppName("AccessSort").setMaster("local[*]")
    val sc: SparkContext = new SparkContext(conf)
    val lines = sc.textFile(args(0))
    val reduced: RDD[(String, Int)] = lines.map(x => {
      (x.split("\t")(1), 1)
    }).reduceByKey(_ + _)
    //(subject,count)=>(subject,(url,count))
    val subjectRDD: RDD[(String, (String, Int))] = reduced.map(x => {
      //学科下的模块
      val url = x._1
      //学科信息
      val subject = new URL(url).getHost
      //学科下具体模块的访问次数
      val count = x._2
      (subject, (url, count))
    }).cache()
    val subjects = subjectRDD.keys.distinct().collect()
    val myPatitioner = new SubjectPartition(subjects)
    val partitionedRDD = subjectRDD.partitionBy(myPatitioner)
    val res = partitionedRDD.mapPartitions(it => {
      it.toList.sortBy(_._2._2).reverse.take(3).iterator
    })
    res.saveAsTextFile(args(1))
    sc.stop()
  }
}
class SubjectPartition(subjects:Array[String]) extends Partitioner {
  //定义一个map，存储学科和分区号
  private val subjectIndex:mutable.HashMap[String,Int] = new mutable.HashMap[String,Int]()
  //定义一个变量，用来创建分区号
  var i = 0
  for(subject <- subjects){
    subjectIndex += (subject->i)
    i += 1
  }
  //定义分区数
  override def numPartitions: Int = subjects.length
  //返回分区号
  override def getPartition(key: Any): Int = subjectIndex.getOrElse(key.toString,0)
}
```

