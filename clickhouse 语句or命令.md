# 命令

```sql
--host, -h -– 服务端的host名称, 默认是localhost。您可以选择使用host名称或者IPv4或IPv6地址。
--port – 连接的端口，默认值：9000。注意HTTP接口以及TCP原生接口使用的是不同端口。
--user, -u – 用户名。 默认值：default。
--password – 密码。 默认值：空字符串。
--query, -q – 使用非交互模式查询。
--database, -d – 默认当前操作的数据库. 默认值：服务端默认的配置（默认是default）。
--multiline, -m – 如果指定，允许多行语句查询（Enter仅代表换行，不代表查询语句完结）。
--multiquery, -n – 如果指定, 允许处理用;号分隔的多个查询，只在非交互模式下生效。
--format, -f – 使用指定的默认格式输出结果。
--vertical, -E – 如果指定，默认情况下使用垂直格式输出结果。这与–format=Vertical相同。在这种格式中，每个值都在单独的行上打印，这种方式对显示宽表很有帮助。
--time, -t – 如果指定，非交互模式下会打印查询执行的时间到stderr中。
--stacktrace – 如果指定，如果出现异常，会打印堆栈跟踪信息。
--config-file – 配置文件的名称。
--secure – 如果指定，将通过安全连接连接到服务器。
--history_file — 存放命令历史的文件的路径。
--param_<name> — 查询参数配置查询参数.

```
有一堆或者太长的SQL需要执行，可以写成一个文件，批量执行：
```linux
clickhouse-client --user 用户名 --password 密码 -d 数据库 --multiquery < /root/temp.sql
```
查看SQL的执行计划：
```linux
clickhouse-client -h localhost --send_logs_level=trace  <<<"SQL语句" >/dev/null
```
导入为csv文件：
```linux
clickhouse-client --query="select * from default.t_city" > city.csv
```
或者
```linux
echo 'select * from default.t_city' | curl localhost:8123?database=default -udefault:password -d @- > table_name.sql
```
导入csv文件：
```linux
cat city.csv | clickhouse-client --query "insert into city FORMAT CSV"
```
比较小的数据量导出csv文件，带上字段名，然后导入
```sql
Clickhouse> select * from default.t_city INTO OUTFILE '/data/t_city.csv' FORMAT CSVWithNames;

SELECT *
FROM default.t_city
INTO OUTFILE '/data/t_city.csv'
FORMAT CSVWithNames

8 rows in set. Elapsed: 0.050 sec. 

cat /data/t_city.csv | clickhouse-client --query="INSERT INTO default.t_city FORMAT CSVWithNames"  --password
```
当数据存在两个集群或者主机的时候可以采用方法：
```sql
insert into default.t_city VALUES
select * from remote('127.0.0.1:9000', default, t_city);

-- 当数据巨大的时候可以采用官方提供的工具clickhouse-copier：
clickhouse-client --host <source>   -q "SELECT * FROM default.t_city FORMAT CSVWithNames" | \
clickhouse-client --host <target> --port 9000  -q "INSERT INTO defualt.t_city FORMAT CSVWithNames"

-- 利用linux的管道命令 节省内存，磁盘和运行时间。
```
# 语句

