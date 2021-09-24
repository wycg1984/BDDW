# spark简单整理

Spark:
第一天(19.05.27)
快;易用;通用;兼容性

通用: 五大核心组件:Spark Core ;Spark SQL(离线) ; Spark Streaming(实时) ; MLib ; GraphX ---后面四个都是最后在spark core上实现 

Application:将程序提交到集群--该程序就被称为Application,每个application启动一个进程,application之间相互独立
driver program--SparkContext 
Deploy mode:Local ; cluster  4040
Worker node:执行程序的结点
Executor:一个应用都会在一个work结点上启动一个Executor--每个应用都是独立的;不同进程之间数据是共享的
Task:具体的计算逻辑实现部分; 线程池中的一个线程
Job:SparkAPI有两种(action和transform):每个action都会触发一个Job,一个Application会对应多个Job; 
Stage:每个Job再进行分隔划分--根据shuffle对于算子进行判断是否分割 --每个stage对应一个task线程

shuffle时数据会落地到磁盘

对底层管理框架无感知

驱动程序与work中的Executor保持通信

RPC通信

spark-shell --master spark://mini1:7077
提交任务:spark-submit 

示例:mainClass/JAR包/master
./spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://mini1:7077 \
--executor-memory 512M \
--total-executor-cores 1 \
/apps/spark-2.2.0-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.2.0.jar 100

集群启动流程
	1，启动Master进程 
	2，Master开始解析conf目录的slaves配置文件，找到相应的Worker节点，开始启动Worker进程
	3，Worker进程开始向Master发送注册信息 
	4，Master接受到Worker的注册信息后并保存到内存和磁盘里，然后向Worker发送注册成功信息
	5，Worker开始和Master建立心跳，Master每次接收到心跳后更新WorkerInfo的最后一次心跳时间


任务提交流程 P13
	1,
	2,
	3,
	

spark提供得方法被称为算子

RDD:特点
弹性分布式数据集 --数据集合(可以容错) --并行执行
---弹性:数据集可以基于磁盘也可以基于内存;数据集大小是弹性的;容错机制
---分布式:分布于多个节点
---并行的
---可容错的

源码中的RDD描述:
RDD:特点
	是抽象的,没有具体数据;
	不可变的--RDD一旦创建则与具体的数据集合一一对应,不可改变;
	可以分区--分别进行计算(在每个worker结点上并行执行)
	
通过隐式转换--基于类型自动可以调用相应的方法

RDD:基本属性:
	每一个RDD对应一个分片列表--一组分片
	每一个RDD有一个函数--用于计算分片的
	形成一个依赖列表,后一个RDD依赖于前一个RDD,每一步都会产生一个新的RDD
	对key-value类型的数据会有一个分区器(hash分区/范围分区)--可选
	有一个最优位置列表--用来计算每个分片
	
RDD与集合的区别:--RDD的特点和属性

为什么要抽象出RDD这个概念:
	简化并发程序的编写:会在每一个worker上并行执行的
	单机版程序和RDD的并发版程序编写逻辑是一致的
	
spark程序:
	driver端和worker端的Executor进程的Task进程上(逻辑计算阶段,每一类Task的处理逻辑是一致的)
	


/apps/spark-2.2.0-bin-hadoop2.7/bin/spark-submit \
--class sparkday01.SparkWordCounts \
--master spark://mini1:7077 \
--executor-memory 512M \
--total-executor-cores 1 \
/root/Jar/scale111-1.0-SNAPSHOT.jar \
hdfs://mini1:9000/in/wordcount \
hdfs://mini1:9000/outSpark

# 注意输出路径应该是不存在的



Yarn模式:
./bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
--deploy-mode cluster \
--driver-memory 512M \
--executor-cores 512M \
--queue thequeue \
/apps/spark-2.2.0-bin-hadoop2.7/examples/jars/spark-examples_2.11-2.2.0.jar 100

