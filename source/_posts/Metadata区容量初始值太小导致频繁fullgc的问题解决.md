---
title: Metadata区容量初始值太小导致频繁fullgc的问题解决
date: 2020-7-26 14:10:43
tags: JVM
categories: JVM
---


# 问题描述
之前在修改过运行参数的时候发现，jar包每一次重新运行起来都至少要经过三次的Full GC才能正常运行

在问题解决之前的jvm运行参数为，JDK8：
> -server 
-Xms512m    
-Xmx512m   
-XX:+PrintGCDetails    
-XX:+PrintGCDateStamps     
-Xloggc:/usr/local/xx/gclogs/gc.log

Full GC原始问题

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/fullgcjstat.png" width=""></div>

堆容量设置为512M，NewRatio参数默认为2（新生代与老年代比例为1:2），意味着老年代起码有三百多兆，才刚启动应用按理来说不会这么容易fullgc

# 原因描述

查看GC日志，发现这三次FullGC发生的原因都是因为[Full GC (Metadata GC Threshold)]

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/fullgclog.png" width=""></div>

## 啥是Metadata GC Threshold？
根据~~百度翻译~~可知，Threshold指阀、界，所以发生FullGC的原因是元数据区扩容，通过jstat可以知道，元数据区容量(MC)为84504.0kb，也就是85M左右

> 通过参数-XX:+PrintFlagsInitial来获取jvm初始参数

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/fullgcmeta.png" width=""></div>

由此可知元数据区默认初始容量为20M左右，最大元数据区容量默认为18446744073709551615字节（假设无穷大），而应用程序从启动到启动完毕元数据区至少需要85兆字节的内存，所以应用启动完毕至少经过了三次的FULLGC

# 解决
## -XX:MetaspaceSize=N
这个参数是初始化的Metaspace大小，这个值越大触发元数据区gc的时机就越晚。这个值在不同的平台上的默认值会有所不同，一般默认值在12M到20M浮动，只要把元数据区的初始容量大小调高，应用在启动过程中就不会因为元数据区容量不足而导致FullGC

> 给应用添加启动参数  
-XX:MetaspaceSize=128M  
-server   
-Xms512m    
-Xmx512m   
-XX:+PrintGCDetails    
-XX:+PrintGCDateStamps     
-Xloggc:/usr/local/xx/gclogs/gc.log

FullGC消失了

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/fullgcresolve.png" width=""></div>
