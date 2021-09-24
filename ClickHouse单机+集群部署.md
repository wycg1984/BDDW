@[TOC](Clickhouse集群搭建)
# Clickhouse集群搭建（rpm）
## 搭建前准备
>配置好防火墙以及搭建好zookeeper,这里就不做搭建,可自行查找攻略
>首先取消打开文件数限制,然后重启不然不生效
```linux
[root@node02 ~]#  vim /etc/security/limits.conf
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072
```
>确保安装了curl，没有的话，安装一下  
```linux
sudo yum install -y curl
```
## 开始安装
### 添加clickhouse源
```linux
curl -s https://packagecloud.io/install/repositories/altinity/clickhouse/script.rpm.sh | sudo bash
```
>由于网络原因，可能会失败，多试几次

### 查一下clickhouse源有没有添加成功
```linux
sudo yum list 'clickhouse*'
```
### 安装servier和client
```linux
sudo yum install -y clickhouse-server clickhouse-client
```
### 检查是否安装成功
```linux
sudo yum list installed 'clickhouse*'
```
### 通过 clickhouse service 来启动,停止,clickhouse
```linux
service clickhouse-server start | stop | status | restart |
```
### 启动服务
```linux
sudo service clickhouse-server start
```
>这种是后台启动，自动使用/etc/clickhouse-server/config.xml作为配置文件，也可以手动启动，指定配置文件：
```linux
clickhouse-server --config=/etc/clickhouse-server/config.xml
```
>    DONE    标识启动成功
```linux
Start clickhouse-server service: Path to data directory in /etc/clickhouse-server/config.xml: /var/lib/clickhouse/
DONE
```
### 使用客户端
```linux
clickhouse-client
```
### 退出客户端
```linux
localhost :) exit
Bye.
```
### 配置 config.xml
```linux
vim /etc/clickhouse-server/config.xml
```
```xml
# 进入编辑后  /<listen_host> 回车,找到这个位置,这个配置是用于,允许IP4和IP6源主机远程访问
     <listen_host>::</listen_host>
    <!-- Same for hosts with disabled ipv6: -->
    <!-- <listen_host>0.0.0.0</listen_host> -->
# 将注释的内容给打开即可, 注意每台节点都要更改
```
>然后进入到/etc目录下创建metrika.xml文件
```linunx
vim metrika.xml   该文件中的配置在config.xml中都有，本人觉的单本一个这个文件是为了配置方便。（不对请指出）
```
>vim metrika.xml内容,我这里是四台节点,配置文件是这样的  **4分片1备份**
>您也可以根据自己的实际情况来搭配。
```xml
<yandex>
# 集群配置
<clickhouse_remote_servers>
	#集群名字
    <perftest_4shards_1replicas>
    	#分片
        <shard>
        		#如果设置为true，则往本地表写入数据时，总是写入到完整健康的副本里，然后由表自身完成复制，这就要求本地表是能自我复制的。
        		#如果设置为false，则写入数据时，是写入到所有副本中。这时，是无法保证一致性的。
             <internal_replication>true</internal_replication>
             #备份
            <replica>
                <host>node01</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <replica>
                <internal_replication>true</internal_replication>
                <host>node02</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <internal_replication>true</internal_replication>
            <replica>
                <host>node03</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
             <internal_replication>true</internal_replication>
            <replica>
                <host>node04</host>
                <port>9000</port>
            </replica>
        </shard>
    </perftest_4shards_1replicas>   
</clickhouse_remote_servers>
# zk配置
<zookeeper-servers>
  <node index="1">
    <host>node01</host>
    <port>2181</port>
  </node>
  <node index="2">
    <host>node02</host>
    <port>2181</port>
  </node>
  <node index="3">
    <host>node03</host>
    <port>2181</port>
  </node> 
  <node index="4">
    <host>node04</host>
    <port>2181</port>
  </node>
</zookeeper-servers>
# 这个位置需要根据不同的节点来更改
<macros>
    <replica>node02</replica>
</macros>
<networks>
   <ip>::/0</ip>
</networks>
#数据的压缩算法设置
<clickhouse_compression>
<case>
  <min_part_size>10000000000</min_part_size>                                         
  <min_part_size_ratio>0.01</min_part_size_ratio>                                                                                                                                       
  <method>lz4</method>
</case>
</clickhouse_compression>
</yandex>
```
>然后将metrika.xml文件分发到其他的节点,然后停止之前启动clickhouse,然后每台节点启动服务,对没看错,不需要在做什么事情了,他会自动读取这个文件,启动就行了