第二天(19.05.28):基于RDD的算子--包括各个算子的效率
RDD的概念:不可变,可分区,里面元素可并行计算的集合
	弹性的含义:内存与磁盘;高效容错机制(RDD的依赖关系--血统);Task和Stage的重试机制(Spark框架本身的容错机制);CheckPoint--持久化机制,Persist--进行缓存;数据调度弹性(DAG图);
	
	分区:逻辑概念
	快的原因: 基于内存; 实时缓存; CheckPoint是对缓存的补充
	
	创建RDD:
		已有的RDD; 已有的Scala集合; 从外部文件中读取
	
	action算子:会进行计算,会触发前面的transform算子进行计算,例如reduce
	transformation算子: 不会进行计算--惰性算子,例如map
	算子一般都是高阶函数
	
	传入函数:匿名函数; 定义到Object单例对象里面--静态方法
	
	闭包(driver端;Executor端):代码会分割为两个部分,以及打包到不同的worker上并行计算--打包随处可见
	
	foreach函数
	
	代码的序列化:

分区:
	为什么:并行处理数据集
	怎么分:分区器(hash分区,范围分区)
	
	分区属性:每个分区不跨结点;不跨RDD
	分区数目:代表并行度 --是线程用来计算的
		分区数目计算规则:[2*集群总核数,任务完成时间100ms],一般在100--10000之间
	分片大小源码:textFile-->HadoopRDD-->getPartitions-->getSplits-->FileInputFormat(看视频)

RDD算子:
transformation算子:惰性算子--延迟加载,action触发时才会执行;RDD调用返回RDD--输入输出
Action算子:触发Job ,结果不一定是RDD
Spark编程: 创建RDD-->调用算子计算
	spark-shell:
	map:对集合进行遍历
		sc.parallelize(List("a","b","c","d")) //通过集合创建RDD
		val mappedRes = res0.map("o"+_) //加前缀字母 o
		mappedRes.collect() //返回结果
	filter:数据清洗中过滤数据常用
		mappedRes.filter(_.contains("a")) //含有a的元素
		res3.collect()
	flatmap:
		sc.parallelize(1 to 5, 3) //3为分区数
		res5.flatMap(1 to _)
		res7.collect() //collect之后res7就不是RDD了,可以不加括号
	mapPartitions:map的变种,元素器是个分区迭代器,返回也是一个可迭代的集合,可能会造成OOM,会将整个分区的元素全部加载至内存--效率更高
		res1.mapPartitions(it=>it.map(_+"o"))
	mapPartitionsWithIndex:同时打印分区号
		def func1(index: Int,it: Iterator[String]): Iterator[String] ={it.map(x=>index+":"+x)}
		res1.mapPartitionsWithIndex(func1).collect
	凡是带Partitions的都是对分区的操作;对数据库的链接(对资源的操作)--最好用带Partitions的算子进行处理
		
	Set:交并差,均为对RDD操作
	union:元素合并不去重
		res2 union res12
	intersection:交集且去重
		val s1 = sc.parallelize(List(1,2,3))
		val s2 = sc.parallelize(List(2,3,4))
		s1 intersection s2 
	distinct:去重
	
	sample:采样(是否放回,采样比例,伪随机种子--子集元素个数,每次运行结果一样)--返回抽样得到的子集
		sc.parallelize(1 to 1000)
		res0.sample(true,0.1,101).collect
	
	keyBy:生成key,结果为tuple
		res0.keyBy(_+"K").take(10) //不用collect即可
	
	mapValues:针对Key-Values类型数据对values进行处理
		res5.mapValues(_+100).take(5)
	
	groupByKey:将各个分区相同Key进行分组-->shuffle(最耗资源)--放到同一分区-->value不聚合
		val r1= sc.parallelize(List(("a",1),("b",1),("c",1),("a",1)))
		r1.groupByKey() //不需要传参
		res7.collect
		res8.map((x:(String,Iterable[Int]))=>(x._1,x._2.sum))
	
	reduceByKey:将各个分区相同Key进行局部聚合(本地聚合)--分组(分区)--全局聚合  :效率更高
		r1.reduceByKey(_+_).collect
	foldByKey:相比于reduceByKey多了一个初始化值:初始化仅用于本地聚合阶段--每个分区仅用一次初始值
		r1.partitions.size  //查看分区个数
		val r1= sc.parallelize(List(("a",1),("b",1),("c",1),("a",1)),3) //设置为3个分区
		r1.foldByKey(100)(_+_).collect
		val fun1 = (index:Int,it:Iterator[(String,Int)])=>{it.map(x=>index+":"+x)}
		r1.mapPartitionsWithIndex(fun1).collect
	aggregateByKey:先本地聚合-->再全局聚合 --两个函数分别处理,上述两个底层用的都是该算子
		r1.aggregateByKey(100)(_+_,_+_).collect //第一个参数是初始值,
		r1.aggregateByKey(100)(_+_,math.max(_,_)).collect
		r1.aggregateByKey(100)(math.max(_,_),_+_).collect //本地比较时初始值作为单独的一个值参与
	combineByKey:--三个函数:类型转化--操作的是values(输入类型与最终的输出类型不一致时)-->本地聚合-->全局聚合
		三个参数:类型转换--作用于所有元素;本地聚合--承接于类型转换之后的数据,
		文档例题
		源码:combineByKey-->
		
	四个应用选择:
		局部/全局聚合时需要类型转换时选择combineByKey
		局部/全局聚合时逻辑一致时选择reduceByKey
		reduceByKey和groupByKey的区别:groupByKey本地不聚合,仅仅分组--reduceByKey如何实现
		
	sortByKey:默认升序,分区个数
		
	sortBy:默认升序,分区个数,不一定是tuple类型的数据


