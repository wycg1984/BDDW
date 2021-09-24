# PostgreSQL学习笔记

PostgreSQL 12.2 手册 http://www.postgres.cn/docs/12/index.html

![image-20210802110809387](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210802110809387.png)

创建数据库
create database db_jike;
创建数据库指定参数信息 指定了所有者 及编码格式
create database db_jike with owner = postgres encoding = 'utf-8';

修改数据库名称
alter database db_jike rename to db_jike2;
修改数据库连接数
alter database db_jike connection limit 20;
删除数据库
drop database db_jike;

创建数据表对象
create table students
(
	id int,
	name varchar(30),
	birthday date,
	score numeric(5,2)
);
修改数据表对象 修改数据表名称
alter table students rename to student1;
修改数据表中的字段名称
alter table student1 rename id to bh;
修改数据表中的字段的类型
alter table student1 alter column name type varchar(50);
修改数据表中删除字段
alter table student1 drop column birthday;
修改数据表新增字段
alter table student1 add column address varchar(200);

删除数据表对象
drop table student1;
drop table if exists student1;

## PostgresSQL常用数据类型

1、数值类型
整数类型：
SMALLINT //小范围整数 取值范围 -32768 ~ 32767
INT(INTEGER) //普通大小整数 取值范围 -2147483648 ~ 32147483647

任意精度浮点数类型：

REAL					//64位十进制数字精度

NUMERIC(m , n)  //任意精度类型



2、日期与时间类型

TIME			 只用于一日内时间		8字节		10:05:05

DATE			 只用于日期				   4字节		1987-04-04

TIMESTAMP 日期和时间				   8字节		1987-04-04 																				  10:05:05



3、字符串类型

CHAR(n)/CHARACTER(n)			固定长度字符串，不足补空白

VARCHAR(n)/CHARACTER VARYING(n) 变长字符串，有长度限制

TEXT  变长字符串，无长度限制



选择正确的数据类型

主要目的：优化存储，提高数据库性能



## PostgreSQL运算符介绍

算术运算符：+	-	*	/	%



比较运算符：=	<>/！=	<=	>=	>	<	

LEAST			在有两个或多个参数时，返回最小值

GREATEST 	在有两个或多个参数时，返回最大值

BETWEEN AND 判断一个值是否落在两个值之间

IN	判断一个值是否是IN列表中的任意一个值

LIKE	通配符匹配



逻辑运算符

NOT（逻辑非）

AND（逻辑与）

OR（逻辑或）

## PostgreSQL常用函数

函数的作用：封装一些通用的逻辑功能，方便程序的调用

**常用的数值函数**

AVG()			  返回某列的平均值

COUNT()		返回某列的行数

MAX()			 返回某列的最大值

MIN()			 返回某列的最小值

SUM()			返回某列的值之和

**常用的字符串函数**

LENGTH(s)			计算字符串长度

CONCAT(s1,s2,...)		字符串合并函数

LTRIM(s)/RTRIM(s)/TRIM(s)	删除字符串空格函数

REPLACE(s,s1,s2)		字符串替换函数

SUBSTRING(s,n,len)		获取子串函数

**常用的时间函数**

EXTRACT(tye FROM d)	获取日期指定值函数

CURRENT_DATE		获取当前日期函数

CURRENT_TIME		 获取当前时间函数

NOW()		获取当前日期时间函数

## PostgreSQL自定义函数

创建函数的语法

基本语法格式：

CREATE FUNCTION 	//声明创建函数

​		add(integer,integer)		//定义函数名称，参数类型

RETURNS integer					//定义函数返回值

​		AS 'select $1 +$2;'				//定义函数体

LANGUAGE SQL						//用以实现函数的语言的名字

RETURNS NULL ON NULL INPUT;	//定义参数为NULL时处理情况



函数的创建

```sql
create or replace function concat_test(integer,varchar,date)
returnd varchar
	as 'SELECT  $1 || $2|| $3;'
	language sql
returns null on null input;
```

```sql
select add(1,2)
```

```sql
select e_no,concat_test(e_no,e_name,e_hireDate) from employee;
```

函数的删除