### 每台节点启动完成了查看是否启动
```linux
ps -aux | grep clickhouse-server
clickho+ 244604  0.4  1.1 1971184 368656 ?      Ssl  18:57   0:09 clickhouse-server --daemon --pid-file=/var/run/clickhouse-server/clickhouse-server.pid --config-file=/etc/clickhouse-server/config.xml
root     248725  0.0  0.0 112824   980 pts/0    S+   19:36   0:00 grep --color=auto clickhouse-server
```
### 如果启动失败可以去查看log ,log位置
```linux
[root@node02 clickhouse-server]# pwd
/var/log/clickhouse-server
[root@node02 clickhouse-server]# ll
总用量 864
-rw-r-----. 1 clickhouse clickhouse 154597 11月 25 18:17 clickhouse-server.err.log
-rw-r-----. 1 clickhouse clickhouse 720270 11月 25 19:37 clickhouse-server.log
-rw-r-----. 1 clickhouse clickhouse   5395 11月 25 18:57 stderr.log
-rw-r-----. 1 clickhouse clickhouse      0 11月 25 18:14 stdout.log
```
### 启动完成后我们进入client ,说明成功了
```linux
[root@node02 clickhouse-server]# clickhouse-client 
```
>查看集群信息,可以看到以下信息 
>perftest_4shards_1replicas这个是我们在metrika.xml配置文件中配置的 select * from system.clusters;
```ClickHose
SELECT *
FROM system.clusters;

┌─cluster───────────────────────────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name─┬─host_address──┬─port─┬─is_local─┬─user────┬─default_database─┬─errors_count─┬─estimated_recovery_time─┐
│ perftest_4shards_1replicas        │         1 │            1 │           1 │ node01    │ 192.168.2.217 │ 9000 │        0 │ default │                  │            0 │                       0 │
│ perftest_4shards_1replicas        │         2 │            1 │           1 │ node02    │ 192.168.2.202 │ 9000 │        1 │ default │                  │            0 │                       0 │
│ perftest_4shards_1replicas        │         3 │            1 │           1 │ node03    │ 192.168.2.203 │ 9000 │        0 │ default │                  │            0 │                       0 │
│ perftest_4shards_1replicas        │         4 │            1 │           1 │ node04    │ 192.168.2.218 │ 9000 │        0 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards           │         1 │            1 │           1 │ 127.0.0.1 │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards           │         2 │            1 │           1 │ 127.0.0.2 │ 127.0.0.2     │ 9000 │        0 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards_localhost │         1 │            1 │           1 │ localhost │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_cluster_two_shards_localhost │         2 │            1 │           1 │ localhost │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_shard_localhost              │         1 │            1 │           1 │ localhost │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_shard_localhost_secure       │         1 │            1 │           1 │ localhost │ 127.0.0.1     │ 9440 │        0 │ default │                  │            0 │                       0 │
│ test_unavailable_shard            │         1 │            1 │           1 │ localhost │ 127.0.0.1     │ 9000 │        1 │ default │                  │            0 │                       0 │
│ test_unavailable_shard            │         2 │            1 │           1 │ localhost │ 127.0.0.1     │    1 │        0 │ default │                  │            0 │                       0 │
└───────────────────────────────────┴───────────┴──────────────┴─────────────┴───────────┴───────────────┴──────┴──────────┴─────────┴──────────────────┴──────────────┴─────────────────────────┘
```





# Clickhouse单机部署（RPM）

### clickhouse的rmp安装包下载地址为：

> https://packagecloud.io/Altinity/clickhouse。

### 需要下载四个rmp包：

> client、server、common-static、server-common。

### 这里我下载的是：

> 可自行下载最新版本。

clickhouse-client-19.16.3.6-1.el7.x86_64.rpm、
clickhouse-server-19.16.3.6-1.el7.x86_64.rpm、
clickhouse-common-static-19.16.3.6-1.el7.x86_64.rpm、
clickhouse-server-common-19.16.3.6-1.el7.x86_64.rpm

