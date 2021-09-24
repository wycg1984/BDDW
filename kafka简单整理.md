# kafka简单整理

## 第一天:sparkday09---test0605

知识点:
	--kafka是什么(3点)-->消息系统(是什么,应用场景,工作模式)-->jms规范(了解)
	--kafka架构
	--kafka核心组件
		--producer(如何发送),
		--consumer(如何消费),
		--consumer group(消费规则),
		--broker(leader,follow,选举,负载均衡,分区怎么分配),
		--topic(是什么,分区策略/命名,segment与分区的对应关系,segment命名规则,根据offset查找对应消息) ,
		--partition(分区规则(2种),分区中的文件如何滚动),
		--offset(写数据时offset怎么增长,根据offset消费数据))
		--segment(数据怎么保存到segment中--数据的存储机制)
	--数据写入/消费流程,不同的确认机制,存储策略
	
	
	
是什么:
	--分布式消息队列--读:发布  写:订阅
	--缓存(存储)/处理数据
消息系统:
	--什么是:
		实际上就相当于是个中间件--没有什么是中间件解决不了的--如果有那就在加中间件
	--为什么需要(使用场景):
		解耦(最重要):方便模块复用,
		冗余:对数据进行持久化,避免数据丢失---插入-获取-删除范式
		扩展性:
		峰值处理能力:将突增数据缓存--削峰
		可恢复性:因为解耦,
		顺序保证:FIFO
		缓冲: 是生产者和消费者处理速度一致
		异步通信:
				
生产者:发布消息--发布到队列中--指定topic类别
消费者:订阅消息--接受队列中的数据--指定topic类别
消息分类:每个消息都有一个主题/类别(topic)
消费模式:
	--点对点:一对一
	--发布/订阅:一对多
		--推模式--生产者控制速度--数据量过大时易造成消费者压力--消费者根据类别消费消息
		--拉模式:消费者主动拉取消息--按照自身处理能力
		--kafka采用的是拉模式
kafka架构:
	producers:产生数据
	brokers(kafka):
		--每个broker上有多个分区
		--按分区存储数据--一个分区类似于一个队列/容器--一个分区对应一个topic(多对一)
		--每个分区多副本机制,分区内有lead和follow
		--lead:负责读写,一个lead对于多个follow--lead宕掉之后,ZK在follows选举出新的lead
		--follow:只负责数据的备份
	consumers:使用数据
	Zookeeper:
	

	消费者组:以组别的形式订阅相应的topic
	通过增加brokers增大吞吐量

核心组件:
	--producers:生产者:指定消息的类别信息(topic)和分区里的leader交互(写)
	--consumer:消费者--消费者组
		--以组别的形式订阅相应的topic
		--一个消费者如何消费数据:每条消息只能被消费者组中的一个消费者消费,但可以被多个消费者组消费
		--广播机制:只要每个consumer有一个独 立的CG就可以了--发给所有的consumer
		--单播机制:只要所有的consumer在同一个CG--发给任意一个consumer
	broker:：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个top
		--topic:物理上不同topic分开存储,逻辑上一个topic的消息虽然保存于一个/多个broker上;可以理解为一个队列，消息根据Topic进行归类。 
		--partition:对应一个文件夹,每个topic对应一个/多个partition--包含数据和索引文件
		--replica:副本
		--leader::replica中的一个角色,负责读写
		--follow:从leader中复制数据
		--offset:偏移量,每条数据都有一个offset,offset做消息的文件名字:方便查找;消费者组中的消费者消费之后该offset会增大,原offset消失--保证组内消费者只能消费一次
	
集群安装:首先查看Spark Streaming版本	
kafka :scala版本号-kafka版本号
	
kafka基本操作:	
	查看topic:  ./kafka-topics.sh --zookeeper mini1:2181 --list
	创建topic:	./kafka-topics.sh --zookeeper mini1:2181 --create --replication-factor 3 --partitions 2 --topic test1
	启动生产者: ./kafka-console-producer.sh --broker-list mini1:9092 --topic test1
	启动消费者: ./kafka-console-consumer.sh --zookeeper mini1:2181 --topic test1
	启动消费者: ./kafka-console-consumer.sh --zookeeper mini1:2181 --from-beginning --topic test1
	数据存放位置: /data/kafka/kafka-logs  几个分区里面就对应几个文件夹(名字为topic名)  文件夹里包含索引文件和日志文件
	查看topic基本信息: ./kafka-topics.sh --describe --zookeeper mini1:2181
kafka基本配置信息:
	broker.id
	log.dirs
	port
	zookeeper.connect
	message.max.bytes
	num.partitions:
	log.segment.bytes/log.roll.hours
	log.retention.hours  :默认数据文件存储一周
	delete.topic.enbale 
	log.retention.bytes 