```sql
drop function concat_test(integer,varchar,date);
```



## PostgreSQL数据库索引介绍

索引的作用：提高数据的检索效率

索引的分类

B-tree索引：适合处理哪些能够按顺序存储数据

Hash索引：只能处理简单的等于比较

GiST索引：一种索引架构

GIN索引：发转索引，处理包含多个值的键

索引的创建和删除

```sql
create index emp_name_index on employee(e_name);

drop index emp_name_index;
```

使用索引的优点和缺点

优点

​		提高数据的查询速度

​		加速表与表之间的连接

缺点

​		创建和维护索引需要耗费时间

​		需要占用磁盘空间



## PostgreSQL数据库视图介绍

视图的含义：是从一张表或多张表中

视图的创建

```sql
create view v_emp_dev as select e_no,e_name,e_salary 

from employee where dept_no = 10  order by e_salary desc;

select * from v_emp_dev;
```

视图的删除

```sql
drop view v_emp_dev;
```

视图的作用

​	简单化

​	安全性

​	逻辑数据独立性

## PostgreSQL数据库事务





## 简单数据插入操作

```sql
create table student(

	id int,

	name varchar(30),

	birthday date,

	score numeric(5,2)

);
```

向单表中插入数据

```sql
insert into student values(1,'张三','1990-01-01',3.85);
```

向数据表中指定字段插入数据

```sql
insert into  student(id,name,birthday) values(2,'张四','1990-01-01');
```



## 批量数据插入操作

使用INSERT语句批量向数据表中插入数据

```sql
insert into  student(id,name,birthday) values

(3,'张四','1990-01-01'),

(4'张四','1990-01-01'),

(5,'张四','1990-01-01');
```

使用SELECT查询批量向数据表中插入数据

```sql
insert into student_new select * from student;
insert into student_new(id,name) select id,name from student;
```



## 数据更新操作

```sql
update student set name = '' where id = 2;
update student set score = 0;
```



## 数据删除操作

```sql
delete from student where id = 3;
truncate table student;
```



## 数据表主键，外键

```sql
create table emp(

	id int primary key,

	name varchar(30),

	salary numeric(9,2)

);

create table dept(

	id int primary key,

	name varchar(40)

);

insert into dept values(1,'开发部');

insert into dept values(2,'测试部');

```

主键约束作用：唯一标识一条记录；提高数据的检索效率。

## 数据表非空约束、唯一约束、默认值约束

非空约束 not null

```sql
create table emp(

	id int primary key,
	
	name varchar(30) not null,
	
	salary numeric(9,2)

);
```

唯一约束 unique

```sql
create table emp(

	id int primary key,
	
	name varchar(30) not null,
	phone varchar(30) unique,
	salary numeric(9,2)

);
```

默认值约束 default 值

```sql
create table emp(

	id int primary key,
	
	name varchar(30) not null,
	phone varchar(30) unique,
	salary numeric(9,2) default 0.0

);
```



## 简单数据查询操作

```sql
select *|[字段列表] from 表名 where 条件
```



## 单表指定条件查询操作

查询指定记录

```sql
select e_no,e_name from employee where e_salary > 5000;
```

IN关键字查询

```sql
select e_no,e_name，dept_no from employee where dept_no in (20,30);
```

BETWEEN AND范围查询

```sql
select e_no,e_name from employee where e_bireDate between '2010-01-01' and '2010-12-01';
```

LIKE模糊查询

```sql
select e_no,e_name from employee where e_name like '李%';
```



## 单表指定条件复杂查询操作

查询空值内容

```sql
select e_no,e_name,e_salary  from employee where e_salary is null;
```

AND、OR多条件查询

```sql
select e_no,e_name,e_salary  from employee where e_gender = 'f' and dept_no = 10;
```

查询结果排序

```sql
select e_no,e_name,e_salary  from employee order by e_salary asc nulls first/last;
```

LIMIT关键字查询

```sql
select e_no,e_name,e_salary  from employee limit 10;
select e_no,e_name,e_salary  from employee limit 10 offset 5;--offset忽略5条数据
```



## 多表连接查询操作

INNER JOIN连接查询操作

