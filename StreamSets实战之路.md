# StreamSets实战之路（一）-环境篇- StreamSets简介

## StreamSets总体介绍

StreamSets是国外一家致力于数据处理与分析的大数据解决方案的公司。公司主要选择DataOps发展路线，解决将数据转化为业务价值的重大挑战。至于为什么选择DataOps这条路子，有兴趣的同学可以查看https://streamsets.com/why-dataops/what-is-dataops/。

自公司成立以来，成功研制了多款用于数据处理的软件及平台。下图是该公司主要的产品：

![img](https://img-blog.csdnimg.cn/20200425120213822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

Streamsets推出的产品主要包含：Data Collector、Transformer和Control Hub。Data Collector和Transformer主要用于数据收集、处理、分析等，Control Hub作为一个平台管理软件提供设计、发布、监控、智能数据流管理等。

Data Collector：是一种轻量级，功能强大的设计和执行引擎，可实时传输数据。使用该软件来路由和处理数据流中的数据。目前为开源产品。

Transformer：是一个执行引擎，可在Apache Spark（一种开放源代码群集计算框架）上运行数据处理管道。由于Transformer管道在群集上部署的Spark上运行，因此管道可以执行转换，这些转换需要以批处理或流模式对整个数据集进行大量处理。暂未开源。

Control Hub：是所有数据流管道的中央控制点。Control Hub允许团队大规模构建和执行大量复杂的数据流。收费产品，可申请试用。

三款产品提供方便用户操作的用户界面，设计及管理业务数据流只需要通过拖拽组件的方式进行，设计数据处理流程中基本上不需要开发多余的代码，并且在设计过程中随时可以验证数据处理的结果与输入的结果进行对比，最终可以验证整个数据处理流程正确性。

Data Collector和Transformer都可以脱离Control Hub单独使用，另外Data Collector和Transformer都提供数据流设计、校验、预览、发布以及数据流监控功能，并提供功能完善的RESTful接口、各类组件的实现接口。

本实践课程主要是针对Data Collector产品的讲解与设计，当然在课程的可能会提到或讲解其他产品。

## 重头戏StreamSets Data Collector（SDC）

那么问题来了？Data Collector到底是啥软件，到底什么时候用到它呢,它给我数据开发为会带来什么样的影响呢？下面一一道来。

介绍Data Collector之前我们先介绍它的子软件Data Collector Edge：

StreamSets Data Collector Edge（ SDC Edge）是一种轻量级的执行代理，可在资源有限的边缘设备上运行数据流，支持多种操作系统及硬件（包括linux、win、mac、Arm等），没有UI界面，但可借助Data Collector设计、发布工作流。可以使用 SDC Edge从边缘设备读取数据或从另一个管道接收数据，然后对该数据进行简单加工处理以及可控制边缘设备。

Data Collector简单的来说就是一款功能强大操作简单的数据ETL软件，但有不仅仅是ETL软件，它还可以通过它的子工具SDC Edge收集边缘设备的数据，自身还可以实现数据爬虫、数据迁移、微服务、流式处理、集群处理等等，提供几十种或上百种组件（并具有完备API，用户可根据业务需求扩充组件）满足各种数据处理业务需求，并拥有面向各种第三方数据系统（像hadoop、spark、hive、mysql、Cassandra、aws、ElasticSearch等等）的组件与之交互，

面临数据业务的复杂性、数据的多源化，面对不同的业务需要，当前的做法主要是不断的造轮子，但是懂业务的不一定董开发，董开发的不一定懂业务，另外加上企业发展需求人才的匮乏，逼迫我们不得不加班熬夜甚至996。作为企业自身,业务扩展十分迅速，线上产品迭代很快，需要迅速实现数据流。StreamSets Data Collector将是你们最佳的选择。

将StreamSets Data Collector像管道一样用于数据流。在整个企业数据拓扑中，您拥有在流向目的地的过程中需要移动，收集和处理的数据流。Data Collector提供了流中跃点之间的关键连接。

为了解决您的提取需求，您可以使用单个Data Collector运行一个或多个管道。或者，您可以安装一系列Data Collector来跨企业数据拓扑流式传输数据

不多说，先来看看它StreamSets Data Collector真身：

![img](https://img-blog.csdnimg.cn/20200425120415464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

# StreamSets实战之路（二）-环境篇- StreamSets安装与配置

## 1.StreamSets DC安装与配置

StreamSets官方网站提供了多种安装方式，包括：手动解压Tarball包安装、通过RPM软件包安装、通过Cloudera Manager安装、通过Docker安装，除了这几种安装方式，还提供了在云服务商上安装或者在MapR集群上安装。当然，你可以安装包括所有阶段库的完整版本的Data Collector，或者，可以安装Data Collector的core版本以仅安装要使用的阶段库，core版本安装使Data Collector可以使用更少的磁盘空间。

### （1）安装需求：

![image-20210903140431168](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210903140431168.png)

*因为StreamSets DC是java语言开发的，因此java运行环境必须要安装与配置。

### （2）设置系统的文件打开数：

Linux操作系默认文件打开数为1024，我们需要将文件的打开数设置为32768或者更大一些。

首先可以通过以下命令查看操作系统的文件打开数：

ulimit -n
文件打开数针对不同的操作系统有不同的配置方式，大家可以参照以下解决方案: https://access.redhat.com/solutions/61334。下面我们针对CentOS Linux做文件打开数的配置：

切换至root用户，使用 ulimit –HSn 32768 命令修改（此时可利用 ulimit –n 查看，发现文件打开数为32768，但是这只能暂时修改，当退出时，文件打开数会变成默认值）
修改配置文件 /etc/security/limits.conf，在文件后加上：
* soft nofile  32768
       * hard nofile  32768

### 1.1手动解压Tarball包安装

可以安装完整或者核心的Data Collector tarball并在所有受支持的操作系统上手动启动。

手动启动Data Collector时，Data Collector 将以运行启动命令时登录到命令提示符下的系统用户帐户身份运行。

（1）通过下面链接下载完整或核心的Data Collector tarball：

https://streamsets.com/products/dataops-platform/data-collector/download/。

（2）将使用以下命令压缩包解压到所需位置：

tar zxf streamsets-datacollector-all-3.15.0.tgz
解压后会看到下图展示的文件：

![img](https://img-blog.csdnimg.cn/2020042522004721.png)

介绍几个重要的目录： 

bin目录：是Streamset DC运行脚本目录

etc目录：是Streamset DC默认的配置文件目录，包括系统配置、权限配置、邮件配置、日志配置等；

data目录：是Streamset DC默认的数据目录，用于存储你设计的数据流等；

log目录：是Streamset DC默认的日志目录，包括GC日志和系统日志；

libexec目录：是Streamset DC默认的运行时环境配置目录

streamsets-libs目录：是Streamset DC默认的系统自带组件的目录

user-libs目录：是Streamset DC放置用户自定义开发组件的目录

edge-binaries目录：是Streamset DC存放Streamsets DC Edge的各种类型的安装包。

（3）使用以下命令启动运行Data Collector：

bin/streamsets dc
或者，使用以下命令在后台运行Data Collector：

nohup bin/streamsets dc >/dev/null 2>&1 &
（4）要访问Data Collector UI，请在浏览器的地址栏中输入以下URL：

http://Ip:18630/

登录默认用户密码为：admin/admin。

 

提示：对于生产环境，请在启动Data Collector之前配置用于存储配置文件，数据文件，日志文件和资源文件的目录，以使它们位于$ SDC_DIST（压缩包的位置）和基本Data Collector运行时目录之外。

对于开发或测试环境，可以使用$ SDC_DIST运行时目录中的默认位置。当然，对于所有环境，建议大家都将配置文件，数据文件，日志文件和资源文件的目录设置到$ SDC_DIST目录之外的目录，并确保文件目录的读写权限。

在$ SDC_DIST运行时目录之外创建用于配置，数据，日志和资源文件的目录。

在$ SDC_DIST / libexec / sdc-env.sh文件中，将以下环境变量设置为新创建的目录：

SDC_CONF- 数据收集器配置目录。

SDC_DATA- 数据收集器目录，用于管道状态和配置信息。

SDC_LOG- 日志的数据收集器目录。

SDC_RESOURCES- 运行时资源文件的数据收集器目录。

将所有文件从$ SDC_DIST / etc复制到新创建的$ SDC_CONF目录。

### 1.2 RPM软件包安装

Data Collector RPM软件包安装主要是将其作为CentOS，Oracle Linux或Red Hat Enterprise Linux上的服务启动。

使用RPM软件包安装，Data Collector使用默认目录并作为默认系统用户和组运行。默认的系统用户和组名为sdc。如果计算机上不存在sdc用户和sdc组，则安装将为您创建用户和组，并为其分配下一个可用的用户ID和组ID。

提示：要为sdc用户和组使用特定的ID，请在安装前创建用户和组，并指定要使用的ID。例如，如果要在多台计算机上安装Data Collector，则可能要在安装之前创建系统用户和组，以确保用户ID和组ID在所有计算机上都一致。安装Data Collector作为服务安装时需要root特权。

（1）通过下面链接下载Data Collector RPM软件包：

https://streamsets.com/products/dataops-platform/data-collector/download/ 。

请下载适用于您的操作系统的RPM软件包：

对于CentOS 6，Oracle Linux 6或Red Hat Enterprise Linux 6，请下载RPM EL6软件包。

对于CentOS 7，Oracle Linux 7或Red Hat Enterprise Linux 7，请下载RPM EL7软件包。

（2）使用以下命令将文件解压到所需位置：

tar xf streamsets-datacollector-<version>-<operating_system>-all-rpms.tar
例如，要在CentOS 7上解压缩版本3.15.0，请使用以下命令：

tar xf streamsets-datacollector-3.15.0-el7-all-rpms.tar
（3）使用以下命令安装完整的Data Collector RPM软件包：

yum localinstall streamsets*.rpm
（4）将Data Collector作为服务启动，请对您的操作系统使用所需的命令：

对于CentOS 6，Oracle Linux 6或Red Hat Enterprise Linux 6，请使用：

service sdc start
对于CentOS 7，Oracle Linux 7或Red Hat Enterprise Linux 7，请使用：

systemctl start sdc
（5）要访问Data Collector UI，请在浏览器的地址栏中输入以下URL：

http://Ip:18630/

登录默认用户密码为：admin/admin。

### 1.3 Docker安装

 Docker方式的安装比较简单，环境和默认配置已经在docker镜像中设置，当然安装前你的操作系统上已经安装了docker软件。

（1）可以用以下命令启动Streamset DC：

docker run --restart on-failure -p 18630:18630 -d --name streamsets-dc streamsets/datacollector dc
（2）启动过后，要访问Data Collector UI，请在浏览器的地址栏中输入以下URL：

http://Ip:18630/

登录默认用户密码为：admin/admin。

 

当然为了防止你在streamsets中配置数据流丢失，最好将数据文件映射到外部文件系统上：

首先在宿主机上创建一个目录：

```shell
mkdir -p /opt/streamset-datas

docker run --restart on-failure –v /opt/streamset-datas:/data -p 18630:18630 -d --name streamsets-dc streamsets/datacollector dc
```

想了解更多的docker安装信息，请参照：https://hub.docker.com/r/streamsets/datacollector/。

![img](https://img-blog.csdnimg.cn/20200425220228289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

## **2.StreamSets DC Edge安装与配置**

Edge的安装官方也提供两种安装方式：手动解压缩包安装和docker安装。

Edge是一个轻量级的代理工具，因此占用非常少量的系统资源。使用Go语言开发现在支持操作系统包括：

Darwin AMD64

Linux AMD64，ARMv6，ARMv7和ARMv8

Windows AMD64

当然可以从SDC Edge开源代码在任何其他操作系统构建自己的程序包。详细请参阅：https://github.com/streamsets/datacollector-edge/blob/master/BUILD.md

你现在可以通过两种方式下载Edge压缩包：

从您安装的StreamSets DC UI界面中下载或从StreamSets DC安装的目录中的edge-binaries目录获取，当然您也可以从streamsets 官方网址下载（https://archives.streamsets.com/index.html）。

若是您要从StreamSets DC UI界面中下载的话，需要您创建一个Edge类型的数据流，例如下图：

![img](https://img-blog.csdnimg.cn/20200425220301635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

### 2.1手动解压缩包安装

（1）解压下载或获取到的压缩包至相应目录，利用管理员身份使用以下命令将Edge安装到您的系统中：

```shell
bin/edge -service install
```

利用以下命令启动Edge：

```shell
bin/edge -service start
```

利用以下命令重启Edge：

```shell
bin/edge -service restart
```

利用以下命令关闭Edge：

```shell
bin/edge -service stop
```

利用以下命令查看Edge的运行状态：

```shell
bin/edge -service status
```

利用以下命令卸载Edge：

```shell
bin/edge -service uninstall
```

### 2.2 docker安装

 使用以下命令启动Edge：

```shell
docker run --publish 18633:18633 --name edge --rm streamsets/datacollector-edge
```

至于怎么设计edge数据流，怎么发布Edge数据流到响应的edge上，后续课程或给大家介绍到。


# StreamSets实战之路（三）-环境篇- StreamSet源码查看与编译

## 1.StreamSets DC源码查看与编译

StreamSets DC源码clone目前只能在linux和Mac操作系统上进行，在Window有出现很多问题。在这只介绍在linux上进行查看源码和编译源码。

### 1.1源码编译环境准备

Git 1.9+

Oracle JDK 8

Maven 3.3.9+

Docker 1.10+ (做集成测试的时候是需要的，编译时不测试不需要.)

Node 0.10.32+1

npm

bower 1.8.2 (macOS, npm -g install bower : Linux, sudo npm -g install bower)

grunt-cli (macOS, npm -g install grunt-cli : Linux, sudo npm -g install grunt-cli)

md5sum (macOS, brew install md5sha1sum)

 

若是只想查看源码，只需要安装git、jdk8、maven这个。开发工具这里我们选用IDEA。

### 1.2 查看源码

(1)克隆源代码

git clone https://github.com/streamsets/datacollector.git

网速慢的可能要等一会。下载完后，您可以通过：

git branch -a 或 git tag
查看现有分支和版本，您可以指定某个分支或版本查看，通过：

git checkout <version>
切换分支或版本。

(2)使用IDEA查看源码

首次打开，可能要很慢，因为打开的时候，会自动去下载很多依赖包，等待下载完后，你会看到：

![img](https://img-blog.csdnimg.cn/20200428231056871.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

这里介绍几个主要的模块：

basic-lib: 是streamsets DC的基础组件包（包含origin、processor、destination等），streamsets DC 中一些常见的组件都在这里，在核心版本和完整版本中这个基础组件包都有；

datacollector-ui:是streamsets DC的前端Web-ui模块，主要采用angular.js框架；

common*和container：是streamsets DC很多模块的依赖包；

cdh*：是cdh相关生态的组件；

hdp*：是cdh相关生态的组件；

tensorflow-lib：是深度学习框架组件；

spark*：与spark streaming 集群流式计算；

jdbc-lib：jdbc相关组件；

kafka、ES、redis、hadoop、…

### 1.3编译源码

建议先进行一遍，这样会把一些相关的依赖给下载下来：

```shell
mvn clean install –DskipTests
```

用于这个工程包含的模块比较多，单线程编译耗时较长，这里我们给他提提速（资源够才行）：

```shell
mvn –T 8 clean install –DskipTests
```

   这里用到8个线程，最后执行：

mvn –T 8 clean package -Drelease -DskipTests -P-rpm
执行完后会在release/target/目录下，生成很多包，其中就包含了，完整版、核心版、RPM版的安装包。

## 2.StreamSets DC Edge源码查看与编译

StreamSets DC Edge源码可以在更多操作系统或硬件上编译。在这只介绍在linux上进行查看源码和编译源码。

### 2.1 环境准备

JDK8

Git 1.9

### 2.2 查看源码

(1)克隆源代码

因为streamsets DC Edge 是Go语言编写的，你首先要设置一个$GOPATH环境变量(不需要安装Go语言环境，会自动下载自动安装)，然后新建一个工程目录例如：

```shell
mkdir –p $GOPATH/src/github.com/streamsets

cd $GOPATH/src/github.com/streamsets

git clone https://github.com/streamsets/datacollector-edge.git
```

下载完后，您可以通过：

```shell
git branch -a 或 git tag
```

查看现有分支和版本，您可以指定某个分支或版本查看，通过：

```shell
git checkout <version>
```

切换分支或版本。

(2)查看源码

![img](https://img-blog.csdnimg.cn/20200428231114659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

主要的包就是stages这个目录了，这里包含了很多组件，当然这些组件在streamsets DC上是有对应的，当你在Edge 上开发一个组件时，DC上也是要开发一个对应的组件，后期在《StreamSets实战之路-开发篇》会讲到。

### 2.3 编译代码

工程编译主要通过gradle进行，但是你也不用安装gradle，会自动下载自动安装。

使用以下命令编译生成所有平台的安装包：

```shell
./gradlew clean dist 
```

 
注意：编译中遇到的一个比较棘手的问题就是，很多依赖包时需要翻墙的，但是在linux上翻墙也是很痛苦的。庆幸的是，在GitHub上都有依赖包镜像，但是这些包需要你手动下载，并设置修改包路径。后期，要是有需要，我可以把我这边的环境打一个包给大家，或者遇到问题及时交流。

# StreamSets实战之路（四）-环境篇- StreamSet工作平台介绍

（1）首次进入工作平台（默认用户名密码：admin/admin）：

![img](https://img-blog.csdnimg.cn/20200429205412430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

 这时我们点击‘CREATE NEW PIPELINE’按钮新建一个数据流：

![img](https://img-blog.csdnimg.cn/20200429205434206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

会看进去数据流设计界面：

![img](https://img-blog.csdnimg.cn/20200429205454107.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

数据流设计界面，根据不同的功能可以分为5大区域：

数据流设计区域：该区域是数据流设计区域，通过拖拽组件选择区域的组件与连接操作设计数据流。

配置区域：该区域在设计阶段：主要配置整个数据流、配置每个组件、设置一个运行时数据规则；在预览阶段：查看对比各个组件的输入输出；在运行阶段，监控整个数据流、各个组件的运行状态、处理效率、出错率等；在快照阶段，可以查看数据流在某一时刻的快照信息。

组件选择区域：该区域放置了很多已经开发好的组件，包括origin类、processor类、destination类等组件，供数据流设计使用。

数据流控制区域：该区域主要用来控制设计好的数据流，比如：启动、校验、预览、分享、复制、删除、停止、导入、导出、数据快照、查看日志等等。

StreamSets DC操作区域：该区域主要是操作Streamsets DC，比如：安装组件、系统启停、消息通知、系统监控、系统日志、权限控制、系统的一些配置（部分）、系统提供的RESTful接口使用等等。

 

这时候我们可以回到主页面-数据流管理界面：

![img](https://img-blog.csdnimg.cn/20200429205507263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

当然除了上述的界面还有其他的：

系统监控界面：

![img](https://img-blog.csdnimg.cn/20200429205525982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

日志界面：

![img](https://img-blog.csdnimg.cn/20200429205544996.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

RESTful API 界面：

![img](https://img-blog.csdnimg.cn/20200429205602196.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

包管理界面：

![img](https://img-blog.csdnimg.cn/20200429205617437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

系统配置信息界面：

![img](https://img-blog.csdnimg.cn/20200429205630609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)






# StreamSets实战之路（五）-基础篇- StreamSets开启第一个数据流

主要介绍第一个工作流的创建、预览、启动过程，这条数据流将本地文件中的数据进行处理，最终将处理过的数据存放到本地磁盘上(streamsets 运行在CentOS7上)。

1 数据准备阶段
  在本地磁盘的/tmp目录下新建一个inputdatas 目录，并将我们准备好的数据放置到该目录下，这里我们准备了一个json文件，再在/tmp目录下新建一个outputdatas用于存放处理后的数据。

2 数据流设计阶段
（1）新建一个数据流，填写数据流名字、描述信息、定义一个标签信息。选择数据流类型为Data Collect Pipeline，点击Save按钮。

![img](https://img-blog.csdnimg.cn/20200510205138891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

  （2）完成第一步会进入数据流设计界面，如下图：

![img](https://img-blog.csdnimg.cn/20200510205529422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（3）首先从下图中的两个区域选择一个数据源插件，用于将外部数据源中的数据输入到Streamsets  中，这里我们选择一个简单的文件目录插件,并配置该插件，设置读取的文件目录、文件类型、输入到streamsets 中的文件格式，其他的配置参数先默认。

![img](https://img-blog.csdnimg.cn/20200510205627163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200510205646664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200510205704240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（4）从组件区，选择一个数据处理插件，这里选择JavaScript插件，并编写js脚本（将字段plugins中的每个记录），将上一个插件的输出和该插件的输入进行连接：

![img](https://img-blog.csdnimg.cn/20200510205735624.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（5）再选择一个数据处理插件，将记录中不包含data字段的数据过滤掉，并将name字段为host的记录保留下来，最后将上一个插件的输出和该插件的输入进行连接：

![img](https://img-blog.csdnimg.cn/20200510205803179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（6）将满足条件1的数据记录进一步处理，不满足条件1的数据记录扔了；从组件选择区选择一个记录过滤插件和一个垃圾桶插件，并将上一个插件的1输出连接记录过滤插件，2输出连接垃圾桶插件；

![img](https://img-blog.csdnimg.cn/20200510205839628.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配置记录过滤插件，保留记录中的三个字段：

![img](https://img-blog.csdnimg.cn/20200510210023892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（7）选择一个记录展开字段，将记录中/data字段的数据平铺

![img](https://img-blog.csdnimg.cn/20200510210138326.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（8）再选择一个表达式插件，进行缺失值补充：

![img](https://img-blog.csdnimg.cn/20200510210207699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（9）从组件选择区将数据输出组件（这里选择本地文件系统插件）拖拉到数据流设计区，并配置数据处理目录和设置好数据输出格式，最后将处理好的数据存储在本地磁盘上：

![img](https://img-blog.csdnimg.cn/20200510210303522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

3 数据流配置、验证和预览

（1）配置数据流,先选择数据流配置界面

![img](https://img-blog.csdnimg.cn/20200510210329438.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

在工作流配置区配置工作流，可以先默认配置，若是有别的需求，可以根据需求配置数据流。

![img](https://img-blog.csdnimg.cn/20200510210355169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（2）数据流验证和预览：点击小红框处，验证数据流

![img](https://img-blog.csdnimg.cn/20200510210434240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

数据流验证成功后，点击数据流控制区的‘眼睛’按钮进行数据预览，可以进行单个组件的数据输入和输出和多个组件的数据对比。

![img](https://img-blog.csdnimg.cn/20200510210505999.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200510210521499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

4 数据流启动与监控

点击‘start’按钮启动工作流

![img](https://img-blog.csdnimg.cn/20200510210643330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

启动数据流后可监控整数据流的数据处理效率，或监控单独组件的数据处理效率。![img](https://img-blog.csdnimg.cn/20200510210702646.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200510210719548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)



# StreamSets实战之路（六）-基础篇- StreamSets-origin类组件使用

主要介绍StreamSets-origin类组件有哪些、分类、主要用途以及使用方法。

Origin类组件主要包含以下：

Amazon S3

Amazon SQS Consumer

Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen2

Azure IoT/Event Hub Consumer

CoAP Server

Cron Scheduler // 任务调度组件，用于调度数据流

Directory // 文件目录组件，用于从文件目录下读取数据

Elasticsearch // ES源组件,用于从ES中读取数据

File Tail // 文件源插件，用于从文件尾源将读取数据行

Google BigQuery

Google Cloud Storage

Google Pub/Sub Subscriber

Groovy Scripting // Groovy脚本组件，自定义groovy脚本，功能强大

gRPC Client // Google RPC客户端组件，用于从gRPC服务器获取数据

Hadoop FS // hadoop 文件系统组件，用于从HDFS中读取数据，可用集群模式下读取数据

Hadoop FS Standalone // hadoop 文件系统组件，用于从HDFS中读取数据

HTTP Client // Http客户端组件，用于从Http服务器获取数据

HTTP Server // Http服务器组件，用于接收HTTP客户端的数据

JavaScript Scripting  // JavaScript脚本组件，自定义JavaScript脚本，功能强大

JDBC Multitable Consumer // JDBC多线程数据源组件，用于以JDBC方式读取数据，适用于可通过JDBC方式连接的数据库（例如：mysql、oracle等），该组件可用于多线程模式

JDBC Query Consumer  // JDBC数据源组件，用于以JDBC方式读取数据，适用于可通过JDBC方式连接的数据库（例如：mysql、oracle等）

JMS Consumer // JMS数据源组件，用于从JMS服务中消费数据

Jython Scripting // Jython脚本组件，自定义Jython脚本，功能强大

Kafka Consumer // Kafka数据源组件，用于从Kafka中消费数据

Kafka Multitopic Consumer // // Kafka多Topic数据源组件，用于从Kafka中消费数据，可用于指定多个topic进行消费，多线程消费

Kinesis Consumer // Kinesis数据源组件，用于从Kinesis中消费数据

MapR DB CDC

MapR DB JSON

MapR FS

MapR FS Standalone

MapR Multitopic Streams Consumer

MapR Streams Consumer

MongoDB // MongoDB数据源组件，用于从MongoDB中读取数据

MongoDB Oplog // MongoDB Oplog数据源组件，用于从MongoDB Oplog中读取数据

MQTT Subscriber // MQTT数据源组件，用于从MQTT中消费数据

MySQL Binary Log // MySQL Binary Log数据源组件，用于从MySQL Binary Log中读取数据

NiFi HTTP Server // NiFi HTTP 服务数据源组件，用于接收NiFi HTTP Client 发送的数据

Omniture

OPC UA Client

Oracle Bulkload // Oracle 批量加载插件，用于从多个Oracle表读取所有可用数据，可用于多线程

Oracle CDC Client

PostgreSQL CDC Client

Pulsar Consumer // Pulsar数据源组件，用于从Pulsar中消费数据

RabbitMQ Consumer //RabbitMQ数据源组件，用于从RabbitMQ中消费数据

Redis Consumer // Redis数据源组件，用于从Redis中读取数据

REST Service // REST 服务组件，用于微服务工作流设置，接收HTTP 请求

Salesforce

SDC RPC

SFTP/FTP/FTPS Client // FTP 客户端组件，用于从FTP服务中获取数据

SQL Server 2019 BDC Multitable Consumer

SQL Server CDC Client

SQL Server Change Tracking

Start Pipeline // 数据流启动组件，用于启动数据流

System Metrics // 系统指标源组件，用于从linux系统上获取CPU、内存等指标信息

TCP Server // TCP 服务组件，用于接收TCP Client发送的数据

Teradata Consumer

UDP Multithreaded Source //UDP多线程服务组件，用于接收UDP Client发送的数据

UDP Source // UDP 服务组件，用于接收UDP Client发送的数据

WebSocket Client // WebSocket客户端组件，用于从WebSocket服务获取数据

WebSocket Server // WebSocket服务组件，用于接收WebSocket Client发送的数据

Windows Event Log // Windows 事件日志组件，用于从Windows系统中获取事件日志，该组件尽可用于Edge数据流

  

使用方法：

![img](https://img-blog.csdnimg.cn/20200517114123925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

注意：origin组件一个工作流只能存在一个origin组件。

# StreamSets实战之路（七）-基础篇- StreamSets-Processor类组件使用

主要介绍StreamSets-Processor类组件有哪些、分类、主要用途以及使用方法。

Processor类组件主要包含以下：

Base64 Field Decoder // base64 解码组件

Base64 Field Encoder  // base64 编码组件

Control Hub API // Control Hub 接口调用组件

Couchbase Lookup // Couchbase查询组件，用于从Couchbase系统中读取数据

Data Generator // 数据序列化组件，将Avro、json、protobuf、text、xml等格式的数据序列成bytearray或string

Data Parser // 数据反序列化组件，将bytearray或string数据反序列成Avro、json、protobuf、text、xml等格式的数据

Databricks ML Evaluator // Databricks机器学习组件，使用Databricks机器模型进行数据分析

Delay // 延迟处理组件，用于数据延时处理

Encrypt and Decrypt Fields // 加解密组件，支持多种加解密算法

Expression Evaluator // 表达式组件，可用该组件添加或修改记录标题属性和字段属性

Field Flattener // 数据平铺组件，可以展平整个记录以生成没有嵌套字段的记录

Field Hasher // 哈希组件，可用于计算数据的哈希值，支持多种哈希算法

Field Mapper // 数据映射组件，可用于将表达式映射到一组字段，以更改字段路径，字段名称或字段值

Field Masker // 数据打码组件，可用于将敏感的数据进行打码

Field Merger // 数据合并组件，将List或Map类型的记录中的一个或多个字段合并到记录中的其他路径

Field Order // 数据排序组件，将List或Map类型的记录中的字段进行排序

Field Pivoter // 数据移位组件

Field Remover // 字段删除组件，用于保留或删除记录中的某些字段

Field Renamer // 重命名组件，用于重命名记录中字段的key

Field Replacer // 数据替换组件，用于填充或替换记录中的缺失值

Field Splitter // 字段切割组件，用于将数据按某一分隔符进行切割

Field Type Converter // 类型转化组件，用于数据的类型转化

Field Zip // 拉锁组件，用于将两个数组进行关联

Geo IP // Ip解析组件，用于将ip解析成对应的经纬度、地理信息等数据信息

Groovy Evaluator // Groovy脚本组件，用于自定义Groovy脚本，根据需求编写一些代码实现一个数据处理任务，功能强大

HBase Lookup // HBase 数据查询组件，用于从HBASE查询数据

Hive Metadata // Hive 元数据组件，与Hive Metastore目标以及Hadoop FS或MapR FS目标配合使用，作为Hive漂移同步解决方案的一部分

HTTP Client // Http 客户端组件，用于从http服务中获取获取数据

HTTP Router // Http 路由组件，根据http 请求方式（post put get）和请求路径进行分支路由

JavaScript Evaluator // JavaScript脚本组件，用于自定义JavaScript脚本，根据需求编写一些代码实现一个数据处理任务，功能强大

JDBC Lookup // JDBC 数据查询组件，用于JDBC从数据库中查询数据，适用于通过JDBC方式连接的数据库（Mysql等）的查询

JDBC Tee // JDBC Tee 组件，使用JDBC连接将数据写入MySQL或PostgreSQL数据库表，然后将生成的数据库列值传递给字段。使用JDBC Tee处理器将部分或全部记录字段写入数据库表，然后用其他数据丰富记录

JSON Generator // JSON 序列化组件，用于将数据记录序列化成JSON字符串

JSON Parser // JSON 反序列化组件，用于将JSON字符串数据反序列化成Java对象数据

Jython Evaluator  // Jython脚本组件，用于自定义Jython脚本，根据需求编写一些代码实现一个数据处理任务，功能强大

Kudu Lookup // Kudu 查询组件，用于从Kudu 系统中读取数据

Log Parser // 日志解析组件，支持多种日志格式的的解析，用于将具有一定格式的日志数据，解析成系统平台可处理的结构化格式数据

MLeap Evaluator // MLeap 数据分析组件，使用存储在MLeap捆绑软件中的机器学习模型来生成评估，评分或数据分类

MongoDB Lookup // MongoDB 数据查询组件，用于从MongoDB中查询数据

PMML Evaluator // PMML数据分析组件,使用以预测模型标记语言（PMML）格式存储的机器学习模型来生成数据的预测或分类

PostgreSQL Metadata //PostgreSQL元数据组件，确定其中每个记录应写入PostgreSQL的表，记录结构对表结构进行比较，然后根据需要创建或改变的表

Record Deduplicator // 记录重复数据删除组件，评估记录中是否有重复数据，并将数据路由到两个流中-一个流用于唯一记录，一个流用于重复记录。使用记录重复数据删除器丢弃重复数据或通过不同的处理逻辑路由重复数据

Redis Lookup // Redis数据查询组件，用于从Redis中查询数据

Salesforce Lookup // Salesforce数据查询组件，用于从Salesforce中查询数据

Schema Generator // Schema 生成组件，基于记录的结构生成模式，并将该模式写入记录头属性。用于生成Avro Schema

Spark Evaluator // spark 数据处理组件，用于将平台与spark关联实现数据处理的分布式处理

SQL Parser // SQL 解析组件

Start Job // 作业启动组件，需要与Controler Hub 配合使用

Start Pipeline // 数据流启动组件，用于启动指定的数据流

Static Lookup // 静态数据查询组件，执行存储在本地内存中的键/值对的查找，并将查找值传递给字段。使用静态查找将字符串值存储在内存中，管道可以在运行时查找这些值，以用其他数据丰富记录

Stream Selector // 数据分选组件，用于通过设置条件，来将数据分选不同分支进行处理

TensorFlow Evaluator // TensorFlow 数据分析组件，通过TensorFlow训练的数据模型，并模型配置到指定目录下，在系统平台上使用，实现数据分析功能

Whole File Transformer // 全文件转换组件，用于全文件目录或文件的快速拷贝或转换

Windowing Aggregator // 窗口聚合组件，用于指定一定窗口大小实现窗口内部数据的聚合操作，支持滚动和滑动窗口

XML Flattener // XML 平铺组件，用于XML数据的展平，可以展平整个记录以生成没有嵌套字段的记录

XML Parser // XML 解析组件，用于将XML数据进行解析，转换成系统平台易处理的数据格式，类似JSON Parser

  

使用方法：

![img](https://img-blog.csdnimg.cn/20200518093815542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)



# StreamSets实战之路（八）-基础篇- StreamSets-Destination类组件使用

主要介绍StreamSets-Destination类组件有哪些、分类、主要用途以及使用方法。

Destination类组件主要是将在StreamSets DC上的数据输出落地到指定的存储服务或其他服务等

Destination类组件主要包含以下：

Aerospike // Aerospike数据输出组件，将数据写到Aerospike（分布式KV库） 库中

Amazon S3 // S3数据输出组件，将数据写到S3上

Cassandra // Cassandra数据输出组件，将数据写到Cassandra库中

CoAP Client // CoAP 客户端，使用CoAP客户端将数据写到支持CoAP协议的服务端

Couchbase // Couchbase数据输出组件，将数据写到Couchbase库中

Databricks Delta Lake // Databricks 数据湖数据输出组件

Elasticsearch // ES数据组件

Einstein Analytics // 将数据输出上传到Einstein Analytics

Flume // 将数据输出到flume数据源中

Google BigQuery //

Google Bigtable

Google Cloud Storage

Google Pub/Sub Publisher

GPSS Producer

Hadoop FS // 将数据输出存储到HDFS上

HBase //将数据输出存储到HBase上

Hive Metastore // Hive元数据处理器和Hadoop FS或MapR FS目标一起使用，作为Hive漂移同步解决方案的一部分

Hive Streaming // 将数据写到Hive中使用ORC (Optimized Row Columnar)数据格式

HTTP Client // HTTP 客户端，将数据写入到支持HTTP协议的数据服务端

InfluxDB // 将数据写入到InfluxDB（时序数据库）

JDBC Producer // 以JDBC的方式写入到支持JDBC连接的数据库中

JMS Producer //将数据写入到JMS消息队列中

Kafka Producer // 将数据写入到Kafka指定的消息队列中

Kinesis Firehose

Kinesis Producer

KineticaDB

Kudu // 将数据输出到Kudu

Local FS // 将数据输出到本地磁盘

MapR DB

MapR DB JSON

MapR FS

MapR Streams Producer

MemSQL Fast Loader

MongoDB // 将数据输出到MongoDB

MQTT Publisher // 将数据输出到MQTT消息队列中

Named Pipe // 将数据输出到UNIX命名管道

Pulsar Producer // 将数据输出到Pulsar消息队列中

RabbitMQ Producer // 将数据输出到RabbitMQ消息队列中

Redis // 将数据输出到Redis

Salesforce // 将数据输出到Salesforce

SDC RPC //将数据传递到一个或多个SDC RPC源

Send Response to Origin // 用在微服务工作流中，将数据返回到用户端

SFTP/FTP/FTPS Client // Ftp客户端，将数据输出到ftp上

Snowflake // 将数据输出到Snowflake

Solr // 将数据输出到solr

Splunk // 将数据输出到Splunk

SQL Server 2019 BDC Bulk Loader // 将数据输出到SQL Server

Syslog //将syslog消息写入Syslog服务器

To Error //将数据流发送到管道错误处理

Trash // 垃圾桶组件，用于将数据丢弃到

WebSocket Client  // WebSocket客户端，通过该组件将数据发送到WebSocket服务端

使用方法：

![img](https://img-blog.csdnimg.cn/20200529213824838.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)






# StreamSets实战之路（九）-基础篇- StreamSets-Executor类组件使用

主要介绍StreamSets-Executor类组件有哪些、分类、主要用途以及使用方法。

    Executor类组件主要在收到事件时会触发任务。

Executor类组件主要包含以下：

ADLS Gen1 File Metadata // 收到事件后，更改文件元数据，创建一个空文件或删除Azure Data Lake Storage Gen1中的文件或目录。

ADLS Gen2 File Metadata // 收到事件后，更改文件元数据，创建一个空文件或删除Azure Data Lake Storage Gen2中的文件或目录。

Amazon S3 // 为指定的内容创建新的Amazon S3对象，复制存储桶中的对象，或将标签添加到现有的Amazon S3对象。

Databricks Job Launcher // 收到事件记录后启动指定的Databricks作业。

Databricks Query // 收到事件后在Databricks上运行Spark SQL查询。

Email // 在收到事件后向配置的收件人发送自定义电子邮件。

HDFS File Metadata // 收到事件后，更改文件元数据，创建空文件或删除HDFS或本地文件系统中的文件或目录。

Hive Query // 收到事件记录后运行用户定义的Hive或Impala查询。

JDBC Query // 收到事件记录后运行用户定义的SQL查询。

MapR FS File Metadata // 收到事件后，更改文件元数据，创建空文件或删除MapR FS中的文件或目录。

MapReduce // 收到事件记录后启动指定的MapReduce作业。

Pipeline Finisher // 收到事件记录后停止并将管道转换为Finished状态。

SFTP/FTP/FTPS Client // 从SFTP，FTP或FTPS服务器移动或删除文件。

Shell // 在接收到事件记录执行shell脚本。

Spark // 收到事件记录后启动指定的Spark应用程序。

  

使用方法：

该数据流实现当文件中的数据读完后或文件没有最新数据时，就将该数据流停止。

（1）首先可以创建一个数据流或使用我们第五节我创建好的数据流，这里我们通过复制数据流将第五节创建好的数据流复制一个出来：

![img](https://img-blog.csdnimg.cn/20200626183727793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626183753961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（2）点击origin的目录插件，配置该插件产生事件

![img](https://img-blog.csdnimg.cn/2020062618383118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（3）从Executor组件框中将Pipeline Finisher拖拉到设计面板，并从processor组件框中将选择组件拖拽到面板，并在条件1中配置为${record:eventType() == 'no-more-data'}，将条件1的输出连接到Pipeline Finisher，条件2的数据丢弃

 ![img](https://img-blog.csdnimg.cn/2020062618385190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626183922908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（4）点击启动运行，会发现当数据读完时，数据流会自动停止。

![img](https://img-blog.csdnimg.cn/20200626183948423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)	






# StreamSets实战之路（十）-基础篇- StreamSets-数据流开发-微服务数据流设计

主要介绍StreamSets中微服务数据流设计，以mysql的CRUD操作为例，来设计微服务数据流。

  

微服务在当前已经是非常流行的技术，将大型或复杂的系统进行模块拆分成功能单一、组织灵活的微服务，从而降低系统的耦合性，提高系统的灵活性、高可用性、运行高效性等。为此，Streamsets也提供了微服务简单、快速的开发方案，在设计微服务时，只需通过现有的插件任意组合就可以设计出简单或复杂的微服务。

 

（1）创建一个微服务数据流

![img](https://img-blog.csdnimg.cn/20200626184723512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（2）会生成一个微服务数据流模板

![img](https://img-blog.csdnimg.cn/20200626184741381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（3）下面对生成的模板进行改进，并运行

![img](https://img-blog.csdnimg.cn/20200626184803484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)



 ![img](https://img-blog.csdnimg.cn/20200626184815969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

该数据流主要的关键组件有REST Service、HTTP Router、JDBC Lookup、JDBC Tee、Send Response to Origin。

REST Service组件：

该组件是微服务的关键组件，让微服务对外提供一个REST Ful API，需要配置端口、APP ID、线程数、最大请求数据大小等：

![img](https://img-blog.csdnimg.cn/20200626184838945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

HTTP Router组件：

该组件设置Http的路由，根据前端请求的不同路径进行路由。

![img](https://img-blog.csdnimg.cn/20200626184857511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

JDBC Lookup组件：

该组件用于数据查询，主要设置jdbc的连接串、mysql的连接用户名和密码、设置查询语句、设置结果返回的方式

![img](https://img-blog.csdnimg.cn/20200626184911546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

使用jdbc需要上传相关数据库的连接驱动，目前支持mysql、oracle等等

![img](https://img-blog.csdnimg.cn/20200626184925304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

生成查询语句sql

![img](https://img-blog.csdnimg.cn/20200626184940322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

JDBC Tee组件：

该组件用于数据增加、修改、删除，主要设置jdbc的连接串、mysql的连接用户名和密码、数据操作类型

![img](https://img-blog.csdnimg.cn/20200626184956695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

Send Response to Origin组件：

将数据返回到Origin组件，并设置状态码为200。

![img](https://img-blog.csdnimg.cn/20200626185013544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（4）微服务访问测试

```
curl -i -X GET http://localhost:8000/rest/v1/searchlist --header "X-SDC-APPLICATION-ID:microservice"

HTTP/1.1 200 OK

Date: Fri, 26 Jun 2020 09:20:06 GMT

Content-Type: application/json

Transfer-Encoding: chunked

Server: Jetty(9.4.12.v20180830)

 

{"httpStatusCode":200,"data":[{"id":1,"name":"小明","age":12,"sex":"男"}],"error":[]}

 

curl -i -X POST http://localhost:8000/rest/v1/add --header "X-SDC-APPLICATION-ID:microservice" -d '{"id":2,"name":"小花","age":16,"sex":"女"}'

HTTP/1.1 200 OK

Date: Fri, 26 Jun 2020 09:21:47 GMT

Content-Type: application/json

Transfer-Encoding: chunked

Server: Jetty(9.4.12.v20180830)

 

{"httpStatusCode":200,"data":[{}],"error":[]}

 

curl -i -X GET http://localhost:8000/rest/v1/searchlist --header "X-SDC-APPLICATION-ID:microservice"

HTTP/1.1 200 OK

Date: Fri, 26 Jun 2020 09:22:28 GMT

Content-Type: application/json

Transfer-Encoding: chunked

Server: Jetty(9.4.12.v20180830)

 

{"httpStatusCode":200,"data":[{"id":1,"name":"小明","age":12,"sex":"男"},{"id":2,"name":"小花","age":16,"sex":"女"}],"error":[]}

这次查询会看到新增了一条数据

 

curl -i -X POST http://localhost:8000/rest/v1/update --header "X-SDC-APPLICATION-ID:microservice" -d '{"id":2,"name":".花","age":26,"sex":"女"}'

HTTP/1.1 200 OK

Date: Fri, 26 Jun 2020 09:24:12 GMT

Content-Type: application/json

Transfer-Encoding: chunked

Server: Jetty(9.4.12.v20180830)

 

{"httpStatusCode":200,"data":[{}],"error":[]}

 

curl -i -X POST http://localhost:8000/rest/v1/findbyid --header "X-SDC-APPLICATION-ID:microservice" -d '{"id":2}'

HTTP/1.1 200 OK

Date: Fri, 26 Jun 2020 09:25:45 GMT

Content-Type: application/json

Transfer-Encoding: chunked

Server: Jetty(9.4.12.v20180830)

 

{"httpStatusCode":200,"data":[{"id":2,"name":"小花","age":26,"sex":"女"}],"error":[]}
```


附：

连接需要在库中创建数据库名为test3，表名为student_tb，建表语句：

```sql
CREATE TABLE `student_tb` (

  `id` int(11) NOT NULL,

  `name` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,

  `age` int(11) DEFAULT NULL,

  `sex` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,

  PRIMARY KEY (`id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;


```



# StreamSets实战之路（十一）-基础篇- StreamSets-数据流开发- Edge数据流设计



主要介绍StreamSets中Edge数据流设计，以系统硬件指标采集为例，将从指定的机器上采集相关指标，并将指标发送到数据流中进行处理。

  

Edge数据流是使用较少资源在远端设备运行的工作流（支持的设备系统包括：linux、mac、win、arm等），主要工作物联网设备终端进行数据采集以及进行简单的数据处理，达到终端设备的智能处理，另外，数据在终端设备中进行简单处理，可以减少数据传输，减少带宽占用。

 

（1）创建一个Edge数据流

![img](https://img-blog.csdnimg.cn/2020062618583473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（2）设置数据流，将System Metrics拖入，用于设备硬件指标的采集；将Expression Evaluator、Field Remover拖入，用于将hostid、hostname从hostInfo中提取处理，并删除多余的hostInfo数据；将Destination组件的HTTP Client的组件拖入，用于将数据发送到HTTP 服务端（数据收集端）。

![img](https://img-blog.csdnimg.cn/20200626185908555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626185920985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626185943358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626185957491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（3）创建一个HTTP服务数据流，将接受edge数据流发送的数据，并启动

![img](https://img-blog.csdnimg.cn/20200626190013739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（4）下载edge并安装

![img](https://img-blog.csdnimg.cn/20200626190043522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626190104866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

下载Edge过后根据StreamSets实战之路（二）-环境篇- StreamSets安装与配置进行安装和启动

 

 

（5）发布edge数据流到指定设备

![img](https://img-blog.csdnimg.cn/20200626190121162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（6）启动edge数据流，并观察采集到的数据

![img](https://img-blog.csdnimg.cn/20200626190200219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626190218593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2020062619023337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)



# StreamSets实战之路（十二）-基础篇- StreamSets-数据流管理

主要介绍StreamSets中数据流管理，包括：导入导出、复制、分享、删除、批量启停。

（1）导入导出，在进行数据流迁移时会用到

![img](https://img-blog.csdnimg.cn/20200626194311141.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626194327222.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（2）数据流复制，在需要多个数据流并行执行时需要用到

![img](https://img-blog.csdnimg.cn/20200626194357112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626194409618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626194421684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（3）数据流分享，多用户不同权限的用户之间可以进行数据流的分享

![img](https://img-blog.csdnimg.cn/2020062619451992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20200626194535802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（4）数据流删除，不需要的数据流可以进行批量删除

![img](https://img-blog.csdnimg.cn/20200626194557144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（5）数据流批量启停，在需要进行多个数据流进行批量启动或停止时

![img](https://img-blog.csdnimg.cn/20200626194632516.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2020062619464815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

# StreamSets实战之路（十三）-实战篇- 采集新浪财经实时新闻

主要通过一个新浪财经实时新闻采集的案例来介绍Streamsets（3.20.0 汉化版）的使用，主要包括数据采集、网页数据解析、数据检测、数据入库、邮件通知等。

学习目的：使用streamset实现网络数据的定时爬取、学习一些组件的使用、学习使用邮件组件、数据解析组件、分流组件、http client组件、调度器组件、ES入库组件等。

前一段时间因为工作较忙，没来及时更新，趁着假期准备一下，感觉还是要坚持的，也感谢很多网友的支持。

数据流视图：

![img](https://img-blog.csdnimg.cn/20210223200251579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


最终数据流的效果图：![img](https://img-blog.csdnimg.cn/20210223200400527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


采集数据用Kibana展示的效果：![img](https://img-blog.csdnimg.cn/20210223200521893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

## 前期准备：

1.搭建一个ES和kibana系统，并创建索引：

```json
{

  "sina_livenews" : {

    "mappings" : {
     
      "properties" : {
     
        "create_time" : {
     
          "type" : "date",
     
          "format" : "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
     
        },
     
        "id" : {
     
          "type" : "long"
     
        },
     
        "rich_text" : {
     
          "type" : "text",
     
          "analyzer" : "ik_smart"
     
        },
     
        "tag" : {
     
          "properties" : {
     
            "id" : {
     
              "type" : "keyword"
     
            },
     
            "name" : {
     
              "type" : "keyword"
     
            }
     
          }
     
        },
     
        "type" : {
     
          "type" : "long"
     
        },
     
        "update_time" : {
     
          "type" : "date",
     
          "format" : "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
     
        },
     
        "zhibo_id" : {
     
          "type" : "long"
     
        }
     
      }
     
    }

  }

}
```


2.在streamset启动前配置一个邮件配置项
在etc/sdc.properties配置一下邮件，这里我用的是我自己的qq邮箱

![img](https://img-blog.csdnimg.cn/2021022320065044.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

在etc/email-password.txt中配置邮箱密码。

## 构建步骤：

1.新建数据流，点击保存，进入数据流构建页面![img](https://img-blog.csdnimg.cn/20210223200744513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

2.下面对构建好的数据流中的重点组件做介绍：
（1）调度器组件![img](https://img-blog.csdnimg.cn/2021022320083612.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


主要用来采集的频率，这里规定频率为每2分钟采集数据，根据需求可以定制，不建议采集过于频繁。

（2）调度器组件
    配置一下采集网页地址，请求头。

![img](https://img-blog.csdnimg.cn/20210223201249881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

​     配置数据的格式，并配置是否是压缩数据和解析数据的长度。这里因为我们采集到的是字符串数据，所以我配置为文本，最大长度1024000000。

![img](https://img-blog.csdnimg.cn/20210223201249873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（3）截取字符串提取JSON数据![img](https://img-blog.csdnimg.cn/20210223201249900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

将JSON字符串转换为JSON对象数据

![img](https://img-blog.csdnimg.cn/20210223201249774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

（4）提取真正关注的那部分数据![img](https://img-blog.csdnimg.cn/20210223201249892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


（5）从ES中查看采集到的数据是否已经入库
配置一下ES的连接，查询、用户名密码等等

![img](https://img-blog.csdnimg.cn/20210223201249902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（6）分流器筛选
    使用分流器将不在ES中的数据传下去，在ES中的直接丢弃掉。

![img](https://img-blog.csdnimg.cn/20210223201249855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（7）使用JSON 生成插件将tag字符转化成一个JSON字符串，以备后面做字符串匹配使用。
（8）保留最终想要的数据，去除一些其他无关的数据。![img](https://img-blog.csdnimg.cn/20210223201249875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


（9）配置邮件插件![img](https://img-blog.csdnimg.cn/20210223201249877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


（10）将数据写入ES中
配置一些ES的地址，索引名等，和用户信息

![img](https://img-blog.csdnimg.cn/20210223201249880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

（11）最后校验数据流，校验成功后启动数据流。

![img](https://img-blog.csdnimg.cn/20210223201249880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)



# StreamSets实战之路（十四）-实战篇- 定时数据迁移

主要通过一个定时数据迁移的案例来介绍Streamsets（3.20.0 汉化版）的使用，主要将mysql的数据定时将前一天的数据迁移到ES中，主要包括任务调度器、定时启动数据迁移数据流等。  

学习目的：学习使用调度器定时调度数据迁移数据流、学习数据迁移的流程、学习怎么自动关闭数据流和消息提醒。

最终数据流的效果图：
需要配置两个数据流

![img](https://img-blog.csdnimg.cn/20210223205952136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

数据迁移调度器数据流

![img](https://img-blog.csdnimg.cn/20210223210050443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

数据迁移工作流

![img](https://img-blog.csdnimg.cn/20210223210107501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

## 前期准备：

（1）需要在mysql数据库中准备一张每天都有新增数据的表，该表必须有一个时间字段，用于选取时间范围。

## 构建步骤：

1.首先构建数据迁移数据流
从mysql中读取数据我们选用origin类JDBC插件，做一些配置，包括：连接串、用户名密码、根据数据迁移调度器传来的时间范围配置查询SQL语句、Max Batch Size (Records)等等。

注意：为什么选用origin类的JDBC插件读取数据而不选取processors类的JDBC插件读取数据呢？大家思考一下。

![img](https://img-blog.csdnimg.cn/20210223210128271.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配置一下ES配置信息，包括地址、用户名密码、索引名、索引类型等，这里不需要我们在ES中提前建索引，ES会根据数据生成索引，当然还是建议大家去自己动手建索引。

![img](https://img-blog.csdnimg.cn/2021022321014286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配置一个任务完成就自动关闭数据流的插件，条件为${record:eventType() == 'no-more-data'}，也就是今天的数据迁移完后，自动关闭该数据流。

![img](https://img-blog.csdnimg.cn/20210223210205232.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

在数据流配置中配置一个任务提醒，当任务出现错误或停止或完成时会通过邮件发送通知。

![img](https://img-blog.csdnimg.cn/20210223210219679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

数据流默认的启动参数

![img](https://img-blog.csdnimg.cn/20210223210250755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

2.构建数据迁移调度器数据流![img](https://img-blog.csdnimg.cn/20210223210307246.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


通过Groovy脚本插件将调度器生成的时间戳生成数据查询的时间范围。

![img](https://img-blog.csdnimg.cn/20210223210325881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

通过数据流启动插件启动数据迁移数据流，需要配置数据迁移数据流的地址、ID、数据迁移数据流的启动参数、指定streamset的用户名密码。

![img](https://img-blog.csdnimg.cn/20210223210343871.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)





# StreamSets实战之路（十五）-实战篇- 数据采集与处理

主要通过一个数据采集与处理的案例来介绍Streamsets（3.13.0）的使用，主要将使用Edge数据流收集streamsets系统的日志和主机性能指标，通过收集数据流收集类数据并进行简单处理，发送至kafka中，性能指标数据入库数据流和日志数据入库数据流分别从kafka中消费数据，并将两类数据进行简单处理加载到数据库中。

学习目的：使用edge和streamset的数据互动，使用streamset进行分布式异步数据处理。

数据流图：

![img](https://img-blog.csdnimg.cn/20210224210834116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

最终数据流的效果图：
需要配置5个数据流，两个edge采集数据流，一个数据收集数据流，两个数据处理与入库数据流

![img](https://img-blog.csdnimg.cn/20210224210901800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

## 前期准备：

（1）需要在数据采集的节点上部署安装Edge（不会使用的同学可以参照前面文章）。

（2）一个现成kafka集群，并创建一个两个topic，kafka集群主要为了让数据流达到分布式异步处理的能力。

（3）一个现成的ES集群。

## 构建步骤：

1.首先构建日志数据采集器数据流
配置edge数据流发布的地址（该主机上一定要安装部署了edge）

![img](https://img-blog.csdnimg.cn/20210224210921354.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配一下文件采集文件和数据格式，数据格式我们直接按文本传输

 ![img](https://img-blog.csdnimg.cn/20210224210951439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210224211010706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)



使用destination 类http client配置一下远程数据收集器的地址和APP ID

![img](https://img-blog.csdnimg.cn/20210224211053261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

2.性能指标数据采集器数据流

配置edge数据流发布的地址（该主机上一定要安装部署了edge）

![img](https://img-blog.csdnimg.cn/20210224211109628.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配置一下系统指标采集插件，采集哪些数据和采集的频率，这里我们采集host、cpu、内存、磁盘等，采集频率为两秒

![img](https://img-blog.csdnimg.cn/20210224211124409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用destination 类http client同样配置一下数据收集器远程的url和APP ID

![img](https://img-blog.csdnimg.cn/20210224211140311.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

3.数据收集与处理数据流
使用origin 类http sevice组件，配置数据收集器的端口、最大并发量以及APP ID

![img](https://img-blog.csdnimg.cn/20210224211155592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用Http 路由插件，将接受到的数据路由不到不同分支，这里配置日志和性能指标数据路由。

![img](https://img-blog.csdnimg.cn/2021022421120923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用日志解析插件对收集到的日志数据进行解析，这里我们选择Log4j解析器，选择使用自定义日志格式，这里的格式按照streamset的格式：

```shell
 %d{ISO8601} [user:%X{s-user}] [pipeline:%X{s-entity}] [runner:%X{s-runner}] [thread:%t] [stage:%X{s-stage}] %-5p %c{1} - %m%n
```


![img](https://img-blog.csdnimg.cn/20210224211332491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)使用kafka生成插件将两类数据输出到不同的topic中。

![img](https://img-blog.csdnimg.cn/20210224211545860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

4.日志数据入库数据流
配置kafka地址和日志数据的topic和消费组

![img](https://img-blog.csdnimg.cn/20210224211616967.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

5.性能指标数据入库数据流
配置kafka地址和性能指标数据的topic和消费组

![img](https://img-blog.csdnimg.cn/20210224211635978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)






# StreamSets实战之路（十六）-实战篇-数据序列化与反序列化

主要通过一个数据序列化与反序列化的案例来介绍Streamsets（3.20.0 汉化版）的使用，因为大数据加工与处理的时候，避免不了数据的序列化与反序列化，这里主要讲一下使用数据序列化插件和反序列化插件实现avro格式数据序列化与反序列化，。当然还是可以序列化其他格式，这里挑一个难的讲一下。

学习目的：学习使用Data Generator 和 Data Parser。

 

## 最终数据流的效果图：

需要配置一个数据流。

序列化效果：

![img](https://img-blog.csdnimg.cn/20210224212653645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

反序列化效果：

![img](https://img-blog.csdnimg.cn/2021022421270958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

## 前期准备：

1.一些JSON格式的数据：

```json
{

    "cont":{
     
        "disk_total":1024,
     
        "disk_used":100,
     
        "disk_free":924,
     
        "file_sum":94,
     
        "file_type_sum":20,
     
        "file_grow_num":20,
     
        "disk_grow_num":20,
     
        "file_type_sum_document_docx":200,
     
        "file_type_sum_document_txt":200,
     
        "disk_used_document_docx":200,
     
        "disk_used_document_txt":600
     
    },
     
    "status_tag":{
     
        "tag_version":"1.0",
     
        "data_type":4,
     
        "data_subtype":16387,
     
        "producer_id":24705,
     
        "timestamp":"2020-03-24 12:00:00.00"
     
    }

}
```

Avro schema：

```json
{

       "type": "record",
     
       "name": "cont1",
     
       "fields": [{
     
           "name": "cont",
     
           "type": {
     
                  "type": "record",
     
                  "name": "cont2",
     
                  "fields": [{
     
                         "name": "disk_total",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "disk_used",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "disk_free",
     
                         "type": "int"
     
                     },
     
                      {
     
                         "name": "file_sum",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "file_type_sum",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "file_grow_num",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "disk_grow_num",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "file_type_sum_document_docx",
     
                         "type": "int"
     
                     },
    
                     {
     
                         "name": "file_type_sum_document_txt",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "disk_used_document_docx",
     
                         "type": "int"
     
                     },
     
                     {
     
                         "name": "disk_used_document_txt",
     
                         "type": "int"
     
                     }
     
                  ]     
           }
     
       }]
}
```

## 构建步骤：

1.使用开发测试origin类组件准备一些json格式的数据![img](https://img-blog.csdnimg.cn/20210224212726917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)


2.数据序列成avro格式
使用数据序列化插件将json格式的数据序列化成avro格式并通过byte数组输出，下面配置一下数据输出的字段cont2和类型Byte array。

![img](https://img-blog.csdnimg.cn/20210224212741484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配置一下数据格式为avro，并需要配置一下avro的schema（不知道avro格式的可以学习一下）

![img](https://img-blog.csdnimg.cn/20210224212756549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

3.将序列的数据反序列
使用数据反序列化插件将avro格式的数据反序列化成json格式，下面配置一下要反序列化的数据字段cont2和数据输出的字段cont2（将反序列化的结果还存到cont2中）。

![img](https://img-blog.csdnimg.cn/20210224212814793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 

同样将序列化中用到的avro schema填入。

![img](https://img-blog.csdnimg.cn/20210224212827554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)






# StreamSets实战之路（十七）-实战篇-数据服务快速微服务化

主要通过一个数据服务快速微服务化的案例来介绍Streamsets（3.20.0 汉化版）的使用。在当前下，微服务化异常盛行，肯能大家之前都用过spring boot实现微服务应用（当然不知道的可以搜下 RESTful API，这里涉及的比较多不重点讲），很多原来不是微服务的服务就需要快速地改成RESTful 接口对外提供服务，要是要用spring boot改写的话，想必一定要Coding了，哈哈，现在不需要了，本章通过一个案例介绍怎么通过streamsets快速实现服务的微服务化。

学习目的：学习使用构建微服务数据流、学习对原有的服务快构建RESTful API。

最终数据流的效果图：
这里我们讲两个微服务化的例子，一个对mysql数据库封装一下，让mysql对外提供RESTful API，另一个实现一个加解密的微服务，对外提供RESTful API。

![img](https://img-blog.csdnimg.cn/20210224213515383.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

访问一下效果：

![img](https://img-blog.csdnimg.cn/20210224213515387.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210224213515405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

访问一下结果：

![img](https://img-blog.csdnimg.cn/20210224213515447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

## 前期准备：

（1）部署安装一个mysql数据库是必要的，建一个表插入一些数据。

## 构建步骤：

1.让mysql对外提供RESTful API
选择创建一个微服务数据流，当然若是选择成默认的普通数据流，进入数据流构建画布中也可以修改，具体怎么修改大家可以思考一下。

![img](https://img-blog.csdnimg.cn/20210224213812864.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

配置一下REST Service origin类组件的端口号和APP ID

![img](https://img-blog.csdnimg.cn/20210224213837389.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210224213837430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

选择destination类jdbc插件，配置jdbc连接串、用户名和密码、表名等，并将操作模式分别配置成INSERT、UPDATE、DELETE模式，实现数据增、改、删，这里展示了增加数据的插件，其他两个配置类似，只不过改一个操作模式。

![img](https://img-blog.csdnimg.cn/20210224213859706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210224213859798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

对数据查询这里选用JDBC Lookup插件，配置jdbc连接串、用户名额和密码，查询语句sql由前面的插件或从用户端传送过来。

![img](https://img-blog.csdnimg.cn/20210224213927452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210224213927483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用destination类response插件将结果返回到用户，返回状态码为200.

![img](https://img-blog.csdnimg.cn/20210224213927547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

2.实现一个提供RESTful API的数据加解密服务
同样创建一个微服务数据流，配置RESTful service插件的端口号和APP ID

![img](https://img-blog.csdnimg.cn/20210224214030309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用http路由插件实现数据的连接路由，这里设置请求方法都是post方法，一个加密路由和一个解密路由。

![img](https://img-blog.csdnimg.cn/20210224214051186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

这里使用数据分流插件检测用户传来的数据字段中是否包含field字段，若是不包含没法对该字段进行加解密，所以在加解密之前对字段的存在性进行检测是必要的，要是没有该字段立即返回用户。

![img](https://img-blog.csdnimg.cn/20210224214051212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用加解密插件进行加密，将插件模式设置加密模式，并配置加密字段，并在Key Provider选项卡中配置加密的方法和秘钥，这里的秘钥要生成base64串后填入，注意秘钥的长度要个加密算法中选择的长度要一致。

![img](https://img-blog.csdnimg.cn/20210224214108645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210224214108694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

在解密算法中，解密字段的类型要求是byte array类型，所以使用表达式插件将前端传来的base64的字符串转换成byte array 类型

![img](https://img-blog.csdnimg.cn/20210224214131666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

使用加解密插件进行解密，将插件模式设置解密模式，并配置解密字段，并在Key Provider选项卡中配置解密的方法和秘钥，这里的秘钥要生成base64串后填入，注意秘钥的长度要个加密算法中选择的长度要一致，并注意要与加密中选择的算法和秘钥一致。

![img](https://img-blog.csdnimg.cn/20210224214131691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

 ![img](https://img-blog.csdnimg.cn/20210224214131722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p3emZncg==,size_16,color_FFFFFF,t_70)

注意一个问题：数据在加密后其实是生成了一个byte array类型的数据，但是用户收到是一个base64 串，这是streamset在数据传输过程字段编码的。

