# Hive实战技能--必知必会

HQL

![image-20210726121930233](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210726121930233.png)

Hive图形化连接工具

## 1.Hive的库与表

![image-20210726123454849](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210726123454849.png)

```sql
--hive里面有哪些数据库
show databases;
--创建数据库
create database tszn_01;
--使用数据库
use tszn_01;

--到hadoop的文件系统进行查看 http://locolhost:50070

--创建数据表
create table person
(
	id int,
	name string,
	age int 
);

--插入数据
insert into table person values
(1,'max',25),
(2,'mat',23),
(3,'leo',24),
(4,'ting',28),
(5,'liang',32);

--查询数据
select * from person;

--删除数据库
drop database tszn_01;
--hive中需要先删除表，才能删除库

--强制删除数据库
Drop database if exists tszn_01 cascade;
```

**外部表与内部表**

```sql
--建立外部表
create external table person_wai
(
	id int,
	name string,
	age int
);
--插入外部表数据
insert into table person_wai values
(1,'max',25),
(2,'mat',23),
(3,'leo',24),
(4,'ting',28),
(5,'liang',32);

--建立内部表
create external table person_nei
(
	id int,
	name string,
	age int
);
--插入内部表数据
insert into table person_nei values
(1,'max',25),
(2,'mat',23),
(3,'leo',24),
(4,'ting',28),
(5,'liang',32);

--删除内部表 
drop table person_nei
--删除外部表 
drop table person_wai
```

删除后：
内部表的元数据和表数据都被删除了；
外部表仅删除了元数据，表的数据还保存在目录下

外部表使用场景：
原始日志文件或者同时被多个部门同时操作的数据集，就需要使用外部表；
如果不小心将meta data删除了，HDFS上的data还在可以恢复，增加了数据的安全性。



insert数据后产生临时表
使用insert会产生临时表，重新连接后临时表会消失。
所以，如果要插入大批量的数据，不建议用insert。



Hive数据库路径下有多个文件
/apps/hive/warehouse  这个是hive的default库



**查询建表和加载数据**
use tszn_01;
--查询建表(有数据)
create table person2
as
select * from person;
--查询建表(无数据)
create table person3 like person;



**加载数据**

```sql
--加载数据到表
insert into table person2 values
(12,'lily',21),
(13,'lila',23);

insert into person3 
	select * from person;

--文本数据使用load data
--从数据库加载到hive使用sqoop

--加载数据到表
create table if not exists person4
(
	id int comment 'ID number',
	name string comment 'people name',
	age int comment 'people age'
)
comment 'write by leo'
row format delimited --指定序列化和反序列化的规则
fields terminated by ','
--字段用逗号分隔
lines terminated by '\n'
--每条数据行分隔
--file format(HDFS文件存放的格式，默认TEXTFILE,即文本格式，可以直接打开)
--location 'input/wc' 指定表在HDFS上的位置 经常用在创建外部表;

load data local inpath 'opt/person4' into table person4;
```



## 2.Hive的特殊数据类型

**特殊数据类型array**
数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从0开始。

```sql
use tszn_01;

create table s1(
id int,
name string,
likes array<string>
)
row format delimited   --指定序列化和反序列化的规则：
FIELDS TERMINATED BY ','   --字段用逗号分隔
COLLECTION ITEMS TERMINATED BY '-'  --集合项用‘-’分隔
lines terminated by '\n';    --每条数据行分隔

load data local inpath '/opt/s1' into table s1;

select * from s1;

select likes[0] from s1;

select * from s1 where likes[0] = '足球';

--explode将数组拆分成行，lateral view来实现聚合
--id  name links
--1   张三  ["足球","骑行","读书","喝茶"]
select hobby,count(*) as c1 from s1 lateral view explode(likes) likes as hobby 
group by hobby order by c1 desc;  
--hobby c1
--读书   2
--喝茶   1
select id,name,hobby from s1  lateral view explode(likes) likes as hobby;
--将原始的两行数据变成多行
--id name hobby
--1   张三 足球
--1   张三 骑行
--1   张三 读书
--1   张三 喝茶
```

**特殊数据类型map**

map是一组键值对元素组成的集合，使用数组表示，可以访问元素。

