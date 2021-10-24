---
title: SSO
categories:
  - SSO
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:03:45
---

## 实现机制
当用户第一次访问应用系统1的时候,因为还没有登录,会被引导到认证系统进行登录:根据用户提供的登录信息,认证系统进行身份验证,如果通过校验,应该返回给用户一个认证的凭据---ticket;用户再访问别的应用的时候就会将这个ticket带上,作为自己的凭据,应用系统接收到请求之后会把ticket送到认证系统进行校验,检验ticket的合法性,如果通过校验,用户可以在不用再次登录的情况下访问应用系统2等..  

要实现SSO, 需要以下主要的功能:  

### CAS

CAS是central Authentication Service的缩写,中央认证服务,一种独立开放指令协议,是开源的企业级单点登录解决方案.  

CAS 包括两部分:CAS Server和CAS Client 

CAS Server负责完成对用户的认证工作,会为用户签发两个重要的票据:登录票据(TGT)和服务票据(ST)来实现认证过程,CAS Server需要独立部署.  

CAS Client 负责处理对客户端受保护资源的访问请求,需要对请求方进行身份认证时,重定向到CAS Server进行认证,准确的说,Client一般会以拦截器实现保护资源,对于访问受保护资源的每个web请求,CAS Client会解析请求是否包含Service Ticket(服务票据)  

#### 核心票据  
CAS的核心就是Ticket,CAS的主要票据有TGT,ST,PGT,PGTIOU,PT,其中TGT,ST是CAS1.0协议中就有的票据,PGT,PGTIOU,PT是CAS2.0协议中有的票据.  

##### TGT 

TGT是CAS为用户签发的登录票据,拥有了TGT,用户就可以证明自己在CAS成功登录过,TGT封装了cookie值以及Cookie值对应的用户信息,用户在CAS认证成功后,生成一个TGT对象,放入自己的缓存,可以是session或Redis,同时CAS生成cookie(其实就是TGT的sessionId) 或者生成一个token给浏览器(token可以跨域),当http再次请求到来时,如果传来有CAS生成cookie,则CAS以此seesionId为key查询缓存中有无TGT,如果有的话,则说明用户之前登录过,如果没有,则用户需要重新登录.  

##### TGC 

CAS Server 生成TGT放入自己的缓存中,而TGC就是这个缓存的唯一标识  

##### ST 
ST是CAS为用户签发的访问某一服务票据,用户访问service时,用户访问service时,service发现用户没有ST,则要求用户去CAS获取ST,用户向CAS发出获取ST的请求,如果用户的请求包含cookie,则CAS会以此cookie值为key查询缓存中有无TGT,如果存在TGT,则用此TGT签发一个ST,返回给用户.用户凭借ST去访问service,service拿ST去CAS验证,验证通过后,允许用户访问资源.   

为了保证ST的安全性,ST是基于随机生成的,而且CAS规定ST只能存活一定的时间,而且CAS协议规定ST只能使用一次,无论Service Ticket验证是否成功,CAS Service 都会清楚服务端中的该Ticket,从而可以确保一个Service Ticket不被使用两次 .  

https://blog.csdn.net/wang379275614/article/details/46337529
 
 
