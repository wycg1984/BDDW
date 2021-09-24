# scala简单整理

Scala:
面向对象:对象和类是基础单元
函数式编程:函数(要求是纯函数--每个输入都有一个输出,中途没有输出)是基础单元
静态类型--编译时确定
(Scala中可以直接使用java中的包和类与方法)运行在JVM中

spark源码是由Scala实现的

变量语法:
var和val的区别:数值类型中;引用类型中:值可变,地址不可变;关键词lazy:对val使用懒惰模式:使用时才赋值;不允许修饰var变量
val会尽早尽快的被垃圾回收;可读性高;官方推荐
多变量赋值:偶尔使用

数据类型:
看官网
Unit类型是独有的;option类型是独有的;
所有的数据类型都是类类型(类似于包装类),所有的值都是一个对象
浮点型默认为double类型,float需要在后面加f;浮点计数法  不能精确表示--离散的
字符类型单引号单字符;
Any可以赋所有类型的值;
Unit = ()

类型转换:
看官网
自动:赋值时;计算时;
强制:调用方法.to...

操作符:
逻辑操作符:&& || !
位操作符:& | ~ ^ 按二进制逐位 
注意点:
	所有操作符均为方法的重载(每个符号都是一个方法;所有的数值都是一个对象):1+1  --> 1 .+(1)
	没有 ++ -- 操作符: a +=1
	
表达式:
以表达式为中心
有返回值--类似于方法/函数--最后一个表达式即返回值;不使用return
	条件表达式:if...else  打印print和赋值的返回值为()
	混合类型条件表达式:返回值类型不同
	代码块中多条语句用;分隔,注意返回值
	方法/函数就是代码块表达式
	
循环:
for推导

方法和函数:	返回值类型可以省略
方法语法:def
函数定义:=> 用_+_表示两变量相加  但不能超过两个下划线

集合:
看官网
可变集合与不可变集合
数组:定长;变长
高阶数组--矩阵--多位数组  -->数组套数组
定长数组是可变集合

方法/函数:
可以使用return ,但会使自动类型推断失效

集合的高阶方法,例如 map,fliter等知道其底层实现原理

map:
看官网API
map返回的是一个集合

tuple:
自动支持最长22个元素
拉链以及拉链处理操作

list:有序(按追加顺序);
不能用map遍历输出


set:无须(非追加顺序),去重
hashSet是不可变的


多线程(高并发):考虑线程是否安全

高并发下使用线程非安全:加锁;集合局部化;


方法总结:
数组:
1. 定义:
	val arr = Array(10, 50, 20, 30)	
	val arr2 = new Array[Int](10) //自动用0初始化
	val arr3 = ArrayBuffer[String]()	//可变
    val arr4 = new ArrayBuffer[Double]()	//可变
2. 访问
	arr(1) = 8  //下标即可
3. 遍历:
	for (elem <- arr2)
	for (i <- 0 until arr.length)
4. 数组基本方法--7个方法
	arr.sum	arr.max	arr.min	
	arr.sorted.toBuffer //升序排序
	arr.sortWith(_ > _).toBuffer  //升序排序
	arr.sortWith((x: Int, y: Int) => x > y) //升序排序
	arr.sorted.reverse.toBuffer //降序
5. 追加元素--5个方法
	arr3 += "java"
    arr3 += ("scala", "python")
	arr3 ++= Array("c", "C++")
	arr3.append("PHP")
	arr3.insert(2, "R") //第几个元素之前
6. 删除元素--3个方法
	arr3.trimEnd(1) //从尾部删除几个元素
	arr3.trimStart(1) //从首部删除几个元素
	arr3.remove(2, 1) //从第几个开始删除,删除几个
	toBuffer toArray //定长数组与变长数组的转换
7. 数组构建--for推导/map映射/filter过滤
	val arr6 = Array(1, 2, 3, 5, 8, 7)
    for (elem <- arr6) yield elem + 10 //for推导
    val res = arr6.map((x:Int)=>x+10)	//map映射
	val res1 = arr6.filter((x:Int)=>x%4==0) //filter过滤
8. 高阶数组/矩阵/多维数组--数组嵌套
	val matrix = Array.ofDim[Int](2,2)
    for (i <- 0 to 1;j <-0 to 1)
      matrix(i)(j)=i*j
    println(matrix(0).toBuffer)
9. 定义不规则多维数组
	val multiArr = new Array[Array[Int]](2)
    multiArr(0) = new Array[Int](2)
    multiArr(1) = new Array[Int](5)
	

