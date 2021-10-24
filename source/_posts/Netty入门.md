---
title: Netty入门
categories:
  - Netty
index_img: >-
  https://199794.oss-cn-shanghai.aliyuncs.com/blog//2019-04-22%20135447_gaitubao_1600x900_1604366529483.jpg
date: 2021-10-24 15:05:35
---

## 同步与异步  
同步与异步关注的是消息通信机制,同步就是在发出一个调用时,在没有得到结果之前,该调用就不返回,但是一旦调用返回,就得到返回值了  
而异步相反,调用在发出之后,这个调用就立即返回了,所以没有返回结果,当一个异步过程调用发出之后,调用者不会立即得到结果,而是在调用发出之后,被调用者通过状态,通知来通知调用者,或者通过回调函数来处理这个调用  

## 阻塞与非阻塞  
阻塞与非阻塞关注的是程序在等待调用结果时的状态  
阻塞调用是指调用结果返回之前,当前线程会被挂起,调用线程只有在得到结果之后才会返回.  
非阻塞调用指的是不能立即得到结果之前,该调用不会阻塞当前线程.  

## NIO  
![Snipaste_20210523_233112.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-05-23_23-31-12_1621783890886.png)  


### 事件和Channel
Netty的时间按照入站或出站数据流的相关性进行分类,可能由入站数据或者相关的状态更改而触发的事件包括:  
- 连接已被激活或者连接失活;
- 数据读取
- 用户事件
- 错误事件

出站事件是未来将会触发的某个操作结果: 
- 打开或者关闭到远程借点的链接
- 将数据冲刷到套接字

Netty的事件可以分发给ChannelHandler处理:  
![Snipaste_20210614_113232.png](http://oss.xiaokoua.cn/blog//Snipaste_2021-06-14_11-32-32_1623643897654.png)  

### 选择器,事件和EventLoop  
在内部,Netty将会为每个Channel分配一个EventLoop,用以处理所有事件,包括: 
- 注册感兴趣的事件
- 将事件派发给ChannelHandler
- 安排进一步的动作

### ChannelPipeline接口 
ChannelPipline提供了ChannelHandler链的容器,当Channel被创建时,它会被自动的分配到它专属的ChannelPipeline. Channelhandler安装到ChannelPipeline中的过程如下:  
- 一个ChannelInitializer的实现被注册到了ServerBootstrap中;
- 当ChannelInitiazer.initChannel()方法被调用时,ChannelInitializer将在ChannelPipeline中安装一组自定义的ChannelHandler;
- ChannelInitializer将它自己从ChannelPipeline中移除