```sql
select e_no,e_name,dept_no,d_name  from employee,dept where dept_no = d_no ;
select e_no,e_name,dept_no,d_name  from employee inner join dept on dept_no = d_no ;
```

LEFT JOIN连接查询操作

```sql
select e_no,e_name,dept_no,d_name  from employee left join dept on dept_no = d_no ;
```

RIGHT JOIN连接查询操作

```sql
select e_no,e_name,dept_no,d_name  from employee right join dept on dept_no = d_no ;
```



## 子查询操作

EXISTS关键字子查询操作

```sql
select * from employee where exitst (select d_no from dept where d_name = '开发部');
```

IN关键字子查询操作

```sql
select * from employee where dept_no in (select d_no from dept where d_name = '开发部');
```

标量子查询操作

```sql
select e_no,e_name,(select d_name || ' ' || d_location from dept where dept_no = d_no ) as address from employee;
```



## 查询结果集合并操作

```sql
select e_no,e_name,dept_no,d_name  from employee where dept_no in (10,20));

union all

select e_no,e_name,dept_no,d_name  from employee where e_salary > 5000;
```

数据库定位

![img](https://img-blog.csdnimg.cn/img_convert/ed28917324b5f5c0d416176a2c61214c.png)

- **OLTP**：(增删改)（on-line transaction processing）翻译为联机事务处理
- **OLAP**：(查)（On-Line Analytical Processing）翻译为联机分析处理

- **SMP**：对称多处理（Symmetrical Multi-Processing），是指在一个计算机上汇集了一组处理器(多CPU),各CPU之间[共享内存](https://baike.baidu.com/item/共享内存/2182364)子系统以及[总线结构](https://baike.baidu.com/item/总线结构/10183496)。它是相对非对称多处理技术而言的、应用十分广泛的[并行技术](https://baike.baidu.com/item/并行技术/3345169)。

- **JIT**：在PostgreSQL等数据库中，JIT指的是即时编译(Just-in-time Compilation)，即程序在运行过程中即时进行编译，其中可以把编译的中间代码缓存或者优化。相对于静态编译代码，即时编译的代码可以处理延迟绑定并增强安全性。简单的说 JIT 是一种提高程序运行效率的方法。通常，程序有两种运行方式：静态编译与动态直译。静态编译的程序在执行前全部被翻译为机器码，而直译执行的则是一句一句边运行行边翻译。 JIT 则混合了这二者，一句一句编译源代码，但是会将翻译过的代码缓存起来以降低性能损耗。相对于静态编译代码，即时编译的代码可以处理延迟绑定并增强安全性。

- **向量计算**：vops插件中新增了瓦片式数据类型，而瓦片本身支持向量计算。向量计算的本质是一次计算多个值，减少函数调用，上下文切换，尽量的利用CPU的缓存以及向量化执行指令提高性能。新增的瓦片式类型，对应的还新增了对应的向量化执行操作符，所以使用VOPS和正常的使用SQL语法是一样的

### 常见集群部署架构

底层都是基于流复制实现的

![img](https://img-blog.csdnimg.cn/img_convert/c121f7b3d4ca14e0253be95d0025da39.png)

![img](https://img-blog.csdnimg.cn/img_convert/fe1ed1d6b0c341b26817e4deb9804b3f.png)

### 数据备份

![img](https://img-blog.csdnimg.cn/img_convert/de3686bab5c31dfa948e39cb0e2c05ed.png)

### 数据还原

![img](https://img-blog.csdnimg.cn/img_convert/67eb9c6e7cb5c1f90c5f59b1b73aa0e7.png)

![img](https://img-blog.csdnimg.cn/img_convert/982d25402556ec6768176e4dcda9967a.png)

### 应用场景

![img](https://img-blog.csdnimg.cn/img_convert/efaec4cce6c8f4b2f48ed96efbe9de62.png)

### PG多模-解决传统架构痛点

![img](https://img-blog.csdnimg.cn/img_convert/02358b55b515da2feb59ed5fb6992659.png)

### PG相关数据库

![img](https://img-blog.csdnimg.cn/img_convert/0c5dec4b078319e58652787a47db9c94.png)

![img](https://img-blog.csdnimg.cn/img_convert/d8ff0bafab9fa55ef7d918de30d30362.png)

![img](https://img-blog.csdnimg.cn/img_convert/82511addc9f438b93bb28da18aa9d2b0.png)

## 安装

- 安装依赖环境

  ```shell
  yum install gcc zlib-devel readline-devel -y
  ```

  下载源码（官网下载对应版本）

  ![img](https://img-blog.csdnimg.cn/img_convert/98f3a7083112da3f67318ac3b52cfa86.png)

  解压

  ```shell
  tar -zxvf postgresql-12.5.tar.gz
  ```


  设定源码编译选项
  使用**./configure – -help**命令来查看不同选项的帮助信息，示例输出如下：

```shell
`configure' configures PostgreSQL 12.5 to adapt to many kinds of systems.

Usage: ./configure [OPTION]... [VAR=VALUE]...

To assign environment variables (e.g., CC, CFLAGS...), specify them as
VAR=VALUE.  See below for descriptions of some of the useful variables.

Defaults for the options are specified in brackets.

Configuration:
  -h, --help              display this help and exit
      --help=short        display options specific to this package
      --help=recursive    display the short help of all the included packages
  -V, --version           display version information and exit
  -q, --quiet, --silent   do not print `checking ...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
  -C, --config-cache      alias for `--cache-file=config.cache'
  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local/pgsql]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]
  ***

```

现在新建一个存储PostgreSQL程序文件的目录，并在configure命令中使用prefix选项

```shell
mkdir /opt/PostgreSQL-12/
./configure --prefix=/opt/PostgreSQL-12
```


示例输出如下：

```shell
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking which template to use... linux
checking whether NLS is wanted... no
checking for default port number... 5432
checking for block size... 8kB
checking for segment size... 1GB
checking for WAL block size... 8kB
checking for WAL segment size... 16MB
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc supports -Wdeclaration-after-statement... yes
checking whether gcc supports -Wendif-labels... yes
checking whether gcc supports -Wmissing-format-attribute... yes
checking whether gcc supports -Wformat-security... yes
checking whether gcc supports -fno-strict-aliasing... yes
checking whether gcc supports -fwrapv... yes
checking whether gcc supports -fexcess-precision=standard... no
....
```

编译完成后，就可以用make命令来生成PostgreSQL

```shell
make
```


生成完成后，使用以下命令来进行PostgreSQL的安装

```shell
make install
```


全部执行完成后，Postgresql 12 就会被安装在 /opt/PostgreSQL-12目录。

现在需要添加一个postgres用户并创建一个存放数据文件的目录，这个目录的所有者应该为数据库用户postgres，并且将该目录的权限设置为700以保证数据文件的安全性。最后，需要将PostgreSQL的执行文件添加至系统的PATH环境变量中

```shell
useradd postgres
passwd postgres
mkdir /pgdatabase/data
chown -R postgres. /pgdatabase/data
echo 'export PATH=$PATH:/opt/PostgreSQL-12/bin' > /etc/profile.d/postgres.sh

```

然后就可以切换至postgres用户来对数据库进行初始化操作了

```shell
su postgres
$ initdb -D /pgdatabase/data/ -U postgres -W
```

![img](https://img-blog.csdnimg.cn/img_convert/a6a3aa41690078dcd012be2deeae9193.png)


其中-D参数用于指定数据文件位置；-U参数用于指定操作数据库的超级用户；-W参数为需要设置超级用户密码。如果想要了解更多initdb工具的详细信息，可以使用initdb –help命令来获得更多帮助信息。

数据库初始化完成后，可以通过修改数据文件目录的postgresql.conf配置文件来达到设置postgresql侦听端口的目的

```shell
listen_addresses = '*'    

```

修改完配置文件后，需要用以下命令重新启动PostgreSQL

```shell
chown -R postgres. /pglog/db_logs/
pg_ctl -D /pgdatabase/data/ -l /pglog/db_logs/start.log start
#停止数据库

# pg_ctl -D /pgdatabase/data/ stop
```


其中-l /pglog/db_logs/start.log参数是用于设置日志文件，方便查看信息，可以根据自己的实际情况自行设置，如不需要也可以去掉这部分内容，并不影响数据库服务器启动。

![img](https://img-blog.csdnimg.cn/img_convert/b345bef66b2ab138cda1f27098c9f4fb.png)

数据库服务启动后，可以用以下命令查看进程信息以及端口信息来进行确认

```shell
ps -ef |grep -i postgres
netstat -apn |grep -i 5432
```

![img](https://img-blog.csdnimg.cn/img_convert/a60a2e9f80ee9fc5d38c8b2abba2f67a.png)

连接数据库，执行SQL语句

```shell
$ psql -p 5432
postgres=# create database test;
postgres=# \l 用于显示所有数据库
postgres=# \q 退出psotgre命令提示符

```

参考：

https://github.com/digoal/blog/blob/master/201901/20190105_01.md

Linux中内核配置
参考：

https://github.com/digoal/blog/blob/master/201611/20161121_01.md

https://github.com/digoal/blog/blob/master/201608/20160803_01.md

sysctl

```shell
# vi /etc/sysctl.conf

# add by digoal.zhou
fs.aio-max-nr = 1048576
fs.file-max = 76724600
kernel.core_pattern= /data01/corefiles/core_%e_%u_%t_%s.%p         
# /data01/corefiles事先建好，权限777，如果是软链接，对应的目录修改为777
kernel.sem = 4096 2147483647 2147483646 512000    
# 信号量, ipcs -l 或 -u 查看，每16个进程一组，每组信号量需要17个信号量。
kernel.shmall = 107374182      
# 所有共享内存段相加大小限制(建议内存的80%)
kernel.shmmax = 274877906944   
# 最大单个共享内存段大小(建议为内存一半), >9.2的版本已大幅降低共享内存的使用
kernel.shmmni = 819200         
# 一共能生成多少共享内存段，每个PG数据库集群至少2个共享内存段
net.core.netdev_max_backlog = 10000
net.core.rmem_default = 262144       
# The default setting of the socket receive buffer in bytes.
net.core.rmem_max = 4194304          
# The maximum receive socket buffer size in bytes
net.core.wmem_default = 262144       
# The default setting (in bytes) of the socket send buffer.
net.core.wmem_max = 4194304          
# The maximum send socket buffer size in bytes.
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_keepalive_intvl = 20
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_time = 60
net.ipv4.tcp_mem = 8388608 12582912 16777216
net.ipv4.tcp_fin_timeout = 5
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syncookies = 1    
# 开启SYN Cookies。当出现SYN等待队列溢出时，启用cookie来处理，可防范少量的SYN攻击
net.ipv4.tcp_timestamps = 1    
# 减少time_wait
net.ipv4.tcp_tw_recycle = 0    
# 如果=1则开启TCP连接中TIME-WAIT套接字的快速回收，但是NAT环境可能导致连接失败，建议服务端关闭它
net.ipv4.tcp_tw_reuse = 1      
# 开启重用。允许将TIME-WAIT套接字重新用于新的TCP连接
net.ipv4.tcp_max_tw_buckets = 262144
net.ipv4.tcp_rmem = 8192 87380 16777216
net.ipv4.tcp_wmem = 8192 65536 16777216
net.nf_conntrack_max = 1200000
net.netfilter.nf_conntrack_max = 1200000
vm.dirty_background_bytes = 409600000       
#  系统脏页到达这个值，系统后台刷脏页调度进程 pdflush（或其他） 自动将(dirty_expire_centisecs/100）秒前的脏页刷到磁盘
vm.dirty_expire_centisecs = 3000             
#  比这个值老的脏页，将被刷到磁盘。3000表示30秒。
vm.dirty_ratio = 95                          
#  如果系统进程刷脏页太慢，使得系统脏页超过内存 95 % 时，则用户进程如果有写磁盘的操作（如fsync, fdatasync等调用），则需要主动把系统脏页刷出。
#  有效防止用户进程刷脏页，在单机多实例，并且使用CGROUP限制单实例IOPS的情况下非常有效。  
vm.dirty_writeback_centisecs = 100            
#  pdflush（或其他）后台刷脏页进程的唤醒间隔， 100表示1秒。
vm.mmap_min_addr = 65536
vm.overcommit_memory = 0     
#  在分配内存时，允许少量over malloc, 如果设置为 1, 则认为总是有足够的内存，内存较少的测试环境可以使用 1 .  
vm.overcommit_ratio = 90     
#  当overcommit_memory = 2 时，用于参与计算允许指派的内存大小。
vm.swappiness = 0            
#  关闭交换分区
vm.zone_reclaim_mode = 0     
# 禁用 numa, 或者在vmlinux中禁止. 
net.ipv4.ip_local_port_range = 40000 65535    
# 本地自动分配的TCP, UDP端口号范围
fs.nr_open=20480000
# 单个进程允许打开的文件句柄上限
net.ipv4.tcp_max_syn_backlog = 16384
net.core.somaxconn = 16384

# 以下参数请注意
# vm.extra_free_kbytes = 4096000
# vm.min_free_kbytes = 2097152 # vm.min_free_kbytes 建议每32G内存分配1G vm.min_free_kbytes
# 如果是小内存机器，以上两个值不建议设置
# vm.nr_hugepages = 66536    
#  建议shared buffer设置超过64GB时 使用大页，页大小 /proc/meminfo Hugepagesize
# vm.lowmem_reserve_ratio = 1 1 1
# 对于内存大于64G时，建议设置，否则建议默认值 256 256 32

```

生效配置

```shell
sysctl -p
```


概念：

脏页－linux内核中的概念，因为硬盘的读写速度远赶不上内存的速度，系统就把读写比较频繁的数据事先放到内存中，以提高读写速度，这就叫高速缓存，linux是以页作为高速缓存的单位，当进程修改了高速缓存里的数据时，该页就被内核标记为脏页，内核将会在合适的时间把脏页的数据写到磁盘中去，以保持高速缓存中的数据和磁盘中的数据是一致的。
Liunx限制设置

```shell
# vi /etc/security/limits.conf

# nofile超过1048576的话，一定要先将sysctl的fs.nr_open设置为更大的值，并生效后才能继续设置nofile.

* soft    nofile  1024000
* hard    nofile  1024000
* soft    nproc   unlimited
* hard    nproc   unlimited
* soft    core    unlimited
* hard    core    unlimited
* soft    memlock unlimited
* hard    memlock unlimited

```



* 大页配置

![img](https://img-blog.csdnimg.cn/img_convert/201662922910f3264ead12251346a42b.png)



![img](https://img-blog.csdnimg.cn/img_convert/83d88247f3ad268007141fd410b506a3.png)

![img](https://img-blog.csdnimg.cn/img_convert/384c1ea2e6482235ed1a5ea686d5d2d0.png)

![img](https://img-blog.csdnimg.cn/img_convert/3d4b3b057d9b44c51a951248ee685481.png)

![img](https://img-blog.csdnimg.cn/img_convert/88f81db733b979153dbb26e9add24f58.png)



存储卷管理![img](https://img-blog.csdnimg.cn/img_convert/14a2423ed75c1b3a3f5ef5132ebd95de.png)


参考：https://github.com/digoal/blog/blob/master/201809/20180919_01.md

文件系统条带、INODE、MOUNT配置

![img](https://img-blog.csdnimg.cn/img_convert/a431abbdb70da12a4b578716e9f617e6.png)

![img](https://img-blog.csdnimg.cn/img_convert/9ac5a4b3d466e13bb96020bab44ee8f3.png)

![img](https://img-blog.csdnimg.cn/img_convert/8c12b60975cf59d32836fc592573f1f2.png)

资源隔离

![img](https://img-blog.csdnimg.cn/img_convert/2ebabbdcdc8a99c138be974eb62ffdc4.png)


编译安装
参考：

https://github.com/digoal/blog/blob/master/201611/20161121_01.md
https://github.com/digoal/blog/blob/master/201710/20171018_01.md
插件安装

![img](https://img-blog.csdnimg.cn/img_convert/f46749dc0e2cb73f61f2599f3520a47b.png)

![img](https://img-blog.csdnimg.cn/img_convert/0b0687515011cceb0f187aa7a45e2214.png)

大版本升级- 二进制校验![img](https://img-blog.csdnimg.cn/img_convert/373180eb6570675b8547bdd9c1ebe328.png)


参考：https://github.com/digoal/blog/blob/master/201412/20141219_01.md

**实例初始化、基本配置**
目标：

熟悉数据库初始化、参数、防火墙、物理架构、逻辑结构、权限体系

**初始化数据库集群**

![img](https://img-blog.csdnimg.cn/img_convert/be94f05e9ae75536ccada6d84d1ab371.png)

Redo log是一个基于磁盘的数据格式，在数据库灾难恢复过程中，用于恢复未完成的事物，修复数据的。在正常的操作过程中，redo log将来自于SQL 语句或者其他API调用的请求，转化为数据表中的变化，当对数据的改变操作还未完成之前，遭遇了意外的停机，那这些变化会在数据库启动初始化的过程中进行replay重演，这个重演的过程，是在接受client请求之前，就会完成。
**配置文件**
参考地址

优先级

数据库配置 > postgresql.auto.conf > postgresql.conf

```sql
-- 查看数据库配置
select * from pg_db_role_setting;

-- 重置指定数据库的所有参数
alter database postgres reset all;

-- 用户的数据操作内存
show work_mem;

-- 查看当前数据库中用户信息
select * from pg_authid;

-- 修改用户密码
alter role user encrypted password '123';

-- 统计获取时间的开销
pg_test_timing
```


**参数优先级**
[参数优先级](https://github.com/digoal/blog/blob/master/201901/20190130_03.md)

**PG防火墙配置**![img](https://img-blog.csdnimg.cn/img_convert/e4d61777573b71f2dba6ed825a029113.png)

**数据库进程结构**

![img](https://img-blog.csdnimg.cn/img_convert/ef996ffdc0c888893844bdcafd13f260.png)

![img](https://img-blog.csdnimg.cn/img_convert/f6467d11563668eabfe0726dafa57d6a.png)

**逻辑结构**
参考

![img](https://img-blog.csdnimg.cn/img_convert/61e87fe47b75a3f5063839ccfa96caa6.png)

![img](https://img-blog.csdnimg.cn/img_convert/2ac48770159c538a47fb26bc014f02f0.png)

**SSL链路**![img](https://img-blog.csdnimg.cn/img_convert/7ac801f99884b46050618292e8b7ee1b.png)

**PostgreSQL常用命令总结**
**启动关闭**
启动

![img](https://img-blog.csdnimg.cn/img_convert/22b7e5ce7a48db72311d0cc0a7a10a03.png)

自启动

![img](https://img-blog.csdnimg.cn/img_convert/ad8dd540d27c2c5dc588cc49291e46af.png)

关闭

![img](https://img-blog.csdnimg.cn/img_convert/4160b296ad66c5b45d154f76fda61027.png)

**单用户模式**

```shell
su - postgres
psql
-- 快速关闭数据库
pg_ctl stop -m fast
-- 单用户启动某个数据库
postgres --single postgres
-- 回收事务XID
vacuum freeze;

top -c -u postgres
```

**性能测试**
硬盘读写性能

```shell
cd /pgdatabase/data
su postgres

# 测试硬盘读写

pg_test_fsync
```


结果如下：

```shell
5 seconds per test
O_DIRECT supported on this platform for open_datasync and open_sync.

Compare file sync methods using one 8kB write:
(in wal_sync_method preference order, except fdatasync is Linux's default)
        open_datasync                     15080.407 ops/sec      66 usecs/op
        fdatasync                         13709.482 ops/sec      73 usecs/op
        fsync                             12809.898 ops/sec      78 usecs/op
        fsync_writethrough                              n/a
        open_sync                         14083.904 ops/sec      71 usecs/op

Compare file sync methods using two 8kB writes:
(in wal_sync_method preference order, except fdatasync is Linux's default)
        open_datasync                      7601.431 ops/sec     132 usecs/op
        fdatasync                         12942.465 ops/sec      77 usecs/op
        fsync                             12128.903 ops/sec      82 usecs/op
        fsync_writethrough                              n/a
        open_sync                          7065.962 ops/sec     142 usecs/op

Compare open_sync with different write sizes:
(This is designed to compare the cost of writing 16kB in different write
open_sync sizes.)
         1 * 16kB open_sync write         14058.028 ops/sec      71 usecs/op
         2 *  8kB open_sync writes         6994.634 ops/sec     143 usecs/op
         4 *  4kB open_sync writes         3572.526 ops/sec     280 usecs/op
         8 *  2kB open_sync writes         1767.462 ops/sec     566 usecs/op
        16 *  1kB open_sync writes          892.704 ops/sec    1120 usecs/op

Test if fsync on non-write file descriptor is honored:
(If the times are similar, fsync() can sync data written on a different
descriptor.)
        write, fsync, close               12142.198 ops/sec      82 usecs/op
        write, close, fsync               11862.460 ops/sec      84 usecs/op

Non-sync'ed 8kB writes:
        write                            333563.771 ops/sec       3 usecs/op

```

**查看配置命令**
查看异步REDO是否开启，默认开启

```
postgres=# show synchronous_commit;
 synchronous_commit 
--------------------
 on
(1 row)

```

查看开启REDO后的数据库wal_writer延迟

```
postgres=# show wal_writer_delay ;
 wal_writer_delay 
------------------
 200ms
(1 row)

```

当服务器崩溃的时候，可能会丢掉200ms内的数据

数据wal的大小范围

```
postgres=# show max_wal_size ;
 max_wal_size 
--------------
 1GB
(1 row)

postgres=# show min_wal_size ;
 min_wal_size 
--------------
 80MB
(1 row)

```

**扩展模块**
PostgresSQL支持丰富的扩展模块，扩展模块可以完善PostgreSQL的功能，主要分为两类

编译安装PostgreSQL时使用world选项安装的扩展模块
GitHub或第三方网站开源项目
使用gmake命令编译安装PostgreSQL时，指定world选项，安装到$GHOME/share/extension目录

```
gmake world
gmake install-world

```

指定数据库安装扩展

```
CREATE EXTENSION [IF NOT EXISTS] extension_name
-- CREATE EXTENSION pg_buffercache

```

查看当前数据库的扩展

```
\dx
SELECT * FROM pg_exteension

```

查看数据库有哪些可加载的扩展模块

```
SELECT * FROM pg_available_extension

```

**常用模块**
**pg_stat_statements**
用于收集数据库中的SQL运行信息，例如SQL的总执行时间、调用次数、共享内存命中情况，常用语监控数据库SQL性能

安装：

修改postgresql.conf

```
# 表示要在启动时导入pg_stat_statements 动态库
shared_preload_libraries = 'pg_stat_statements'
# 表示监控的语句最多为10000句
pg_stat_statements.max = 10000
# 设置哪类SQL会被记录，all表示全部，top记录最外层不包含嵌套的SQL
pg_stat_statements.track = all
# 表示对 INSERT/UPDATE/DELETE/SELECT 之外的sql动作也作监控
pg_stat_statements.track_utility = on
# 表示当postgresql停止时，把信息存入磁盘文件以备下次启动时再使用
pg_stat_statements.save = on

```

重启postgres

```
pg_ctl restart -m fast
```



使用：

**创建表空间**
windows上使用pgadmin4新建表空间，并指定目录，在创建数据库、索引或表的时候指定表空间

![img](https://img-blog.csdnimg.cn/img_convert/fa67521b033d645b73d8b8621429630f.png)

![img](https://img-blog.csdnimg.cn/img_convert/f7903ffb8e774e38679ee79bd2e90622.png)

linux上可以直接使用命令行创建
```sql
# 在指定的数据文件夹下创建目录

mkdir -p /data/pg_data/tsp

# 为postgres用户赋予权限

sudo chown -R postgres:postgres /data/pg_data/tsp

# 创建表空间

create tablespace tbs_test owner postgres location '/data/pg_data/tsp';
```







# PostgreSQL函数实战