Producer配置信息:
	seriallzer.class
	message.send.max.
Consumer配置信息:
	consume.timeout.ms

kafka文件存储机制:
	1. topic中partition存储分布 
	2. partiton中文件存储方式 
	3. partiton中segment文件存储结构 
	4. 在partition中如何通过oﬀset查找message 
		
topic中partition的存储分布:
	同一个topic下有多个不同partition，每个partition为一个目录，partiton命名规则为topic名称+有序序号，第一个partiton序号从0开始，序号最大值为partitions数量减1

partition中文件存储方式:
	--partition命名规则: 为topic名 称+有序序号，第一个partiton序号从0开始，序号最大值为partitions数量减1
	创建topic时指定partition个数,生成相应的文件夹
	每个文件夹下平均生成大小相等的segment数据文件,每个segment file消息数量不一定相等--方便旧segment file快速被删除
	每个partition支持顺序读写,segment文件的生命周期(创建和删除)由服务器配置参数决定	
	对于消费者而言:
		--分区内有序(offset递增),分区无序
Segment(字段):
	--命名 :partion全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件 最后一条消息的oﬀset值。数值最大为64位long大小，19位数字字符长度，没有数字用0填充
	--数据文件log:存储大量消息 ; 索引文件index :存储大量元数据 索引文件中元数据指向对应数据文件中message的物理偏移地 址
	当前log文件里记录所有消息的最小的offset值-1
	segment data ﬁle由许多message组成
kafka如何查找消息:
	--元数据记录的是每一个组对应的每一个分区消费的offset
	--查找segment file (根据消息的offset值与segment文件名比较来确定)
		--以起始偏移量命名并排序这些文件，只要根据oﬀset 二分查找文件列表，就可以快速定位到具体文件。
	--通过segment ﬁle查找message :依次定位到index的元数据物理位置和log的物理偏移地址,然后再通过log顺序查找直到等于oﬀset为止

读写消息的流程:
	--发送消息:
		--只要涉及到存储一般都会序列化
		--ack值=0(leader写入成功即可),ack=1(leader和follow都写入成功),ack=-1(无确认机制)
	--写消息:
		--确认机制:
		--producer收到ack后的处理:
			--at least once:只要没有返确认机制/写入失败-->消息重发
			--at most once:每条消息只发一次-->无论成功与否
			--exactly once:保证每条发送一次且仅发送一次,而且是写入成功(事物机制/幂等机制...)
分区机制:轮训机制(key为null时)/hash分区	
	
	
	

## 第二天:

高效存储--高吞吐:顺序读写,PageCache,sendFile
3.33.14及之后
如何消费已经消费过的数据:
1 消费数据消费过程,offset,修改之后,只是修改了当前组对应当前分区的offset值
2 一条消息只能被消费者组里的一个消费者消费
3 数据存储在kafka磁盘,默认7天
解决方案:
1 通过镜像kafka集群,从镜像集群去消费数据
2 修改offset值(自己实现)
3 修改其他的消费者组的消费者去消费数据

分区与消费者的关系:
每个分区内的数据只能被组内的一个消费者消费

同步副本:
	--ISR的副本保持与leader的同步---防止leader宕机的数据丢失	
	--HW:一个分区的最后一条消息的offset,leader将HW持续发送给slave,broker将其写入磁盘以便将来恢复
	
生存周期:

ZK管理kafka:首先确定ZK版本(了解)

kafka常见问题:记忆

kafka保证一致性和可靠性(消费方面)	:
	--HW--一致性
	--确认机制acks--可靠性 0无确认 1leader确认 -1leader与follow确认 
	
在高并发下避免消息丢失和消息重复:
	丢失:发生情景;--无确认机制时且producer无重试机制时;ISR内副本均已用尽
		--acks=all;--确认机制 --给与消息唯一标识符
	重复:
	
幂等原理:消息相同时只处理一条

消费一次且仅消费一次:
	--幂等机制:保证kafka数据不重复
	--事物:保证操作的原子性
	--流处理EOS
	
低阶API:难度大,自由度大,可自己调用	
	
	
	


​	
理解:
一条数据如何写入到分区里面:
​	--指定topic类别-->kafka管理元数据(指定topic分入几个分区)-->确定分区-->找到leader-->一个对应一个文件夹-->最终写入该文件夹下的某文件中
一条数据如何被消费:
​	--先加入某个消费者组(不知道时会加入默认消费者)--(只能消费该消费者组对应的topic信息)-->找到该topic对应的分区内的leader-->读取出数据
如何实现广播机制:
如何实现单播机制:所有消费者放到同一个消费者组中


1, 对producer consumer的认识  
2, kafka文件存储机制 

1，分区和消费者的关系 
2，如何消费已经消费过的消息数据 
3，kafka如何保证一致性和可靠性

第三周总结:
常见问题必考