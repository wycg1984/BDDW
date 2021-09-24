# Spark Streaming笔记

## 数据处理的方式：

流式（Streaming）数据处理

批量（batch）数据处理



## 数据处理延迟的长短：

实时数据处理：毫秒级别

离线数据处理：小时 or 天级别



SparkStreaming 准实时（秒、分钟），微批次（时间）的数据处理框架。设定时间来处理数据。



## DStream(离散化流)

```scala
//TODO 创建环境对象
//StreamingContext创建时需要两个参数
//第一个参数表示环境配置
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
//第二个参数表示批量处理的周期（采集周期）
val ssc = new StreamingContext(sparkConf,Seconds(3))
val word = lines.flatMap(_.split(" "))

val wordToOne = word.map((_,1))
//聚合
val wordToCount:DStream[(String,Int)] = wordToOne.reduceByKey(_+_)

wordToCount.print()
//TODO 逻辑处理
//获取端口数据
val lines:ReceiverInputDStream[String] = ssc.socketTextStream("localhost",9999)


//TODO 关闭环境'
//由于SparkStreaming采集器是长期执行的任务，所以不能直接关闭
//ssc.Stop()

//1.启动采集器
ssc.start()
//2.等待采集器的关闭
ssc.awaitTermination()
```



## DSream创建

RDD队列

可以通过ssc.queueStream(queueOfRDDs)来创建DStream

```scala
//TODO 创建环境对象
//StreamingContext创建时需要两个参数
//第一个参数表示环境配置
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
//第二个参数表示批量处理的周期（采集周期）
val ssc = new StreamingContext(sparkConf,Seconds(3))
val word = lines.flatMap(_.split(" "))

val wordToOne = word.map((_,1))
//聚合
val wordToCount:DStream[(String,Int)] = wordToOne.reduceByKey(_+_)

wordToCount.print()
//TODO 逻辑处理
//创建RDD队列
val rddQueue = new mutable.Queue[RDD[Int]]()
//创建QueueInputDStream
val inputStream = ssc.queueStream(rddQueue,oneAtATime = false)
//处理队列中的RDD数据
val mappedStream = inputStream.map((_,1))
val reduceStream = mappedStream.reduceByKey(_ + _)
//打印结果
reduceStream.print()
//1.启动采集器
ssc.start()
//循环创建并向RDD队列中放入RDD
for(i< -1 to 5)
{
    rddQueue += ssc.sparkContext.makeRDD(seq = 1 to 300,numSlices = 10)
    Thread.sleep(2000)
}
//2.等待采集器的关闭
ssc.awaitTermination()
```

自定义数据源

需要继承Receiver，并实现onStart、onStop方法来自定义数据源采集

```scala
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
val ssc = new StreamingContext(sparkConf,Seconds(3))
val messageDS: ReceiverInputDStream[String] = ssc.receiverStream(new MyReceiver())
messageDS.print()

ssc.start()
ssc.awaitTermination()
/*
自定义采集器
1.
*/
class MyReceiver extends Reveiver[String](StorageLevel.MEMORY_ONLY){
	private var flag = true 
    override def onStart():Unit = {
        new Thread(new Runnable{
            while( flag ){
                val message = "采集的数据为：" + new Random().nextInt(10).toString
                store(message)
                Thread.sleep(500)
            }
        }).start()
        
    }
	override def onStop():Unit = {
        flag = false;
        
        
    }

}
```

## Kafka数据源

kafka0-10 Direct模式

```scala
//1.创建SparkConf
val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming")
//2.创建StreamingContext
val ssc = new StreamingContext(sparkConf,Seconds(3))
//3.定义kafka参数
val kafkaPara: Map[String,Object] = 
//4.读取kafka数据创建DStream
val kafkaDataDS: InputDStream[ConsumerRecord[String,String]]= KafkaUtils.createDiectStream[String,String](ssc,LocationStrategies.PreferConsistent,ConsumerStrategies.Subscribe[String,String](Set(""),kafkaPara))
//5.将每条消息的KV取出
kafkaDataDS.map(_.value()).print()
//
ssc.start()
ssc.awaitTermination()
```



## DStream转换

状态操作
