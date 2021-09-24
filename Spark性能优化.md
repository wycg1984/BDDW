

# 基于Spark的数据处理面对数据倾斜实战调优

# 1、数据倾斜

定义：指的是，在**并行**处理海量数据过程中，由于**某个**或者**某些**分区的数据**显著多于**其它分区，从而使得该部分数据的处理速度过慢成为整个数据集处理的**瓶颈**，此时我们说数据处理过程中产生了数据倾斜。

有什么危害？

1）[轻]违背分布式计算的初衷

2）[中]处理时间远远高于其它的任务，后续的计算也受影响，集群资源的浪费

3）[重]导致整个应用失败

```scala
object WordCount{
    def main(args: Array[String]): Unit = {
        val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("word_count")
        val sparkContext = new SparkContext(sparkConf)
        val sourceFilePath: String = "源文件位置"
        val sinkFilePath: String="目标文件位置"
        
        sparkContext.textFile(sourceFilePath)
        	.flatMap(_.split(" "))
        	.map((_, 1))
        	.reduceByKey(_ + _)
        	.saveAsTextFile(sinkFilePath)
        
        sparkContext.stop()
    }
    
}
```

运行情况  用块的大小类比数据量的大小

![image-20210823150139853](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823150139853.png)

![image-20210823150452238](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823150452238.png)

![image-20210823150753849](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823150753849.png)

数据倾斜可能产生的原因

1）Shffule Write导致的数据倾斜

2）Shffule Read导致的数据倾斜

3）Shffule Write和Shffule Read同时导致的数据 倾斜

# 2、何因&何解

![image-20210823151230197](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823151230197.png)

![image-20210823151629842](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823151629842.png)

![image-20210823151825844](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823151825844.png)

![image-20210823152030949](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823152030949.png)

![image-20210823152249373](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823152249373.png)

![image-20210823152630058](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823152630058.png)

![image-20210823152654368](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823152654368.png)

![image-20210823153013508](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823153013508.png)

怎么确定数据倾斜的Key

```scala
/**
* 确定数据倾斜的Key
*/
object DeterminKeyOfSkewData{
    def main(args: Array[String]): Unit = {
        val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("word_count")
        val sparkContext = new SparkContext(sparkConf)
        
        val rdd: RDD[String] = sparkContext.makeRDD(
        	List("",),2
        )
        
        rdd.sample(false,0.5)
        	.map((_, 1))
        	.reduceByKey(_ + _)
        	.map(tuple =>(tuple._2,tuple._1))
        	.sortByKey(false)
        	.take(2)
        	.foreach(printIn)
        
    }
}
```

![image-20210823154204163](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823154204163.png)

![image-20210823154452859](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823154452859.png)





# 3、最后 的最后

![image-20210823160059430](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823160059430.png)

![image-20210823160917296](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823160917296.png)

![image-20210823161043872](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823161043872.png)

![image-20210823161533968](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823161533968.png)

![image-20210823161549733](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823161549733.png)

![image-20210823161846629](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210823161846629.png)