### 安装顺序为
>顺序最还不要错，否则可能会出错。

rpm -ivh clickhouse-common-static-19.16.3.6-1.el7.x86_64.rpm
rpm -ivh clickhouse-server-common-19.16.3.6-1.el7.x86_64.rpm
rpm -ivh clickhouse-server-19.16.3.6-1.el7.x86_64.rpm
rpm -ivh clickhouse-client-19.16.3.6-1.el7.x86_64.rpm


### 启动服务
输入：
```linuxn
sudo service clickhouse-server start
```
若正常启动，会显示：

```linux
Start clickhouse-server service: Path to data directory in /etc/clickhouse-server/config.xml: /var/lib/clickhouse/
DONE
```

### 进入客户端
输入：
```linux
clickhouse-client
```
控制台会显示
```linux
ClickHouse client version 19.16.3.6.
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 19.16.3 revision 54427.
```
### 修改配置文件
默认配置路径
```text
logger : /var/log/clickhouse-server/clickhouse-server.log
data   : /var/lib/clickhouse/
```
> 也可以用上文的metrika.xml文件  单机也可以不用。仅仅安装就好。

# 明文密码

user.xml文件
```linux
vim /etc/clickhouse-server/users.xml
```
修改密码
```xml
<!-- Users and ACL. -->
    <users>
        <!-- If user name was not specified, 'default' user is used. -->
        <default>
            <!-- Password could be specified in plaintext or in SHA256 (in hex format).


             If you want to specify password in plaintext (not recommended), place it in 'password' element.
             Example: <password>qwerty</password>.
             Password could be empty.

             If you want to specify SHA256, place it in 'password_sha256_hex' element.
             Example: <password_sha256_hex>65e84be33532fb784c48129675f9eff3a682b27168c0ea744b2cf58ee02337c5</password_sha256_hex>
             Restrictions of SHA256: impossibility to connect to ClickHouse using MySQL JS client (as of July 2019).

             If you want to specify double SHA1, place it in 'password_double_sha1_hex' element.
             Example: <password_double_sha1_hex>e395796d6546b1b65db9d665cd43f0e858dd4303</password_double_sha1_hex>

             How to generate decent password:
             Execute: PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
             In first line will be password and in second - corresponding SHA256.

             How to generate double SHA1:
             Execute: PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha1sum | tr -d '-' | xxd -r -p | sha1sum | tr -d '-'
             In first line will be password and in second - corresponding double SHA1.
        -->
        <password>123456</password>
```
重启服务
```linux
service clickhouse-server restart
```
# sha256sum【加密密码】
```linux
echo -n 123456 | sha256 | tr -d '-'
8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92    -- 不是和123456对应的，只是示例。
```
修改default用户密码
```xml
<!-- Users and ACL. -->
<users>
    <!-- If user name was not specified, 'default' user is used. -->
    <default>
        <!-- Password could be specified in plaintext or in SHA256 (in hex format).

             If you want to specify password in plaintext (not recommended), place it in 'password' element.
             Example: <password>qwerty</password>.
             Password could be empty.

             If you want to specify SHA256, place it in 'password_sha256_hex' element.
             Example: <password_sha256_hex>65e84be33532fb784c48129675f9eff3a682b27168c0ea744b2cf58ee02337c5</password_sha256_hex>
             Restrictions of SHA256: impossibility to connect to ClickHouse using MySQL JS client (as of July 2019).

             If you want to specify double SHA1, place it in 'password_double_sha1_hex' element.
             Example: <password_double_sha1_hex>e395796d6546b1b65db9d665cd43f0e858dd4303</password_double_sha1_hex>

             How to generate decent password:
             Execute: PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
             In first line will be password and in second - corresponding SHA256.

             How to generate double SHA1:
             Execute: PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha1sum | tr -d '-' | xxd -r -p | sha1sum | tr -d '-'
             In first line will be password and in second - corresponding double SHA1.
        -->
        <!--
        <password>123456</password>
        -->
        <password_sha256_hex>8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92</password_sha256_hex>
```
重启服务
```linux
service clickhouse-server restart
```