Map映射
1. 定义/构建
	val map1 = Map("tttm"->20,"jey"->18)	//键值对形式
    val map2 = Map(("tom",20),("jerry",18)) //元组形式
	val map3 = Map(("tom",20),("jerry",18)) //元组形式(可变集合时)
2. 访问 --3个方法
	map2.contains("tom")  map2("tom") //先判断再打印,不会报异常
	map2.getOrElse("tom1",0) //此方法最好,实际上是通过get方法实现的
	map2.get("tom").get //若键存在返回some,不存在返回None
3. 修改:添加(添加顺序随机,无字典顺序)/删除 -- 6个方法
	map3("tom") = 25 //修改,键存在时;添加,键不存在时
	map3.update("kate",19) //添加/更新--看键是否存在
	map3 +=("dim"->21) //可以是元组,可以是键值对
    map3 ++= map1
    map3 -= ("dim") //只需键即可
    map3 --= Array("kate","tom")
4. 遍历
	for(elem <- map3)
	for ((k,v) <- map3)
	for (k <-map3.keySet) println(k+"="+map3(k))
5. HashMap 元素个数大于等于5时 使用的才是HashMap
	var hashMap = HashMap("tom"->20,"jry"->18) //定义
	hashMap.getClass  //获取类型
	hashMap.getOrElse("jry",0)  //访问
	hashMap.put("tom1",22)  //添加
	hashMap += (("tom2",23)) //注意是两个括号,键值对是用括号括起来的
    hashMap = hashMap.drop(1) //删除几个元素
    hashMap.foreach(println) //遍历输出
    hashMap.foreach((x:(String,Int))=>println(x)) //元组形式
6. treemap 有序--按键大小排序,只有不可变的集合
	其余操作参照map及HashMap
	

List列表
1. 定义/构造
	val list1 = List("a","bb","ccc")
	val empty = Nil	//空列表
    val list2 = List()	//空列表
	val list3 = "a"::"bb"::"ccc"::Nil	//::右结合--从右向左计算
    val list4 = ("a"::("bb"::("ccc"::Nil)))
2. 访问
	list15(0)	下标即可
3. 遍历
	for (elem <- list15)
	list15.iterator.foreach(println) //遍历输出
4. 元素添加
	list15 = "e1"::list15 //头部追加
    list15 = "e2" +: list15 //头部追加
	list15 :+ list15 +"e3" //左结合 尾部追加
5. 删除
	list15.drop(2) //删除个数
6. 基本操作
	list15.isEmpty 
	list15.head //只返回一个元素
	ist15.tail //是一个集合--除去第一个元素之外的
	list15.sorted //默认升序
	list15.sorted.reverse //降序
    list15.size //长度,元素个数
7. 列表的拆分和合并
	val List(a,b,c)=List("aa","bb","cc")
	list15.splitAt(3) //在哪一位进行切分 前三位|后面
	println(rest._1.foreach(println)) //最后会打印小括号出来:因为内部println;拆分后的第一部分
	list15.take(2) //取前两位
	list15.drop(2) //删除前两位
	list15.dropWhile(_.startsWith("aa")) //注意与filter的区别
8. 列表内容的操作(求和) --折叠函数 fold reduce foldLeft foldRight reduceLeft reduceRight
	var list6 = List(10,20,30,4,5,6,7)
	list6.map(_+10) -- list6.map((x:Int)=>x+10)  //map实现了对元素的遍历
	list6.fold(0)(_+_) //list6所有元素相加之和,0+和 由foldleft累加实现
	list6.fold(0)((x:Int,y:Int)=>x+y)
	list6.reduce(_+_) //同fold
9. 压平 --前提 集合里面的元素依然是一个集合,内层集合中的元素提取出来--即参数也应为集合
	相当于flattern+map,先map后flatten
	list8.flatMap(_+"nn")  list10.flatMap(_.reverse)  //括号内应为集合类型
	list10.flatten
	//flatten只对集合有效,压平--压成一层--map之后生成的是集合
	val list11 = List(1,2,3,4)
    list11.flatMap((x:Int)=>List(x))
	
	
	
	

aggregate:最灵活,使用场景少,聚合使用(局部和全局)--多线程, 功能和fold与reduce一样--但fold与reduce要求局部与全局聚合的逻辑一致才行
	可进行类型转换,
	
	
