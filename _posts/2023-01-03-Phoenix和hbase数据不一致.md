---
layout: post
title: "Phoenix和hbase数据不一致"
tags: ["phoenix","hbase","java"]
---

目前为止遇到过的不同情况下的Phoenix和hbase查询数据不一致的情况，做个记录。

## 索引问题

### 问题现象
phoenix出现一些“幽灵”数据，即通过
```roomsql
select count(*);
```
查询数据总量要大于
```roomsql
select * from table_name;
```
真实查询出的数据量。甚至诡异增长。

### 解决办法
重建该表索引
```shell
alter index INDEX_NAME on TABLE_NAME REBUILD;
```

***注意，所有写入操作最好都通过phoenix写入，直接调用hbase接口写入数据时phoenix索引无法感知，
也有可能出现数据不一致***


## 重新建表

### 问题现象
有一张表phoenix执行
```roomsql
select count(*) from TABLE_XXX;
```
可以查询到数据，但是再执行
```roomsql
select * from TEBLE;
```
却返回空，hbase shell执行
```roomsql
count TABLE_XXX
```
返回结果与phoenix count(*) 相同，执行
```shell
scan TABLE_XXX
```
也可以正常返回数据。 还有最重要的一个现象，phoenix执行
```roomsql
select ROW from TABLE_XXX;
```
可以正常查询出数据。

数据真实存在，那么可能是映射出现了问题。

### 解决方法
+ hbase shell执行
```shell
desc TABLE_XXX
```
结果类似于
```shell
Table TABLE_XXX is ENABLED                                                                                                                                                   
TABLE_XXX, {TABLE_ATTRIBUTES => {coprocessor$1 => '|org.apache.phoenix.coprocessor.ScanRegionObserver|805306366|', coprocessor$2 => '|org.apache.phoenix.coprocessor.Ungroupe
dAggregateRegionObserver|805306366|', coprocessor$3 => '|org.apache.phoenix.coprocessor.GroupedAggregateRegionObserver|805306366|', coprocessor$4 => '|org.apache.phoenix.coproce
ssor.ServerCachingEndpointImpl|805306366|', coprocessor$5 => '|org.apache.phoenix.hbase.index.Indexer|805306366|org.apache.hadoop.hbase.index.codec.class=org.apache.phoenix.inde
x.PhoenixIndexCodec,index.builder=org.apache.phoenix.index.PhoenixIndexBuilder'}                                                                                                 
COLUMN FAMILIES DESCRIPTION                                                                                                                                                      
{NAME => 't', BLOOMFILTER => 'NONE', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'FAST_DIFF', TTL => 'FOREVER', COMPRESSION => '
NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                                                
1 row(s) in 0.1380 seconds
```
可以看到phoenix的coprocessors,我们删除这些coprocessors
+ hbase shell先将表disable
```shell
disable TABLE_XXX
```
+ hbase shell删除coprocessors
```shell
alter TABLE_XXX,METHOD=>'table_att_unset',NAME=>'coprocessor$1'
alter TABLE_XXX,METHOD=>'table_att_unset',NAME=>'coprocessor$2'
...
alter TABLE_XXX,METHOD=>'table_att_unset',NAME=>'coprocessor$5'
```
+ 如果hbase的desc除了正常的t之外，还包含如下输出（没有则无需操作）
```shell
{NAME => 'L#t', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => '172800 SECONDS (2 DAYS)', COMP
RESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
```
也将其删除
```shell
alter "vul_list",{NAME => 'L#t',METHOD => 'delete'}
```
+ 如果phoenix该表有索引表，将其删除
phoenix-sqlline中执行
```shell
!tables
```
查看，使用drop删除。

+ 删除phoenix中的表信息
```roomsql
DELETE from SYSTEM.CATALOG where TABLE_NAME =‘TABLE_XXX’;
```

+ 重启hbase
+ hbase shell中enable改表
```roomsql
enable TABLE_XXX
```
+ phoenix-sqlline重新建该表