```sql
use tszn_01;

create table s2(
id int,
name string,
deductons map<string,string>
)
row format delimited   --指定序列化和反序列化的规则：
FIELDS TERMINATED BY ','   --字段用逗号分隔
COLLECTION ITEMS TERMINATED BY '-'  --集合项用‘-’分隔
MAP KEYS TERMINATED BY ':'   --map形式是key:value
lines terminated by '\n';    --每条数据行分隔


load data local inpath '/opt/s2' into table s2;

select * from s2;

select *  from s2 where deductons['英语'] = '四级';

select deductons['日语'] from s2;

select explode(deductons) as (note_name,note_level) from s2;  

select id,name,note_name,note_level from s2 lateral view explode(deductons) deductons as note_name,note_level;   
```

**特殊数据类型Struct**

Struce和C语言的struce类似，都可以通过点符号访问元素内容
针对一组数据中有不同的数据类型，是升级版的map。

```sql
use tszn_01;
create table s3(
id int,
address struct<city:string,province:string,area:string,code:int>
)
row format delimited   
FIELDS TERMINATED BY ','   --字段用逗号分隔
COLLECTION ITEMS TERMINATED BY '-'  --集合项用‘-’分隔
MAP KEYS TERMINATED BY ':'   --map形式是key:value
lines terminated by '\n';    --每条数据行分隔

load data local inpath '/opt/s3' into table s3;

select * from s3;

select address.city from s3;

select * from s3 where address.city = '大连';

select address.city,count(1) from s3 group by address.city;
```

**建立包含特殊数据类型的表**

```sql
--建立包含特殊数据类型的表
drop table students

create table students(
sno int,
sname string,
sage int,
slikes array<string>,
snote map<string,string>,
sscore struct<Chinese:int,English:int,Math:int>
)
row format delimited   
FIELDS TERMINATED BY ','   --字段用逗号分隔
COLLECTION ITEMS TERMINATED BY '-'  --数组项中用‘-’分隔
MAP KEYS TERMINATED BY ':'   --map形式是key:value
lines terminated by '\n';    --每条数据行分隔


load data local inpath '/opt/data' into table students;

insert into table students
    values(11,'leo',25,'足球'-'游泳','英语':'四级'-'日语':'二级',90-100-80) ;
    
select * from students;

select sum(sscore) from students;

select sscore.math+sscore.english+sscore.chinese from students;

select avg(sscore.Math) from students;
```



## 3.Hive必会技术-分区

**为什么要分区**
在Hive Select查询中一般会扫描整个表内容，会消耗很多的时间做没必要的工作。有时候只需要扫描表中关心的一部分数据，因此建表时引入了partition概念。分区表指的是在创建表的时候指定的partition的分区空间。
Hive可以对数据按照某列或者某些列进行分区管理，所谓分区我们可以拿下面例子进行解释。
当前互联网应用每天都有存储大量的日志文件，几G、几十G甚至更大都是有可能。存储日志，其中必然有个属性是日志产生的日期，在产生分区时，就可以按照日志产生的日期列进行划分。把每一天的日志当作一个分区。将数据组织成分区，主要可以提高数据查询的速度。至于是用户存储的每一条记录到底放在哪个分区，由用户决定。即用户在加载数据的时候必须显示的指定该部分数据放在哪个分区。

```sql
--创建分区表
create database tszn;
use tszn;

create table students(
sno int,
sname string,
sage int
)
partitioned by (sclass string)
row format delimited   
FIELDS TERMINATED BY ','   --字段用逗号分隔
lines terminated by '\n';    --每条数据行分隔

--加载数据
load data local inpath '/opt/data' into table students partition (sclass='12001');
load data local inpath '/opt/data1' into table students partition (sclass='12002');



--添加分区
alter table students add partition (sclass = '12003');

--删除分区 数据也会删掉
alter table students drop partition (sclass = '12002');


--查询分区表没有加分区过滤，会禁止提交这个任务。
set hive.mapred.mode = strict;
set hive.mapred.mode = nostrict;

select * from students 

--查看分区的键及属性
describe extended students; 
```

**多重分区技术**

