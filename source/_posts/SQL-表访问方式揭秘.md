---
title: SQL 表访问方式揭秘
categories:
  - mysql
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:08:09
---


## 单表访问方法  
对于单表查询来说,查询的执行方式大致分为以下两种: 
- 使用全表扫描进行查询
- 使用索引进行查询  

### const 
我们可以直接通过主键列来定位一条记录,比如我们创建一个案例: 

```
CREATE TABLE table (
id INT NOT NULL AUTO_INCREMENT,
key1 VARCHAR(100),
key2 INT,
key3 VARCHAR(100),
key_part1 VARCHAR(100),
key_part2 VARCHAR(100),
key_part3 VARCHAR(100),
common_field VARCHAR(100),
PRIMARY KEY (id),
KEY idx_key1 (key1),
UNIQUE KEY idx_key2 (key2),
KEY idx_key3 (key3),
KEY idx_key_part(key_part1, key_part2, key_part3)
) Engine=InnoDB CHARSET=utf8;

select * from table where id=1438;
```
![Snipaste_20210520_220728.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-05-20_22-07-28_1621519662111.png)  

### ref访问方式  
看看使用二级索引并回表方式的查询步骤:  
```
select * from table where k2=3841
```
![Snipaste_20210520_221032.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-05-20_22-10-32_1621519842586.png)  

注意以下几个情况:  
- 无论普通二级索引还是唯一二级索引,它们的索引列对包含null值得数量并不限制,所以我们采用 key is null 这种形式的搜索条件最多只能使用ref的访问方法而不是const的访问方法.  
- 对于包含多个索引列的二级索引来说,只要是最左边连续的索引列与常数的等值比较就可能采用ref的访问方法:  
```
select * from table where key_part1='god like' and key_part12='legendary' and key_part3='pent'
```
但是如果最左边的连续索引列并不全是等值比较的话,它的访问方法就不能称为ref了,比如:  
```
select * from table where key_part1='god like' and key_part2> 'leng'
```
### ref_or_null 
```
select *  from table where key1='abc' or key1 is null 
```
![Snipaste_20210520_222516.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-05-20_22-25-16_1621520726788.png)  

### range  
range 就是对于某个key 进行范围查询,略.. 

### index 
```
select key_part1,key_part2,key_part3 from table where key_part2='abc' 
```
由于 key_part2 并不是联合索引idx_key_part最左边索引列,所以我们无法使用ref或者range访问方法来执行这个语句,但是这个语句刚好是覆盖索引,而且key_part2 也在此索引中,也就是说我们可以通过遍历idex_key_part索引的叶子节点的记录来比较key_part2='abc' 这个条件是否成立,把成功匹配的假如结果集,这种采用遍历二级索引记录的执行方式称为index.  

### all 
全表扫描,就是直接扫描聚簇索引  

### 有的搜索条件无法使用索引的情况  
```
SELECT * FROM table WHERE key2 > 100 OR common_field = 'abc';
```  
我们把使用不到 idx_key2 索引的搜索条件替换为 TRUE
```
SELECT * FROM single_table WHERE key2 > 100 OR TRUE
```
接着简化  
```
SELECT * FROM single_table WHERE TRUE;
```  
也就是说一个使用到索引的搜索条件和没有使用该索引的搜索条件使用or连接起来后是无法使用该索引的.  

### 索引合并  
使用到多个索引来完成一次查询的执行方法称之为： index merge   

#### Intersection合并

mysql在特定的情况下才可能会使用到Intersection索引合并:  
- 情况一: 二级索引是等值匹配的情况,对于联合索引来说,在联合索引中的每个列都必须等值匹配,不能出现只出现匹配部分列的情况.  
- 情况二:主键可以是范围查询,二级索引必须是等值匹配,因为只有在这种情况下根据二级索引查询出的结果集是按照主键值排序的.  


#### union合并  
- 情况一: 二级索引是等值匹配的情况,对于联合索引来说,在联合索引中的每个列都必须等值匹配,不能出现只匹配部分列的情况  
- 情况二:主键列可以是范围匹配 
- 情况三: 使用 Intersevtion索引合并的搜索条件  


