---
title: Mysql数据行格式
categories:
  - mysql
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:12:17
---


## InnnDB行格式
MySQL服务器上负责对表中的数据读取和写入工作的部分是**存储引擎**,我们常用的存储引擎InnoDB,真实数据在不同存储引擎中存放的格式一般是不同的,InnoDB真正处理数据的过程是发生在内存中的,所以需要把磁盘的数据加载到内存中,如果是处理写入或者修改请求的话,还需要把内存中的记录刷新到磁盘上,磁盘的读写速度和内存相比非常慢,所以InnoDB采取的方式是:**将数据划分为若干个页,以页作为磁盘和内存之间交互的基本单位,InnoDB中页的大小一般为16KB**,也就是说,一次最少从磁盘读取16KB的内容到内存中,一次最少把内存中的16KB内存刷新到磁盘中。数据是以记录插入到表中的,记录的存放方式也被称为行格式,InnoDB有4种行格式,分别是**Compact,Redundant,Dynamic,Compressed** 


### Compact行格式  


![snipaste_20201224_133417.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_133417_1608788072352.png)

#### 记录的额外信息  
这里是描述记录的元信息,分别是变长字段长度列表,NULL值列表和记录头信息.  
我们知道常用的一些变长数据类型,比如VARCHAR(M),VABINARY(M)等,变长字段存储多少字节的数据是不固定的,所以在存储真实数据的时候把数据占用的字节也存储起来. 
创建一个示例: 
```
mysql> CREATE TABLE record_format_demo (
-> c1 VARCHAR(10),
-> c2 VARCHAR(10) NOT NULL,
-> c3 CHAR(10),
-> c4 VARCHAR(10)
-> ) CHARSET=ascii ROW_FORMAT=COMPACT;
```
表中数据如下: 
![snipaste_20201224_134654.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_134654_1608788828570.png)
在Compact行格式中,把所有变长字段的真实数据占用字节长度存放在记录的开头部位,从而形成一个变长字段长度列表,各变长字段数据占用的字节数按照列的顺序逆序存放,从这个例子来看,c1、c2、c4列都是变长类型的，都采用ascii字符集，所以每个字符只需要一个字节来编码，看一下，列的长度： 
![snipaste_20201224_142251.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_142251_1608790980798.png)
查看一下记录实际效果：  
![snipaste_20201224_142251.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_142251_1608791049227.png)

对于变长类型VARCHAR（M）来说，这种类型表示能存储最多M个字符，所以这个类型能表示的字符串最多占用的字节数就是M乘以编码最大表示字节数，比如utf8是3，utf8mb64是4，**如果该可变字段允许存储的最大字节数（M × W ）超过255字节并且真实存储的字节数（L）超过127字节，则使用2个字节，否则使用1个字节。  

需要注意：变长字段长度列表只存储值为非NULL的列内容占用的长度，值为NULL的列的长度是不存储的。

#### NULL值列表 
表中某些列可能存储NULL值，如果把这些NULL值都放到记录的真实数据中存储会很占地方，所以Compact行格式把这些值为NULL的列统一管理起来，存储到NULL值列表中，处理过程如下：  

  1. 统计表中的NULL值有哪些 
  2. 如果表中没有运行存储NULL的列，则NULL值列表也不存在了，否则将每个允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列，1表示该列为null，0表示该列不为null。
  3. 规定NULL值列表必须用整个字节的位表示，如果使用的二进制位个数不是整个字节，则在字节的高位补0.


![snipaste_20201224_142251.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_142251_1608796397248.png)

#### 记录头信息
除了变长字段长度列表，NULL值列表之外，还有一个用于描述记录的记录头信息，他是由固定的5个字节组成：  
![snipaste_20201224_142251.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_142251_1608796598727.png)  

![snipaste_20201224_155719.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_155719_1608796650444.png)

#### 记录的真实数据 

MySQL会为每个记录默认的添加一些列（隐藏列），具体如下：  

![snipaste_20201224_155929.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_155929_1608796780750.png)




 InnoDB 表对主键的生成策略：优先使用用户自定义主键作为主键，如果用户没有定义主键，则选取一个 Unique 键作为主键，如果表中连 Unique 键都没有定义的话，则 InnoDB 会为表默认添加一个名row_id 的隐藏列作为主键。所以我们从上表中可以看出：InnoDB存储引擎会为每条记录都添加 transaction_id和 roll_pointer 这两个列，但是 row_id 是可选的（在没有自定义主键以及Unique键的情况下才会添加该列）。这些隐藏列的值不用我们操心， InnoDB 存储引擎会自己帮我们生成的

因为例子没有定义主键,所以会为每条记录增加上述的3个列： 
![snipaste_20201224_155929.png](https://199794.oss-cn-shanghai.aliyuncs.com/blog//snipaste_20201224_155929_1608797030169.png)
注意几点：  
1. 表使用的是 ascii 字符集，所以 0x61616161 就表示字符串 'aaaa' 0x626262 就表示字符串 'bbb' 
2. 注意第1条记录中 c3 列的值，它是 CHAR(10) 类型的，它实际存储的字符串是： 'cc' ，而 ascii 字符集中的字节表示是 '0x6363' ，虽然表示这个字符串只占用了2个字节，但整个 c3 列仍然占用了10个字节的空间，除真实数据以外的8个字节的统统都用空格字符填充，空格字符在 ascii 字符集的表示就是 0x20 
3. 注意第2条记录中 c3 和 c4 列的值都为 NULL ，它们被存储在了前边的 NULL值列表 处，在记录的真实数据处就不再冗余存储，从而节省存储空间  

#### char(M)的存储格式  
如果char(M)的字符集采用的也是变长字符集的话，也会记录到变长字段长度列表，