```sql
drop table log_m;
use tszn;

--创建多重分区表
create table log_m(
id int,
name string,
age int
)
partitioned by (year1 string,month1 string,day1 string)
row format delimited
FIELDS TERMINATED BY ',' 
COLLECTION ITEMS TERMINATED BY '-'  
MAP KEYS TERMINATED BY ':'  
lines terminated by '\n';


--插入数据
insert into table log_m  partition (year1 ='2012',month1 ='12',day1 ='21')
values(1,'la',12); 

create table person(
id int,
name string,
age int
)
row format delimited
FIELDS TERMINATED BY ',' 
COLLECTION ITEMS TERMINATED BY '-'  
MAP KEYS TERMINATED BY ':'  
lines terminated by '\n';
insert into table person values(2,'leo',15);

insert into table log_m  partition (year1 ='2012',month1 ='12',day1 ='1')
select id,name,age from person; 

--新增多重分区

alter table log_m add partition(year1 ='2010',month1 = '12',day1 =' 21')
location 'hdfs://t10-002.hadoop:8020/apps/hive/warehouse/tszn.db/log_m/';


alter table log_m add partition(year1 ='2013',month1 = '12',day1 = '21');



--删除多重分区
alter table log_m drop partition (month1 = '12');


--修改字段
alter table log_m change column age sage string
comment'修改列名字和属性,放在id后边'
after id;

--修改表的存储属性
alter table log_m 
set fileformat sequencefile;
```

**动态分区技术**

```sql
drop table dy_table;

create table dy_table(
id int,
name string,
age int
)
partitioned by (year1 string,month1 string,day1 string)
row format delimited
FIELDS TERMINATED BY ',' 
COLLECTION ITEMS TERMINATED BY '-'  
MAP KEYS TERMINATED BY ':'  
lines terminated by '\n';


--开启动态分区
set hive.exec.dynamic.partition=true;  

--动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，nonstrict模式表示允许所有的分区字段都可以使用动态分区。
set hive.exec.dynamic.partition.mode=nonstrict;  

--默认值：100
--在每个执行MR的节点上，最大可以创建多少个动态分区。
--该参数需要根据实际的数据来设定。
--比如：源数据中包含了一年的数据，即day字段有365个值，那么该参数就需要设置成大于365，如果使用默认值100，则会报错。
SET hive.exec.max.dynamic.partitions.pernode = 1000;

--默认值：1000
--在所有执行MR的节点上，最大一共可以创建多少个动态分区。
--同上参数解释。
SET hive.exec.max.dynamic.partitions=1000;


--默认值：100000
--整个MR Job中，最大可以创建多少个HDFS文件。
--一般默认值足够了，除非你的数据量非常大，需要创建的文件数大于100000，可根据实际情况加以调整。
hive.exec.max.created.files

--默认值：false
--当有空分区生成时，是否抛出异常。
--一般不需要设置。
hive.error.on.empty.partition

--插入后动态分区
insert into table dy_table partition (year1,month1,day1) 
select * from test1;
--或
insert into table dy_table partition (year1,month1,day1) 
values(1,'leo',25,'2016','10','10'),
(2,'max',26,'2018','02','01');


--查询分区数据
select * from dy_table where year1 = '2018';


--清空分区数据
alter table dy_table drop partition (year1='2018');

alter table dy_table drop partition (month1='10');

truncate table dy_table partition(month1 = '10');
```



## 4.Hive必会技术-索引

**索引的作用**

Hive支持索引，但是Hive的索引与关系型数据库中的索引并不相同。
Hive索引可以建立在表中的某些列上，以提升一些操作的效率，以减少MapReduce任务中需要读取的数据块的数量。
在可以预见到分区数据非常庞大的情况下，索引常常是优于分区的。
虽然Hive并不像事物数据库那样针对个别的行来执行查询、更新、删除等操作。
它更多的用在多任务节点的场景下，快速地全表扫描大规模数据。但是在某些场景下，建立索引还是可以提高Hive表指定列的查询速度。
索引适用的场景:
适用于不更新的静态字段。以免总是重建索引数据。每次建立、更新数据后，都要重建索引以构建索引表。
**Hive索引的机制如下：**
hive在指定列上建立索引，会产生一张索引表（Hive的一张物理表），里面的字段包括，索引列的值、该值对应的HDFS文件路径、该值在文件中的偏移量;
0.8后引入bitmap索引处理器，这个处理器适用于排重后，值较少的列（例如，某字段的取值只可能是几个枚举值）
因为索引是用空间换时间，索引列的取值过多会导致建立bitmap索引表过大。
但是，很少遇到hive用索引的。说明还是有缺陷or不合适的地方的。