面向对象:特别灵活,变化很多
	自定义get/set方法
	构造方法
	
	特质trait:类似于java8的接口   --补课
		属性(抽象--只有字段名和类型--未初始化)
		方法(抽象--有无实现均可)
		与类的区别: 不能实例化
		接口和抽象类的区别:
			接口多个实现;抽象类--继承时,减少代码重复
		
	scala传名调用
	
	抽象类:与接口的区别
	
	内部类:什么时候需要内部类:一般只有当前的外部类有关系,不太建议
		通过内部类对象/通过外部类 访问
		匿名子类:了解--只调用一次

对象:
	单例对象(独立对象)
		实现类似java的静态对象/静态方法的功能 没有构造方法,有属性和方法
	
	伴生对象,是一个单例对象   --重点 拓展
		均在伴生对象中定义
			apply 生成器--用来生成一个对象 ,生成器
			unapply提取器:模式匹配时,提取属性 

App 入口
继承 override super 
重写抽象方法override可省略
isintanceOf类型判断
asinstanceOf类型转换
样例类--模式匹配,
	属性只能访问不可修改val
	系统自动条用apply,unapply,

模式匹配:重点
	
面向对象详解:
基本概念:
类
1. 基本概念
	并不用声明为public;
	如果没有定义构造器，类会有一个默认的无参构造器 ;
	var修饰的变量，对外提供getter setter方法 ;
	val修饰的变脸，提供getter方法，没有setter方法
	
	用var修饰的变量，默认同时有公开的getter方法和setter方法;
	声明一个属性,必须显示的初始化，然后根据初始化数据的类型自动推断，属性类型可以省略;
	_表示一个占位符，编译器会根据你变量的具体类型赋予相应的初始值,使用占位符，变量类型必须指定
		 //类型  _ 对应的值  
		 //Byte Short Int Long默认值  0  
		 //Float Double    默认值 0.0  
		 //String 和 引用类型   默认值 null  
		 //Boolean     默认值 false
	如果赋值为null,则一定要加类型，因为不加类型, 那么该属性的类型就是Null类型.
	用val修饰的变量是只读属性的，只带getter方法但没有setter方法;相当与Java中用final修饰的变量;val修饰的变量不能使用占位符
	类私有字段,有私有的getter方法和setter方法，只能在类的内部使用;其伴生对象也可以访问
	对象私有字段,访问权限更加严格的，类的方法只能访问到当前对象的字段 
2. 自定义get/set方法
	将成员变量定义为私有 
	字段属性名以“_”作为前缀，如定义：_x 
	getter方法定义为：def x = _x 
	setter方法定义时，方法名为属性名去掉前缀，并加上后缀，后缀是：“x_=”
3. 自定义类的构造方法
	作用是完成对新对象的初始化，构造器没有返回值
	class 类名(形参列表) //主构造器
	def this(形参列表) ///辅助构造器 函数的名称this, 可以有多个，编译器通过不同参数来区分.
	
	
	

Map匹配未实现	
密封类
Option类型
偏函数:源码较多,自我使用较少--与普通函数的区别:方法体case语句--模式匹配;是一个对象;只有一个参数;有一个返回值;偏函数只处理case后面的相关对象
偏应用函数
字符串插值 s/f/raw

文件读写
正则表达式
异常:final--文件关闭等	
	
高阶函数:参数/返回值是函数
闭包:一个函数引用了函数外的变量,变量和函数形成一个闭包
柯里化:分步传多个参--
隐式转换: 隐式转换方法/类--现在本类中找方法,没有时再去隐式类中查找--compare排序  
泛型:源码有,平时接触不到  协变/上界/上下文界定

协变?
	
嵌套??	 柯里化?? 
	
	
	
	未完待续......


​	
​	
高阶函数:	

并发:一段时间内同时执行
并行:同时执行,多核CPU	
共享资源加锁--死锁
Actor--相当于一个线程
同步:
异步:
	
	
	
	
苏宁600-800G :15台左右  云服务器
管理/架构/算法
框架就那几个,几年就搞定了
5-8年:3--5W -- 瓶颈
先想思路在下笔
时间是有限的--多线程工作
伪代码解释代码
回答前先思考几秒
15.5K   --18K以上
几天后再说/有什么问题还要问吗
给你纠正问题--有一定希望--可以说你现在手中有2个offer,需要在某某天给出答复 --- 



规律/惜时/自律/习惯
当你老了  追梦赤子心  平凡之路  纸短情长  等你下课

计算机这么多眼花缭乱的编程语言/中间件/软件 起源都是什么??

学术志 