​	
各个算子的源码实现		
​		
​		
	聚合操作就是多变一的操作
	ByKey后缀处理的是Key-Value类型/tuple类型的数据,会发生shuffle过程,操作的是values


​	
azkaban: 	

mapValues:职改变map的值

filter:

map:

combineByKey:可以考虑用map改变类型+reduceByKey替代

第三天:各算子详解
1 算子源码实现
	map : 遍历每个元素
	DD/RDDFunction(Pair/Double/SwquenceFile) :算子位置
	clean:发送到worker端执行时无关变量进行清理
	map是对每个分区做的,
	mapPartitions 以分区为计算单位,参数是迭代器,更高效
	MapPartitionsRDD
	
	filter :
	mapValues:是对key-value类型数据处理,只对value做处理
	reduceByKey:key类型不可以是数组类型
		combineByKeyWithClassTag
	aggregateByKey:
	
	join:对两个RDD进行join操作,相同key放到同一个分区,对每个Key下的元素进行笛卡尔积  --底层由cogroup实现
		结果为(key(tuple))
		shuffle运算
	cogroup:第二个参数(即value)是一个tuple(compactBuffer类型的)
	两者结果差别很大;cogroup 可以对三个RDD连接
---	join算子优化方法:????
		sort merge join 
		broadcast join:不进行shuffle过程
		hash join: 

	RDD类型调用的一般是spark上的方法而非是scala方法

所有的算子操作都是基于分区的
action算子:自动触发,无需collect
	reduce:对数据类型无要求
	fold :	初始化值会在本地和全局都使用--本地聚合之后使用初始化值--然后全局聚合之后再使用一次聚合
	aggregate :有初始化值,
	
	collect: //项目中一般不用
	collectAsMap:
	
	count:RDD元素个数
	
	take相关算子会load into到driver端,所以数据量小时可以考虑--慎用
	
	foreach和map的区别:
		算子类别
		map产生一个新的RDD
		
	saveAsTextFile:
	
	各个算子的: 底层实现,应用场景,优缺点
	
	AccessTopN练习题的三种实现方法:常规URL方法/cache方法/自定义分区器方法

第四天:
	action算子和transformation算子的区别:
		1 调用结果不一样:所有的transformation返回RDD,action返回一个结果集而非RDD的抽象集
		2 transformation是惰性的
		3 所有的action都会出发Job进行计算
		4 都是基于RDD抽象集做的操作
	reduceByKey和groupByKey的区别
		1 都会发生shuffle的聚合算子
		2 都是transformation算子
		3 groupByKey仅仅分组,返回为key-compactBuffer
		4 reduceByKey:聚合-分组-聚合,shuffle传输数据较少,效率更高
	
