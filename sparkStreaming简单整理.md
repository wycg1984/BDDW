# sparkStreaming简单整理

Spark Streaming简单整理:
第一天:sparkday11
是什么:
	基于sparkcore的扩展,高吞吐的可容错的可伸缩的实时流处理平台
	kafka/flume/HDFS --> HDFS/Databases/Dashboards
工作机制(原理):
	实时数据输入流-->分割为数据批次batches(实际上相当于转变为了离线数据-->离线处理方式可用)-->sparkcore批次处理(本质上是一种离线处理)-->批次处理结果
特点:
	毫秒级延时处理:
DStream:
	数据抽象,连续的变化的实时的数据流
	创建方式:外部数据源(实时数据流)/已有DStream生成
	本质上代表的是RDD序列(一个批次对应一个RDD)
	对DStream代表的实时数据流的操作:会转化为对底层RDD的操作(所有RDD操作一致)-->转化为新的DStream的新的RDD
	
股票/售票/交通/电商/
Flink: 
Structure Streaming:

原语(方法):
	Transformations类似于RDD--产生新的DStream:分为有无状态的操作-->无状态(每一个操作(每个RDD/批次)不依赖之前批次的数据);窗口操作
		查看官网: map/flatmap/filter/repartition/union/count/reduce/countByValue/reduceByKey/join/cogroup/transform(参数是DStream的一个个RDD)/updateStateByKey(有状态的)
			
	Output(原语) 可以立即执行:
		print/saveAsTextFiles/saveAsObjectFiles/foreachRDD(结果为当前批次下的每一个RDD--转变为sparkcore下的RDD处理)/

transform:
	将实时数据处理变成RDD类型数据
	可以使用DStream中没有而RDD中有的算子
	
窗口操作:
	自定义计算连续的多个批次的DStream  
	窗口大小时长
	窗口滑动间隔
		
第二天:
Spark Streaming 即将没落	
Structed Streaming  /  Flink 	
updateStateByKey底层源码实现--会发生join操作--耗费性能   维护历史信息--CheckPoint   会拿取全部的历史信息
	val cogroupedRDD = parentRDD.cogroup(prevStateRDD, partitioner)
    val stateRDD = cogroupedRDD.mapPartitions(finalFunc, preservePartitioning)
mapWithState底层源码实现  需要维护历史信息
	有超时机制--定时清理key-Value;无cogroup(join) --更高效
	只拿取当前批次key对应的历史信息
输出结果不一样	
两者都需维护历史信息--会产生大量小文件 --可以使用数据库代替
处理类型都是key-Value类型的

窗口操作:窗口长度为批次时间的整数倍 --有时间多看窗口方法的源码
	具体方法:
		--countByWindow/reduceByKeyAndWindow(原理看源码)/groupByKeyAndWindow...具体看官网 
		--返回值均为DStream  
		--多个批次构成一个窗口--多个批次(一个窗口)一起操作
		--作用在连续的多个批次的逻辑计算
		--一个窗口中的数据对应多个RDD--对应多个批次(每个批次对应一个RDD)--先每批次操作再置入Window再窗口内操作
		--多个RDD将聚合结果置入一个window中,然后window再执行一次相同的聚合操作
	
spark Streaming 和 kafka 的集成:
	kafka0.8版:
		方式:2种
			--Receiver-based -- 已基本放弃使用
				--kafka高级API
				--Executor中的Receiver/Zk自动更新offsets
				--WAL:WriteLog:分时间大小/数据大小接收数据--数据过大时写入磁盘,全部接受之后再消费
			--Direct -No Receiver
			
		两种对比:
			实现机制:
			--直连:SparkStreaming与kafka之间分区(RDD)对分区建立一对一通路,并行消费数据--边读边处理
			--直连:消费数据前Driver先查询offset,自行维护offset值;更高效--无需WAL
			--Receiver: 保证了数据零丢失,但可能会出现数据重复消费(ZK维护offset失败时)
			--直连:实现了exactly-once
		示例:代码复杂 ,读懂
		0.10版
			--消费策略:在检查点重启之后
		代码测试:kafka集群--生产者--程序为消费者--
	
	直连:	
		简化的并行性：不需要创建多个输入Kafka流并将其合并。与此同时directStream，Spark Streaming将创建与使用Kafka分区一样多的RDD分区，这些分区将全部从Kafka并行读取数据。所以在Kafka和RDD分区之间有一对一的映射关系，这更容易理解和调整。

