---
title: JVM性能检测、故障检测工具
date: 2020-07-21 11:35:11
tags: JVM
categories: JVM
---

{% aplayer "REMIND YOU(LIVE)" "小林未郁" "http://fahoud.gitee.io/hexo-dist/music/小林未郁_REMIND_YOU(LIVE).mp3" "https://fahoud.gitee.io/hexo-dist/music/UCvol_3.jpg" %}

# JPS：虚拟机进程状况工具
> jps(JVM Process Status Tool)，可以列出正在运行的虚拟机进程，并显示虚拟机执行主类(Main Class，Main()函数所在的类)名称以及这些进程的本地虚拟机唯一ID(LVMID,Local Virtual Machine Identifier).

JPS主要用于查询LVMID来确定需要监控的是哪一个虚拟机进程。对于本地虚拟机进程，LVMID与操作系统的进程ID一致，但如果同时启动了多个虚拟机进程，无法根据进程名称定位时，就必须依赖jps命令显示主类的功能才能区分。

> jps命令格式：jps [ options ] [ hostid ]

|选项        |作用                              |
|------------|-----------                      |
|-q          |只输出LVMID，省略主类的名称        |
|-m          |输出虚拟机进程启动时传递给主类main()函数的参数|
|-l          |输出主类的全名，如果进程执行的是jar包，则输出jar路径|
|-v          |输出虚拟机进程启动时的JVM参数| 

> JPS执行样例：

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/JVMjps.png" width=""></div>

# jstat：虚拟机统计信息监视工具

> jstat(JVM Statistics Monitoring Tool)是用于监视虚拟机各种运行状态信息的命令行工具。可以显示本地或者远程虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据，在没有GUI图形界面、只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的常用工具。

> jstat命令格式为：jstat [ option vmid [ interval[s|ms] [count] ] ]

|选项              |作用                  |
|------            |-------               |  
|-class            |监视类加载、卸载数量、总空间以及类装载所耗费的时间|
|-gc               |监视Java堆状况，包括Eden区、2个Survivor区、老年代、永久代等的容量，已用空间，垃圾收集时间合计等信息|
|-gccapacity       |监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大，最小空间|
|-gcutil           |监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比|
|-gccause          |与-gcutil功能一样，但额外输出导致上一次垃圾收集产生的原因|
|-gcnew            |监视新生代垃圾收集状况|
|-gcnewcapacity    |监视内容与-gcnew基本相同，输出主要关注使用到的最大、最小空间|
|-gcold            |监视老年代的收集状况|
|-gcoldcapacity    |监视内容与-gcold基本相同，输出主要关注使用到的最大、最小空间|
|-gcpermcapacity   |输出永久代使用到的最大、最小空间|
|-compiler         |输出即时编译器编译过的方法、耗时等等信息|
|-printcompilation |输出已经被即时编译的方法|

> jstat执行样例：

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/JVMjstat.png" width=""></div>

-gccause说明：
- 新生代Eden区（E）使用了82.08%的空间，
- S0（Survivor区）为空，S1占比41.42%，
- 老年代（O区）使用22.7%的空间，
- 压缩类空间和元数据空间分别占比92.9%和93.64%，
- Minor GC次数一共发生215次，总耗时2.389s, Full GC触发4次，总耗时0.781s， 所有GC总耗时(GCT)为3.171s。

> -gccause的LGCC(Last GC Cause)表示上一次GC发生原因为Allocation Failure，内存分配失败，如Eden区满导致Minor GC或老年代最大连续空间无法存入大对象导致Full GC。

使用jstat工具在纯文本状态下监视虚拟机状态变化，在用户体验上不如JMC、VisualVM等可视化监视工具直接以图表形式展现的直观，但在实际生产环境中，多数服务器管理员已经习惯了在文本控制台工作，直接在控制台使用jstat命令依然是一种常用的监控方式。
