---
title: HashMap
categories:
  - JDK
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:00:51
---


HashMap 实现了Map接口,允许放入key为null的元素,也允许插入value为null的元素,除该类未实现同步外,其余跟hashtable大致相同,跟TreeMap不同,该容器不保证元素顺序,容器可能会对元素重新hash,元素的顺序也会被打散,因此不同时间迭代同一个hashMap的顺序可能会不同. java解决hash冲突的办法是冲突链表方法.  

![HashMap_base.png](http://oss.xiaokoua.cn/blog//HashMap_base_1627094785616.png)  

有两个参数可以影响HashMap的性能:初始容量(inital capacity)和负载系数(load factor).初始容量指定了初始table的大小,负载系数用来指定自动扩容的临界值.当entry的数量超过capacity*load_factor时,容器将自动扩容并重新hash,对于插入元素较多的场景,将初始容量设大可以减少重新hash的次数.  

将对象放到HashMap时,需要特点关注Hashcode()和equals()方法,hashCode()方法决定了对象会被放到哪个bucket里,当多个对象的hash值冲突时,equals()方法决定了这些对象是否是"同一个对象"  