第三天:Redis  
	--NoSQL数据库:菲关系型,分布式--水平拓展
	--键值对类型
CAP理论:选择分布式数据库的依据
特点:
	--存储的数据均是key-value 类型
	--支持五种值类型
	--作为消息系统/缓存系统

使用:
	./redis-cli -h 192.168.91.3 -p 6379     //客户端操作
String类型:
	set strkey strvalue		//增加
	get strkey		//查找
	del strkey		//删除
	set strkey 100
	incr strkey 	//默认每次value值加1
	decr strkey		//默认每次减1
	incrby strkey 100		//数值增加100
	decrby strkey 50
	mset lisi 20 zhangsan 10   //多个键值对--批量操作
	mget lisi zhangsan strkey		//批量查询
	
散列表类型:存储面向对象的属性对象/json解析
	hset obj m1 v1 		//obj是键,后面的值是键值对形式的
	hset obj m2 v2 
	hget obj m1
	hmset obj m3 v3 m4 v4
	hmget obj m1 m2 
	hdel obj		//删除对象
	hdel obj m1 	//删除具体元素
	hset obj m5 100
	hincrby obj m5 50 

list :
	lpush keylist kk jj hh   //左端(链表头)追加--链表实现--利于增删
	rpush keylist 1 2 3 	//右端追加--数组实现--利于查询
	lrange keylist 0 1		//查询[0,1]个元素
	lrange keylist 0 -1 	//查询所有数据
	lpop keylist 		//从头部取一个元素,列表中元素消失
	rpop keylist		//从尾部取一个元素
	lrem keylist 2 3		//删除元素  2 删除2个  3  删除3这个元素
	
set类型:无序去重
	sadd setkey kk jj ll
	smembers setkey 
	sadd setkey kk aa   //只会添加aa,随机位置添加
	srem setkey kk 		//删除元素
	sismember setkey mm 	//不存在返回0 存在返回1
	sadd setkey2 1 2 3 4
	sinter setkey setkey2 	//交集
	sdiff setkey setkey2	//差集 --返回第一个集合的元素
	sunion setkey setkey2	//并集

zset类型:有序集合
	zadd zsetkey 1 1 22 3 5 4   //前面的值排序使用
	zrange zsetkey 0 -1 		//
	内部值不可重复,序号值可以重复
	zrem zsetkey 3 		//删除元素
	
Keys命令:	
	EXPIRE zsetkey 10		//设置生存时间
	TTL zsetkey
	persist zsetkey 		//持久化,取消生存时间设置
	
Redis持久化:
	RDB 数据集快照	--需要触发条件(看配置文件),效率较高,但有可能丢失数据
	AOF 操作命令备份  --默认关闭,基本不会丢失数据,但效率很低--重新执行所有命令
	各自的优缺点:
事务:	
	开始事物--命令入队--执行事务
	multi  //开始
	只要队列中的命令有一个错误队列中所有命令均不执行,回滚
	exec 	//提交

主从复制:
redis集群:
容错:
集群搭建(至少需要6台:3主3从):
	


​		
理解:
1,对DStream的认识 
2，DStream上的API分为哪几类，各自的使用场景是什么	
​	

redis-cluster 搭建
在/apps下新建目录redis-cluster

vi start-all.sh	
cd redis01/bin
./redis-server redis.conf
cd ../..
cd redis02/bin
./redis-server redis.conf
cd ../..
cd redis03/bin
./redis-server redis.conf
cd ../..
cd redis04/bin
./redis-server redis.conf
cd ../..
cd redis05/bin
./redis-server redis.conf
cd ../..
cd redis06/bin
./redis-server redis.conf
cd ../..		

启动集群前需要先将每个redis下的aof和dump/rdb文件删除		
		
将redis-3.0.0中的src下的.rb复制到一个空的redis0707目录下,然后				
./redis-trib.rb create --replicas 1 192.168.91.3:7001 192.168.91.3:7002 192.168.91.3:7003 192.168.91.3:7004 192.168.91.3:7005 192.168.91.3:7006

/apps/redis-cluster/redis02/bin/redis-cli -p 7002 -c

出现错误in `call': ERR Slot 5416 is already busy (Redis::CommandError)时解决方法:		
连接:http://www.kyjszj.com/wdzl/142.html
slushall 
cluster reset		
		
增加主节点:
./redis-trib.rb add-node 192.168.91.3:7007 192.168.91.3:7001	
		
		