1 如果不发生shuffle,则各个RDD是相互一一对应的 -- pipline  --transformation算子之所以是惰性的就是为了等待形成流水线
	流水线????---批处理读入--Task--shuffle前等待
	shuffle时会先数据本地化再分组
	一个Task(任务--逻辑计算单元)对于一个分区
	每个Executor有多个core,每个core管理一个Task
	HDFS上的block与HadoopRDD不是一一对应的

	rdd.toDebugString  ---查看源码运行
	
	RDD的宽窄依赖:形成血统
		join(k-v类型)是个例外:
			key相同时不发生shuffle(分组--Key的Hash值)--窄依赖(NarrowDependency--(OneToOne/Rang));
			key不同时发生shuffle--宽依赖(ShuffleDependency) --一个分区可能包含不同key的数据(根据分区数来决定)
		有shuffle:宽依赖,形成一个新的stage
		无shuffle:窄依赖
		
		窄依赖获取父分区:原因:获取相应数据
		
		为什么定义依赖:
			1 形成血统--容错机制(RDD的容错/集群的容错(worker/Executor(Application)/Task(线程)))
			2 形成DAG图:依赖关系
			3 任务调度:
				每个action触发一个job,每一个job对应一个DAG图--有join时才会可能出现分支
				action-->DAG--DAGScheduler-->stage-->TaskScheduler(cluster manager--worker)--Task(worker)
				Driver端:Task之前的阶段(RDD-DAG调度--Task调度)
				有可能发生宽依赖--形成新的stage(同时会切断依赖),stage里面不会再次发生shuffle
				task任务形成:每个scheduler形成一个task:任务并行处理/形成pipline
				task由driver发送给Executor(RPC通信)--通过Netty模型

DAG(Stage)划分:
Task生成:
Task在Executor的执行:

源码分析:
	创建一个类对象时类中的所有语句都会被执行
	SparkContext:创建时会初始化DAG调度和Task调度等各种初始化
	DAGScheduler:action的runjob方法追踪;submitJob
	getMissingParentStares(DAG调度中):stage划分逻辑(从最后一个RDD逆推)  P445行
	submitMissingTasks(生成TaskSet,DAG调度中):
		--submitTasks:提交Task --tasks(P1024)-2种task 
		---分区和Task一一对应(一个分区号对应一个Task) 
		--一个Stage对应多个分区
	submitTasks(Task调度):backend(ScheduleBackend)--LocalSchedulerBackend(localEndpoint/reviveOffers--launchTask)--NettyRpcEndpointRef
	launchTask(Executor中):维护一个线程池
	
	分区/stage/Task的相互对应关系

任务划分:
	stage根据分区数(分区号)与Task建立关系	
资源调度方式:FIFO/Fair
RDD的缓存:
	1 什么时候使用:
	2 怎么使用:可以缓存到内存/磁盘
		cache(有默认的缓存级别--12种)
		persist:
		为什么还需要CheckPoint:
			缓存由spark框架管理(worker/Execut);CheckPoint则不受影响
	3 缓存释放: unpersist	
	4 CheckPoint是一个惰性算子,需要触发才会执行
			
				
代码调试:
查看代码:Ctrl+左键 /Ctrl+B
跳回:ctrl+alt+left
源码查看
Debug模式:
	断点-->F8(单步)(鼠标停滞显示;F9是跳至下一断点)-->
	断点设置方法:


spark 源码:
-- Wordcount案例源码分析:
	textFile:调用了HadoopRDD[偏移量,内容] MapPartitionsRDD[内容]  --均在RDD.scala中
	flatmap:调用了MapPartitionsRDD[内容]  --在RDD.scala中
	map:调用了MapPartitionsRDD[内容]   --在RDD.scala中
		Return a new RDD by applying a function to all elements of this RDD
	reduceByKey:ShuffleRDD[内容] --在PairRDDFunctions.scala中     ByKey均会发生shuffle过程
		similarly to a "combiner" in MapReduce
	sortBy:调用了MapPartitionsRDD,ShuffleRDD,MapPartitionsRDD 追根溯源一直到出现RDD为止 
		 this.keyBy[K](f).sortByKey(ascending, numPartitions).values
	collect:runjob--->DAGScheduler.scala中的runJob
-- Wordcount图解:

-- 任务数与分区数
	blocks(HDFS)--InputFormat--InputSplit(不可跨文件)--Task(与分片一一对应)--partition(与Task一一对应)(worker上Executor中的core执行结果)

-- 血缘 : transformation算子形成 每个RDD记录父RDD转换的方法 -->pipline
	
-- spark调优中，增大RDD分区数目，增大任务并行度的做法。 	
	
	
第五天:
spark shuffle 流程:
3种shuffle : 细分为5种
Hashshuffle
	有几个reduce(分区),Task会生成几个结果文件进行对应:会产生大量的中间小文件--根据分区器(默认hash--key的hash值)(分组)
