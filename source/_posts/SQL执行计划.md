---
title: SQL执行计划
categories:
  - mysql
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:08:52
---


### Explain 

```
explain select 1
```
执行上面sql语句,将得到下面执行计划:  
![Snipaste_20210310_170616.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-03-10_17-06-16_1615367395708.png)  

解释一下每个字段的意思:  
- id  在一个大的查询语句中每个 SELECT 关键字都对应一个唯一的 id
- select_typ SELECT 关键字对应的那个查询的类型
- table 表名 
- partitions 匹配的分区信息
- possible_keys 可能用到的索引
- key 实际上使用的索引
- key_len 实际使用到的索引长度
- ref  当使用索引列等值查询时，与索引列进行等值匹配的对象信息
- rows 预估的需要读取的记录条数
- filtered 某个表经过搜索条件过滤后剩余记录条数的百分比
- Extra 一些额外的信息 

我们看一下连接查询
```
explain select * FROM test1 t1 INNER JOIN test2 t2 on t1.guid=t2.guid
```
![Snipaste_20210310_172117.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-03-10_17-21-17_1615368088190.png)  


##### id 
查询语句中每出现一个 SELECT 关键字，MySQL就会为它分配一个唯一的 id值。
对于连接查询来说，一个 SELECT 关键字后边的 FROM 子句中可以跟随多个表，所以在连接查询的执行计划中，每个表都会对应一条记录，但是这些记录的id值都是相同的.出现在前边的表表示驱动表，出现在后边的表表示被驱动表  


对于包含 UNION 子句的查询语句来说，每个 SELECT 关键字对应一个 id 值也是没错的，不过还是有点儿特别的:  

```
EXPLAIN SELECT * FROM single_table s1 WHERE s1.key1 = 'a' UNION SELECT * FROM single_table s2 WHERE s2.key1 = 'a'
```
![Snipaste_20210310_180011.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-03-10_18-00-11_1615370425433.png)

UNION字句会把多个查询的结果集合并起来并对结果集中的记录进行去重,怎么去重? 它使用的是内部临时表,正如上边的查询计划中表示,UNION字句是为了把id为1的查询和id为2的查询结果集合并起来并去重,所以在内部创建了一个名为<union1,2>的临时表,跟 UNION 对比起来， UNION ALL 就不需要为最终的结果集进行去重，它只是单纯的把多个查询的结果集中的记录合并成一个并返回给用户，所以也就不需要使用临时表。所以在包含 UNION ALL 子句的查询的执行计划中，就没有那个 id 为 NULL 的记录  

##### select_type  
- simple 查询语句中不包含UNION或者子查询的都算做SIMPLE类型,比如单表查询和连接查询.  
- primary 对于包含UNION,UNION ALL 或者子查询的大查询来说,它是由几个小查询组成的,其中最左边的那个查询的select_type 值就是primary,比方说:  
```
EXPLAIN SELECT * FROM single_table s1 WHERE s1.key1 = 'a' UNION SELECT * FROM single_table s2 WHERE s2.key1 = 'a'
```
![Snipaste_20210310_211907.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-03-10_21-19-07_1615382364591.png)  
- union 对于包含 UNION 或者 UNION ALL 的大查询来说，它是由几个小查询组成的，其中除了最左边的那个小查询以外，其余的小查询的 select_type 值就是 UNION,临时表就是**UNION RESULT** 
- SUBQUERY 如果包含子查询的查询语句不能转为对应的emi-join的形式,并且该子查询是不相关子查询,并且查询优化器决定采用将该子查询物化的方式来执行该子查询时,该子查询的第一个select关键字代表的那个查询的select_type就是SUBQUERY,比如下面的查询:  

- DEPENDENT SUBQUERY  如果包含子查询的查询语句不能够转为对应的 semi-join 的形式，并且该子查询是相关子查询，则该子查询的第一个 SELECT 关键字代表的那个查询的 select_type 就是 DEPENDENT SUBQUERY,**select_type为DEPENDENT SUBQUERY的查询可能会被执行多次**
```
EXPLAIN SELECT
	* 
FROM
	single_table s1 
WHERE
	s1.key1 IN ( SELECT s2.key1 FROM single_table s2 WHERE s1.key2 = s2.key2 ) 
	OR s1.key3 = 'a'
```
- DEPENDENT UNION 