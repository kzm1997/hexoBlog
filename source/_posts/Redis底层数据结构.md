---
title: Redis底层数据结构
categories:
  - Redis
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 14:59:09
---


## 简单动态字符串  
redis没有使用C语言的字符串表示(以空字符结尾的字符数组),而是构建了一种名为简单字符串(simple dynamic string,SDS)的数据结构,SDS除了在保存字符值外,SDS还被用作缓冲区(buffer):AOF模块中的,以及客户端状态的输入缓冲区.  

### SDS定义:  
![Snipaste_20210812_072500.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_07-25-00_1628724317755.png)  

因为保存了字符串的长度,所以查询字符串长度(strlen)的时间复杂度是O(1)

redis设置的free空间可以杜绝缓冲区溢出 ,减少修改字符串时带来的内存重分配次数
当SDS的API对一个SDS进行修改,并且需要对SDS进行空间扩展的时候,程序不仅会为SDS分配修改所必须要的空间,还会为SDS分配额外的未使用空间.  

#### 空间预分配
在SDS的长度小于1MB的时候,Redis会为SDS分配和len属性同样大小的未使用空间(比如13kb时,分配13kb的未使用空间,那么总长度27kb),当长度大于1MB后,会为SDS分配1MB的为使用空间(比如30MB时,分配1MB,总长度31MB+1kb),  
当将字符串修改为一个更长的字符串时,如果len长度不够,那么将进行扩容,在第二次修改字符串时,如果检测到字符串的未使用空间够本次修改使用,那么将不需要扩容

#### 惰性释放  
惰性释放用于优化字符串缩短操作,程序并不立即回收多出来的字节. 

例子:  

![Snipaste_20210812_072500.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_07-25-00_1628726243960.png)
![Snipaste_20210812_075752.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_07-57-52_1628726291375.png)

惰性释放后,如果修改字符串为更长的字符串,可能可以为redis省去一次预分配的操作,算是一种空间换时间的操作. 

## 链表    

redis链表就是一个双向链表  

## 字典hash  

redis的hash结构由dictht定义  
![Snipaste_20210812_072500.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_07-25-00_1628776000904.png)  
```
typedef struct dictht{
  dictEntry **table;  //哈希表数组
  unsigned long size;  //哈希表大小 
  unsigned long sizemask; //哈希表大小掩码,用于计算索引值,总是等于size-1
  unsigned long used;  //哈希表已有节点的数量
}dictht
```
```
typedef struct dictEntry{
  void * key //键
  union{  //值
    void * val;
    uint64_tu64;
    int64_ts64;
 }v
  //指向下一个哈希表节点,形成链表
  struct dictEntry *next;
}dictEntry
```
key属性保存着键值对中的键,而v属性保存键值对的值,值可以是一个指针,或者是一个uint64_t整数,或者是int64_t整数. 
```
typedef struct dict{
  //类型特定函数 
  dictType *type;
  //私有数据 
  void *privdata;
  //哈希表
  dictht ht[2];
  int rehashidx;  //rehash索引,当rehash不进行时,值为-1;
}
``` 
type指针指向了保存了一簇用于操作特定类型键值对的函数dictType.  

ht属性是一个包含两个项的数组,每项都是dictht哈希表,只有在rehash的时候会用到ht[1] ,rhashidx记录了rehash的速度. 如果没有在hash,则值为-1  

![Snipaste_20210812_220628.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_22-06-28_1628777203654.png)

下面简单说一下rehash:  

当hash表需要扩容时,会为ht[1]分配内存空间,分配的空间的大小也是2的次幂的大小和java一样(ps:为了减少在rehash时,hash的重新分布计算,提高rehash的速度)
![Snipaste_20210812_221050.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_22-10-50_1628777470397.png)
![Snipaste_20210812_221127.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_22-11-27_1628777501726.png)  
![Snipaste_20210812_221225.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_22-12-25_1628777556629.png)  

redis会利用rehashidx进行,渐进式hash,通过递增该值,反映rehash进度,这样可以不用等待rehash全部完成才能进行接下去的操作.  


## 跳跃表  

redis利用跳跃表实现zset 

先来解释一下什么是跳跃表  

对于一个单链表来说,即便表中存储的数据是有序的,如果我们要在其中查找出某个数据,也只能从头到尾遍历链表,这样查找效率会很低,如果我们想要提高查找效率,可以考虑在链表上建立索引,每两个节点提取一个节点到上一级,我们把抽出来的那一级叫做索引.  

![一层跳跃表.png](http://oss.xiaokoua.cn/blog//%E4%B8%80%E5%B1%82%E8%B7%B3%E8%B7%83%E8%A1%A8_1628778246574.png)

这个时候,如果我们假设要查找节点8,我们可以先在索引层遍历,当遍历到索引层中值为7的节点时,发现下一个节点是9,那么要查找的节点肯定就在这之间,我们下降到连表层就找到了这个节点. 同理,再加一层索引:  
![二层跳跃表.png](http://oss.xiaokoua.cn/blog//%E4%BA%8C%E5%B1%82%E8%B7%B3%E8%B7%83%E8%A1%A8_1628778504952.png)
其实这也是利用二分的思想  
![Snipaste_20210812_222925.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-08-12_22-29-25_1628778684969.png)

zskiplist结构包含以下属性:  
header:指向跳跃表的头结点. 
tail:指向跳跃表的尾节点.
level:记录目前跳跃表内,层数最大的那个节点的层数.
length:记录跳跃表的长度,就是跳跃表目前包含的节点的数量.  

```
typedef struct zskiplistNode{
  //层
  struct zskiplistLevel{
     //前进指针 
     struct zskiplistNode *forward;
     //跨度
     unsigned int span;
  }level[];
  //后退指针
  struct zskipListNode * backward;
  //分值
  double score;  
  //成员对象 
  robj *obj;
}
```
zskiplistNode包含以下属性: 
level数组:其内部元素指向下一个索引的指针 
前进指针:指向下一个索引
跨度:跨度并不只是和遍历有关,跨度实际上是用来计算排位(rank)的,在查找节点的过程中,将沿途访问过的所有层的跨度累计起来,得到的结果就是目标节点在跳跃表中的排位.  
后退指针:指向上一个节点

在同一个跳跃表中,多个节点可以包含相同的分值,但每个节点的成员对象必须是唯一的,跳跃表中的节点按照分值大小进行排序,当分值相同时,节点按照成员对象的大小进行排序.  

## 整数集合  


## 压缩列表