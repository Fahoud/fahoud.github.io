---
title: JVM MinorGC和FullGC触发条件
date: 2020-7-17 10:45:38
tags: JVM
categories: JVM
---


# Java垃圾回收机制
GC，即就是Java垃圾回收机制。目前主流的JVM（HotSpot）采用的是分代收集算法。作为Java开发者，一般不需要专门编写内存回收和垃圾清理代码，对内存泄露和溢出的问题。与C++不同的是，Java采用的是类似于树形结构的可达性分析法来判断对象是否还存在引用。即：从gcroot开始，把所有可以搜索得到的对象标记为存活对象。

> 具体过程：

当GC线程启动时，会通过可达性分析法把Eden区和From Survivor区的存活对象复制到To Survivor区，然后把Eden Space和From Survivor区的对象释放掉。当GC轮训扫描To Survivor区一定次数后，把依然存活的对象复制到老年代，然后释放To Survivor区的对象。

对于用可达性分析法搜索不到的对象，GC并不一定会回收该对象。要完全回收一个对象，至少需要经过两次标记的过程：

- 第一次标记：对于一个没有其他引用的对象，筛选该对象是否有必要执行finalize()方法，如果没有执行必要，则意味可直接回收。

>（筛选依据：是否复写或执行过finalize()方法；因为finalize方法只能被执行一次）。

- 第二次标记：如果被筛选判定位有必要执行，则会放入FQueue队列，并自动创建一个低优先级的finalize线程来执行释放操作。如果在一个对象释放前被其他对象引用，则该对象会被移除FQueue队列。

> finalize()只能执行一次，意味着对象只能自救一次，触发下一次gc是若该对象未被标记则会直接清除

# 虚拟机中GC的过程

> 在初始阶段，新创建的对象被分配到Eden区，survivor的两块空间都为空。
<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc1.png" width=""></div>

> 当Eden区满了的时候，触发MinorGC

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc2.png" width=""></div>

> 经过扫描与标记，未被标记的对象从Eden区清除，存活的对象复制或移动到From Survivor区

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc3.png" width=""></div>

>  在下一次的Minor GC中，Eden区的情况和上面一致，没有引用的对象被回收，存活的对象被复制到From survivor区。然而在From Survivor区，From Survivor的所有的数据都被复制到To Survivor，需要注意的是，在上次minor GC过程中移动到From Survivor中的两个对象在复制到To Survivor后其年龄要加1。此时Eden区From Survivor区被清空，所有存活的数据都复制到了To Survivor区，并且To Survivor区存在着年龄不一样的对象

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc4.png" width=""></div>

> 再下一次MinorGC则重复这个过程，这一次survivor的两个区对换，存活的对象被复制到S0，存活的对象年龄加1，Eden区和另一个survivor区被清空

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc5.png" width=""></div>

> 再经过几次Minor GC之后，当存活对象的年龄达到一个阈值之后（可通过参数配置，默认是8），就会被从年轻代Promotion到老年代

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc6.png" width=""></div>

> 随着MinorGC一次又一次的进行，不断会有新的对象被promote到老年代


<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc7.png" width=""></div>

> 上面基本上覆盖了整个年轻代所有的回收过程。最终，MajorGC将会在老年代发生，老年代的空间将会被清除和压缩

<div align=center>
<img src="http://fahoud.gitee.io/hexo-dist/jvm/jvmgc8.png" width=""></div>

从上面的过程可以看出，Eden区是连续的空间，且Survivor总有一个为空。经过一次GC和复制，一个Survivor中保存着当前还活着的对象，而Eden区和另一个Survivor区的内容都不再需要了，可以直接清空，到下一次GC时，两个Survivor的角色再互换。因此，这种方式分配内存和清理内存的效率都极高，这种垃圾回收的方式就是著名的“停止-复制（Stop-and-copy）”清理法（将Eden区和一个Survivor中仍然存活的对象拷贝到另一个Survivor中），这不代表着停止复制清理法很高效，其实，它也只在这种情况下（基于大部分对象存活周期很短的事实）高效，如果在老年代采用停止复制，则是非常不合适的。

老年代存储的对象比年轻代多得多，而且不乏大对象，对老年代进行内存清理时，如果使用停止-复制算法，则相当低效。一般，老年代用的算法是标记-压缩算法，即：标记出仍然存活的对象（存在引用的），将所有存活的对象向一端移动(<i>标记-整理算法</i>)，以保证内存的连续。

在发生Minor GC时，虚拟机会检查每次晋升进入老年代的大小是否大于老年代的剩余空间大小，如果大于，则直接触发一次Full GC，否则，就查看是否设置了-XX:+HandlePromotionFailure（允许担保失败），如果允许，则只会进行MinorGC，此时可以容忍内存分配失败；如果不允许，则仍然进行Full GC（这代表着如果设置-XX:+Handle PromotionFailure，则触发MinorGC就会同时触发Full GC，哪怕老年代还有很多内存，所以，最好不要这样做）。

关于方法区即永久代的回收，永久代的回收有两种：常量池中的常量，无用的类信息，常量的回收很简单，没有引用了就可以被回收。对于无用的类进行回收，必须保证3点：

> 1. 类的所有实例都已经被回收。
> 2. 加载类的ClassLoader已经被回收。
> 3. 类对象的Class对象没有被引用（即没有通过反射引用该类的地方）。

永久代的回收并不是必须的，可以通过参数来设置是否对类进行回收。

# Minor GC ，Full GC 触发条件

## Minor GC触发条件：
> 当Eden区满时，触发Minor GC。

## Full GC触发条件：
>（1）调用System.gc时，系统建议执行Full GC，但是不必然执行  
>（2）老年代空间不足  
>（3）方法去空间不足  
>（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存  
>（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。