```sql
use default;

--建立索引   CompactIndexHandler压缩索引
--通过将列中相同的值得字段进行压缩从而减小存储和加快访问时间
select count(*) from emp;

create table emp_copy as select * from emp;
create table emp_1 as select * from emp;

create index emp_sex_index
on table emp (sex)
as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler'
with deferred rebuild;
--指定了deferred rebuild，那么新索引将呈现空白状态。在任何时候，都可以进行第一次索引创建

--建立索引   Bitmap位图索引
--如果索引列只有固定的几个值，那么就可以采用位图索引来加速查询
--利用位图索引可以方便的进行AND/OR/XOR等各类计算
create index emp_sex_index_1
on table emp_1 (sex)
as 'BITMAP'
with deferred rebuild;


--重建索引
--指定了deferred rebuild，那么新索引将呈现空白状态。在任何时候，都可以进行第一次索引创建
--或者使用alter index对索引进行重建
alter index emp_sex_index_1 on emp_1 rebuild;

--索引生效，还得设置使用索引：默认是不使用的。
SET hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat;
SET hive.optimize.index.filter=true;
SET hive.optimize.index.filter.compact.minsize=0;


--显示索引
show formatted index on emp;

--删除索引
drop index emp_sex_index on emp;
```



## 5.Hive必会技术-函数

**一般函数  **

1、关系运算 > < =

```sql
-- 注意： String 的比较要注意(常用的时间比较可以先 to_date 之后再比较)
select 1 > 2;
select 1 < 2;
select 1 = 1;
select to_date('2017-01-01') > to_date('2018-01-01');

-- 空值判断
select 1 from dual where NULL is NULL limit 1;

-- 非空判断
select 1 from dual where 1 is not NULL limit 1;

--like
--字符串A符合表达式B的正则语法,则为TRUE;否则为FALSE. B中字符”_”表示任意单个字符，而字符”%”表示任意数量的字符。
select 1 from dual where '1234' like '12%';

select 1 from dual where '12x4' like '12_4';

--rlike
--语法: A RLIKE B
--描述: 字符串A符合JAVA正则表达式 B 的正则语法，则为 TRUE；否则为 FALSE
--  + 号代表前面的字符必须至少出现一次（1次或多次）

select 'qwerqwer123' rlike 'qwer+123';
```

2、数学运算 + - * /

hive中最高精度的数据类型是 double,只精确到小数点后16位，在做除法运算的时候要特别注意

```sql
select 100+100;

--   %
select 7 % 3;
```

3、逻辑运算 逻辑与AND 逻辑或OR 逻辑非NOT --   优先级依次为NOT AND OR。

```sql
select 1=2 and 3=4 or 1=1;
```



4、条件函数 IF CASE COALESCE
--说明: COALESCE返回参数中的第一个非空值；如果所有值都为 NULL，那么返回 NULL

```sql
select
if(4>5,5000,1000), 
COALESCE(null,1,3,5),
COALESCE(null,null,null,null),
case 3 
when 1 then 'lalala' 
when 2 then 'ohye' 
else 'nonono' end;
```



5、数值计算函数 

```sql
--取整: round
--语法: round(double a)
--说明: 遵循四舍五入
select round(1.79898);



--指定精度取整: round
--语法: round(double a, int d)
select round(1.789789,3);

--向下取整: floor
--说明: 返回等于或者小于该 double 变量的最大的整数
select floor(1.1223);
select floor(1.9898);

--向上取整: ceil
--说明: 返回等于或者大于该 double 变量的最小的整数
select ceil(1.1223);
select ceil(1.9898);

-- 向上取整: ceiling
--说明: 与ceil功能相同
select ceiling(1.1223);
select ceiling(1.9898);

--取随机数: rand
--说明: 返回一个 0 到 1 范围内的随机数。如果指定种子 seed(整数)，则会得到一个稳定的随机数序列。
select rand();
select rand(2);
```

**日期函数**

