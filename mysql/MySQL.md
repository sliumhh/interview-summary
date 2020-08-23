[TOC]

## MySQL是3层还是4层？

取决于数据类型和数据量，索引越小越好；

前缀索引优化，减小索引；

## 为什么推荐id自增？

取决于数据库是不是分布式的，不建议

不是分布式，推荐，插入数据，顺序后再后面插入数据，中间插入数据，会导致页的分裂、合并，16k很快，但是并行操作时出问题；

1. ### 回表：

   使用二级索引（辅助索引）时，没有发生索引覆盖

   ---

2. ### 索引覆盖：

   select id from table where name = ?;

   ---

3. ### 索引下推：

   数据存储在磁盘中，mysql有自己的服务，要跟磁盘发生交互

   ​	**没有索引下推：**先从存储引擎中根据一个索引（name）拉取数据；，再mysql server根据另一个字段（age）筛选；

   ​		缺点：IOl量大

   ​	**有索引下推：**会根据name，age来获取数据，不需要server做任何的数据筛选；

   ​		缺点：需要在磁盘上多做数据筛选，原来的筛选是放在内存中的，现在放在磁盘，查找数据的环节，看起来成本比较高，但是数据是排序的，所有的数据是聚集存放的，所以性能不会有影响，而且整体的IO量会大大减少，反而提升性能。

   ---

   

4. ### 最左匹配：

   4.1 组合索引：（name,age）

   ~~~sql
   select * from table where name = ? and age=?
   
   select * from table where name = ?;
   
   select * from table where age = ?; --不会走索引，不符合最左匹配
   
   select * from table where age = ? and name = ?  --会走索引,mysql优化器:CBO,基于成本的优化  
   ~~~

   CBO 成本优化

##### MRR：mult_range_read

​	内存排序，--》范围查找

##### FIC：fast index create

​	插入和删除数据：

		1. 先创建一个临时表，将数据导入临时表；
  		2. 把原始表删除
    		3. 修改临时表的名字

给当前表添加一个share锁，不会创建临时文件的资源消耗，还是在源文件中，但是此时如果有人发起dml操作，很明显会导致数据不一致，索引添加share锁，读取时没有为题的，但是DML会有问题