Sortshuffle  
	一个Executor中的
	两种运行机制:为什么划分--可以选择是否排序
		普通运行机制:先sort再写入内存--再写入磁盘--磁盘文件merge(一个Task对应一个文件--建一个索引文件--区分分区)
			多了一个sort和文件合并(外加一个索引文件)
		bypass运行机制:无sort--maptask数据量小于200
	shuffle调优:调整参数
		spark.shuffle.manager:选择shuffle类型
		参数的含义,怎么调节,调节结果
tungsten:

Broadcast Variables:广播变量
	原因(作用):
		--多个Task可以共享此变量,在Executor上存放一份变量,减少数据传输
		--如果两个数据集join操作可能会发生shuffle可以将小的RDD以广播的形式发送到Executor上,避免shuffle
	怎么用:
		--每个Task对其只读,不可修改
		--broadcast方法(参数是集合)--task端:value获取数据
		--广播前使用spark框架进行了预处理,则需要先将数据收集再广播
		--几百M乃至G级别的数据都可以做成广播
	应用场景:
		--需要数据共享时,每个Task需要参考一份公共数据
		--spark优化:数据集广播,避免shuffle
累加器:Accumulators
	定义: 变量;对其他变量累加;在Executor端不可读数据;全局统计;惰性算子--可能会发生重复加(陷阱);action算子则不会
	怎么使用:
		--对Long或者Double,集合类型的数据会使用系统的累加器;
		--在driver端调用add方法,value查看累加结果(driver端)
	自定义累加器:
		--实现抽象类AccumulatorV2
	使用陷阱:
	如何避免陷阱:
		--使用cache
JDBCRDD:将数据库数据取出放入RDD中
	-- 首先项目中新建dir 命名为lib 导入mysql的JAR包
	-- 其次给host授权:
		--use mysql;
		--select host from user;
		--update user set host ='%' where user ='root';
排序:	
	--自定义排序
	
	
概念题整理


编程题整理	
sparkday01 1个
sparkday02 --combineByKey  -sortByKey(了解) 1个半




​	

每日默写:
1，对伴生对象的理解 
2，scala定义类的属性和构造方法需要注意的细节问题

1，standalone模式集群启动流程 
2，任务提交流程 
3，对RDD的认识

1，对RDD的认识；
2 对比action算子和transformation算子
3，比较reduceByKey和GroupByKey


1 对依赖的理解:
	什么时候形成--如何划分--形成血统--容错(RDD容错--分宽窄依赖不同恢复策略)/重试机制
	--DAG图(对应一个Job)--DAG调度(如何实现--逆向切分成stage)--Task调度(作用)
2 stage划分流程(根据源码):
	看着容错图解说
	stage之间是串行的:stage之间是宽依赖划分的;每个stage内部的RDD是并行的
3 什么时候使用RDD缓存:
	1 RDD被反复使用时
	2 shuffle之后产生的RDD--缓存/CheckPoint(切断依赖/持久化到磁盘--可靠性更好,效率略低)
	3 计算逻辑比较复杂时--例如机器学习逻辑,结果RDD进行缓存
	4 RDD依赖链过长时--CheckPoint/缓存
4 CheckPoint和cache的比较

1，stage划分流程 
	shuffer.逆推,finalStage  3个点
2，任务生成的四个阶段 
	--两类Task
	--Task调度:资源调度(FIFO/Fair)
3，spark的缓存机制和使用场景

1,广播变量的使用场景，如何使用，使用注意问题
2,累加器的使用场景，如何使用，使用注意问题
3,shuffle的过程和你对shuffle的理解,hadoopshuffle和sparkshuffle的区别











```scala



import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.rdd.RDD

object Exam {
  def main(args: Array[String]): Unit = {
    //创建一个配置信息对象
    //设置的应用程序名称
    val conf:SparkConf = new SparkConf()
      .setAppName("exam")
      .setMaster("local[*]")

    val sc:SparkContext = new SparkContext(conf)
    
    val lines: RDD[String] =  sc.textFile(args(0))
    //广告投放请求id
        val ads_id = lines.map(x=>{
      val arr = x.split(" ")
      (arr(0),1)
    })

  }
}
```