```sql
--UNIX时间戳转日期: from_unixtime
--日期转UNIX时间戳,指定格式日期转UNIX 时间戳,获取当前UNIX时间戳: unix_timestamp
--说明: 转换格式默认为"yyyy-MM-dd HH:mm:ss"的日期到 UNIX 时间戳。如果转化失败，则返回 0。
select 
    from_unixtime(1530950463),
    from_unixtime(1530950463,'yyyyMMdd')
 select   unix_timestamp(),
    unix_timestamp('2018-07-07 16:01:03'),
    unix_timestamp('20180707 16-01-03','yyyyMMdd HH-mm-ss');

--日期时间转日期:to_date  日期转年:year 日期转月:month 日期转天:day 日期转小时:hour  日期转分钟:minute  日期转秒:second
select
to_date('2018-07-07 10:03:01'),
year('2018-07-07 10:03:01'),
month('2018-07-07'),
day('2018-07-07 10:03:01'),
hour('2018-07-07 10:03:01'),
minute('2018-07-07 10:03:01'),
second('2018-07-07 10:03:01');

--日期转周:weekofyear  日期比较:datediff
select 
weekofyear('2018-01-01'),
weekofyear('2018-07-07'),
datediff('2018-08-07','2018-07-31');

-- 日期增加: date_add 日期减少: date_sub
select date_add('2018-07-07',10),date_add('2018-07-07',-10),
date_sub('2018-07-07',-1),date_sub('2018-07-07',1); 
```

**字符串**

```sql
--字符串长度:length,字符串反转:reverse,字符串连接:concat
select length('hello'),reverse('hello');



--字符串截取函数： substr,substring
--语法: substr(string A, int start),substring(string A, int start)
--说明：返回字符串 A 从 start 位置到结尾的字符串
select substr('abcdefg',3);
--语法: substr(string A, int start, int len),substring(string A, int start, int len)
--说明：返回字符串A从start位置开始，长度为len的字符串
select substr('abcdefg',2,3);



--字符串转大写:upper,ucase  字符串转小写:lower,lcase
select upper('abc');
select lower('ABC');



-- 去两边的空格:trim  左边去空格:ltrim  右边去空格:rtrim
select trim(' hello   ');
select ltrim(' hello   ');
select rtrim(' hello   ');



--左补足函数:lpad  右补足函数:rpad 
--语法: lpad(string str, int len, string pad)
select lpad('abc',10,'12'),rpad('abc',10,'12'); 



--分割字符串函数: split
select split('we,like',',');

select split('BK3+ERP+CTRK+YFGT','\\+');

drop table person;
create table person(
name string,
word string
);

insert into table person values
('mat','hi,hello'),
('ting','what,is');

select * from person;

SELECT name, word1  FROM person LATERAL VIEW explode(split(word,',')) add1 AS word1;



--集合查找函数: find_in_set
--语法: find_in_set(string str, string strList)
--说明: 返回 str 在 strlist 第一次出现的位置， strlist 是用逗号分割的字符串。如果没有找该 str 字符，则返回 0
select find_in_set('ab','ef,ab,de');
select find_in_set('ab','ef,de,gh');

-- string转map：str_to_map 
--语法：str_to_map(text, delimiter1, delimiter2)
--说明：使用两个分隔符将文本拆分为键值对。 Delimiter1将文本分成K-V对，Delimiter2分割每个K-V对。
--对于delimiter1默认分隔符是'，'，对于delimiter2默认分隔符是':'。
select str_to_map('a:1,b:2,c:3,d');
select str_to_map('aa-11+bb-22+cc-33', '\\+', '-');


--面向 array、map、struct 查询
use tszn;

create table s1017(
sno int,
sname string,
sage int,
slikes array<string>,
snote map<string,string>,
sscore struct<chinese:int,english:int,math:int>
)
Row Format Delimited   
FIELDS TERMINATED BY ','  
COLLECTION ITEMS TERMINATED BY '-'  
MAP KEYS TERMINATED BY ':'   
LINES TERMINATED BY '\n';    

load data local inpath '/opt/data01' into table s1017;

select * from s1017;

--查询某一个值
select slikes[1],snote['日语'],sscore.math from s1017; 

--查询array/map的长度
select size(slikes),size(snote) from s1017;

--元素的类型转化
select cast(sscore.math as double) from s1017;

--LATERAL VIEW 行转列
SELECT 
sname, hobby
FROM s1017 
LATERAL VIEW explode(slikes) addTable AS hobby;

SELECT 
sname, note,class
FROM s1017 
LATERAL VIEW explode(snote) addTable AS note,class;

SELECT 
sname, count(1)
FROM s1017 
LATERAL VIEW explode(snote) addTable AS note,class 
group by sname ;

SELECT 
sname,course,score
FROM s1017 
LATERAL VIEW explode(sscore) addTable AS course,score;

load data local inpath '/opt/data1017' into table s1017;

select * from s1017;

SELECT 
sname, hobby
FROM s1017 
LATERAL VIEW  explode(slikes) addTable AS hobby;

--OUTER 避免永远不产生结果，无满足条件的行，在该列会产生NULL值
SELECT 
sname, hobby
FROM s1017 
LATERAL VIEW OUTER explode(slikes) addTable AS hobby;
```

