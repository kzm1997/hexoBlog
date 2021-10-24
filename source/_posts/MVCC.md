---
title: MVCC
categories:
  - mysql
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 14:55:51
---

### 事务并发执行下遇到的问题

- 脏写 
如果一个事务修改了另一个未提交事务修改过的数据,就是发生了**脏写**,  
![Snipaste_20210928_072825.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-09-28_07-28-25_1632785315057.png)  
sessionA和sessionB各开始了一个事务,如果sessionB中的事务发生了回滚,那么sessionA中的更新也不复存在,这种现象就是脏写 

- 脏读  
如果一个事务读到了另一个未提交修改过的数据,那就是发生了脏读
![Snipaste_20210928_073239.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-09-28_07-32-39_1632785570717.png)  
sessionB先修改了number为1的数据,在sessionB还未提交事务的时候,sessionA读取了number为1的数据,得到name为'关羽',那么sessionA就读取到了sessionB还未提交的脏数据

- 不可重复读

如果一个事务只能读到另一个已经提交的的事务修改过的数据,并且其他事务每对该数据进行一次修改并提交后,该事务都能查询到最新值,那么就是发生了不可重复读  

![Snipaste_20210928_074117.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-09-28_07-41-17_1632786087192.png)  

sessionB提交了几个事务(隐式提交,语句结束事务就提交了),每次提交后,sessionA中的事务都可以查看到最新的值,这种现象称为不可重复读.  
