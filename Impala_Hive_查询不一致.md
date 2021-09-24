# Impala 与 Hive 查询结果不一致

## 现象

1. 使用相同SQL 过滤统计查询，Hive 是12条，Impala是29条，并且重新覆盖数据后 Impala 的结果会变动。

2. 使用新的表名，Impala 和Hive 查询结果一致。

3. 通过`refresh table` 和 `invalidate metadata table`更新数据和元数据后仍然无效。

## 解决

1. 删掉数据表 ，同时删除 HDFS 垃圾箱数据:
```SQL
Drop table if exists tablename PURGE;
```
2. 删掉hdfs 中表的对应目录:
```shell
hdfs dfs -rm -R /xxxxx/DBxxx.db/tablename;
```
3. 重启所有Impala 服务(Impalad,Statestored,catalogd)

OK!