**窗口函数**

```sql
--做一个样例表
create table s1018(
cno int,
cname string,
cage int,
salary int,
job string,
department int,
leader string);

--插入数据
insert into table s1018 values(1001,'leo',26,16900,'DA',901,'mat'),
(1002,'max',21,7500,'DA1',901,'leo'),
(1003,'liu',22,9500,'DA1',901,'leo'),
(1004,'nana',29,10500,'DA',902,'mat'),
(1005,'mana',26,8000,'DA1',902,'nana'),
(1006,'mou',25,9500,'DA2',902,'nana'),
(1007,'zala',35,17500,'DA',903,'mat'),
(1008,'nano',36,9500,'DA1',903,'zala'),
(1009,'gala',28,6500,'DA1',903,'zala'),
(1010,'tuto',26,5500,'DA3',903,'zala'),
(1011,'alex',22,4500,'DA2',903,'zala');

--窗口函数使用来做什么的？
--1）没有窗口函数：
select job,avg(salary) from s1018 group by job;
--按维度聚合后记录变窄，若还想保留原记录（我习惯叫做表头，也就是聚合前的表不动）。
--2）用窗口函数：
select cno,cname,job,avg(salary) over(partition by job) as avg_salary 
from s1018 order by job,cno;

--分组排名
--RANK（并列排名计数）、ROW_NUMBER（行号）、DENSE_RANK（并列排名不计数）
select 
    cno,
    job,
    cname,
    salary,
    rank() over (partition by job order by salary desc) as r,
    row_number() over (partition by job order by salary desc) as r_n,
    dense_rank() over (partition by job order by salary desc) as d_r
from
    S1018
order by 
    salary,
    job;

--开窗区域选择范围，及COUNT、SUM、MIN、MAX、AVG等函数结合
select 
    cno,
    job,
    salary,
    row_number() over (partition by job order by salary asc) as r_n,
    --默认为从起点到当前行
    sum(salary) over(partition by job order by salary asc) as S_1,
    --从起点到当前行，与上面不同
    sum(salary) over(partition by job order by salary asc rows between unbounded preceding and current row) as S_2,
    --当前行+往前3行
    sum(salary) over(partition by job order by salary asc rows between 3 preceding and current row) as S_3,
    --当前行+往前3行+往后1行
    sum(salary) over(partition by job order by salary asc rows between 3 preceding and 1 following) as S_4,
    --当前行+往后所有行  
    sum(salary) over(partition by job order by salary asc rows between current row and unbounded following) as S_5,
    --往前所有行，不含当前行
    sum(salary) over(partition by job order by salary asc rows between unbounded preceding and 1 preceding) as S_6,
    --分组内所有行
    sum(salary) over(partition by job) as S_7
from 
    S1018
order by  
    job,
    salary,
    cno;
结果和ORDER BY相关,默认为升序
如果不指定ROWS BETWEEN,默认为从起点到当前行;
如果不指定ORDER BY，则将分组内所有值累加;
ROWS BETWEEN：
PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：无界限（起点或终点）
UNBOUNDED PRECEDING：表示从前面的起点 
UNBOUNDED FOLLOWING：表示到后面的终点
其他COUNT、AVG，MIN，MAX，和SUM用法一样。

--取组内特殊位置的数据
--first_value与last_value（不是亲兄弟）
--last_value的值不仅与partition相关，也与order相关，只有在两者都相同时，才被算做为同一组，并取同一组的最后一个值。
select 
    cno,
    job,
    cname,
    salary, 
    row_number() over (partition by job order by salary desc) as r,
    first_value(cname) over (partition by job order by salary desc) as max,
    first_value(cname) over (partition by job order by salary asc) as min,
    last_value(cname) over (partition by job order by salary desc) as last_min,
    last_value(cname) over (partition by job order by salary asc) as last_max
from 
    S1018;

--取邻行数据
--lead与lag
select
    cno,
    job,
    cname,
    salary,
    lead(cno) over (order by salary) as default_after_one_line,
    lag(cno) over (order by salary) as default_before_one_line,
    lead(cno,2) over (order by salary) as after_two_line,
    lag(cno,2,123) over (order by salary) as before_two_line
from 
    S1018
order by 
    salary,
    job;

--组内数据分桶（分等级，取高价值样本）
--NTILE
--把有序的数据集合平均分配到指定的数量（num）个桶中, 将桶号分配给每一行。
--如果不能平均分配，则优先分配较小编号的桶，并且各个桶中能放的行数最多相差1。
select 
    cno,
    job,
    cname,
    salary,
    --分组内将数据分成2片
    NTILE(2) OVER(PARTITION BY job ORDER BY salary) AS nt2,
    --分组内将数据分成3片    
    NTILE(3) OVER(PARTITION BY job ORDER BY salary) AS nt3,
    --分组内将数据分成4片    
    NTILE(4) OVER(PARTITION BY job ORDER BY salary) AS nt4,
    --将所有数据分成4片
    NTILE(4) OVER(ORDER BY salary) AS all_nt4
from 
    S1018
order by 
    job,
    salary;

--序列分析
--CUME_DIST、PERCENT_RANK 
--CUME_DIST 小于等于当前值的行数/分组内总行数，
--比如，统计小于等于当前薪水的人数，所占总人数的比例；
--PERCENT_RANK 分组内当前行的RANK值-1/分组内总行数-1，
--百分比排位。

select 
    cno,
    job,
    cname,
    salary,
--没有partition,所有数据均为1组
    CUME_DIST() OVER(ORDER BY salary) AS cd1,
--按照job进行分组
    CUME_DIST() OVER(PARTITION BY job ORDER BY salary) AS cd2 
from 
    S1018;   

select 
    cno,
    job,
    cname,
    salary,
--分组内总行数      
    SUM(1) OVER(PARTITION BY job) AS s, 
--RANK值  
    RANK() OVER(ORDER BY salary) AS r,    
    PERCENT_RANK() OVER(ORDER BY salary) AS pr,
--分组内     
    PERCENT_RANK() OVER(PARTITION BY job ORDER BY salary) AS prg 
from 
    S1018;




--Grouping 、Cube、Rollup
--通常用于OLAP中，不能累加，而且需要根据不同维度上钻和下钻的指标统计

--1）GROUPING SETS
--在一个GROUP BY查询中，根据不同的维度组合进行聚合，等价于将不同维度的GROUP BY结果集进行UNION ALL, 
--其中的GROUPING__ID，表示结果属于哪一个分组集合。

--补充：UNION与UNINON ALL
(1,leo,23
2,max,24
3,lily,26)
union              5行结果 去重
(4,nemo,21
1,leo,23
5,matt,29)

(1,leo,23
2,max,24
3,lily,26)
union all         6行结果  不去重        
(4,nemo,21
1,leo,23
5,matt,29)

--根据job，salary分别聚合计算，所以GROUPING__ID是两类
select
    job,
    salary,
    count(cno) as counts,
    GROUPING__ID 
from 
    S1018
group by 
    job,salary
GROUPING SETS(job,salary) 
ORDER BY 
    GROUPING__ID;
--根据job，salary，(job,salary)分别聚合计算，所以GROUPING__ID是三类
select
    job,
    salary,
    count(cno) as counts,
    GROUPING__ID 
from 
    S1018
group by 
    job,salary
GROUPING SETS(job,salary,(job,salary)) 
ORDER BY 
    GROUPING__ID;

--2）CUBE
--根据GROUP BY的维度的所有组合进行聚合。
select
    job,
    salary,
    count(cno) as counts,
    GROUPING__ID 
from 
    S1018
group by 
    job,salary
WITH CUBE 
ORDER BY 
    GROUPING__ID;

--3）ROLLUP 
--是CUBE的子集，以最左侧的维度为主，从该维度进行层级聚合。
select
    job,
    salary,
    count(cno) as counts,
    GROUPING__ID 
from 
    S1018
group by 
    job,salary
WITH ROLLUP 
ORDER BY 
    GROUPING__ID;
```



## 6.Spark SQL on Hive 





