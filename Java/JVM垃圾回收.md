[toc]
# 1. 引言
垃圾收集器主要关注的是堆和方法区中的垃圾收集。
# 2. 判断对象是否存活的依据
## 2.1 引用计数法
其方法是给对象添加一个引用计数器，每当有一个地方引用他时，计数器加1，当引用失效时，计数器值减1，但是这个方法存在一个问题就是互相引用时，计数器当值永远不为0。
## 2.2 可达性分析算法
通过一系列称为“GC Roots”的对象作为起始点，一个对象到GC Roots的路径称为“引用链”（Reference Chain）当一个对象到GC Roots不存在任何引用链时，就会判定他为可回收对象。
可作为GC Roots的对象包括：
1. 虚拟机栈中引用的对象
2. 方法区中类静态属性引用的对象
3. 方法区中常量引用的对象
4. 本地方法栈中引用的对象
# 3. 引用类型
引用分为强引用(Strong Reference)、软引用(Soft Reference)、弱引用(Weak Reference)和虚引用(Phantom Reference)4种。  
- 强引用类似"Object obj = new Object()"，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。
- 软引用用来描述一些还有用但非必需的对象，对于软引用关联的对象，在系统发生内存溢出之前，会把这些对象列进回收范围进行二次回收，相关的类为SoftReference
- 弱引用用来描述非必需的对象，被弱引用关联的对象只能生存到下一次垃圾收集发生之前，相关的类为WeakReference 
- 虚引用也称为幽灵引用或者幻影引用，是最弱的一种引用关系，相关类为PhantomReference。
# 4. 垃圾收集算法
此处不探讨实现细节，只是讲一下算法的思想
## 4.1 标记——清除(Mark-Sweep)算法
简而言之就是先标记需要回收的内存片段，然后回收。他有两点不足：首先是效率不高，其次是会产生大量不连续的内存碎片。
## 4.2 复制算法（新生代）
将内存容量划分为相等的两块，每次只使用其中一块，当使用完以后就将还存活的对象复制到另一块内存中并将原来的那块内存清除。有研究发现超过百分之90以上的新生代对象都是“朝生夕死”，所以没有必要1:1的划分为两块相等的内存区域，在HopSpot虚拟机中，Eden与两块Survivor的大小比例是8:1:1，每次回收时将Eden和一块Survivor中还存活的对象一次性复制到另一块Survivor空间中，这样只有10%的内存被“浪费”。
## 4.3 标记——整理算法（老年代）
在存活率较高的老年代并不适合使用复制算法，在老年代使用的垃圾收集策略是让所有存活的对象都在内存中向一端移动并靠拢，清后清除掉边界以外的内存。
## 4.4 分代收集算法
综上所述，根据不同的对象存活周期，将内存划分为好几块，在新生代中使用复制算法，在老年代中使用“标记-整理”或者“标记—清除”算法进行垃圾回收。
# 5. 垃圾收集器
每种垃圾收集器都有最适合的应用场景。
![垃圾收集器](https://raw.githubusercontent.com/little-motor/uml/master/Java/jvm/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.png)
<center>HotSpot虚拟机的垃圾收集器</center>
## 5.1 Serial收集器(新生代)
他是最悠久的收集器，一种单线程收集器，同时在垃圾收集时会暂停所有的工作线程，直到收集结束。他是Client模式下的默认新生代收集器。

![Serial + Serial Old](https://raw.githubusercontent.com/little-motor/uml/master/Java/jvm/Serial%20%2B%20Serial%20Old.jpg)

<center>Serial + Serial Old</center>

## 5.2 ParNew收集器（新生代）
ParNew收集器是Serial收集器的多线程版本，同时能够与CMS（Concurrent Mark Sweep），ParNew收集器也是使用<code>-XX:+UseConcMarkSweepGC</code>选项后默认新生代收集器，也可以使用<code>-XX:+UseParNewGC</code>选项来强制使用他。
需要注意的是ParNew收集器由于线程切换的开销，所以在单核或者核心较少的情况下，对比Serial收集器并没有优势。可以使用<code>-XX:ParallelGCTHreads</code>参数来限制垃圾收集的线程数。
## 5.3 Parallel Scavenge（新生代）
他使用的也是复制算法，同时也是并行多线程收集器，只是他与ParNew的关注点并不相同，CMS等垃圾收集器是尽可能缩短垃圾收集器收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控的吞吐量（Throughput)。
所谓吞吐量=用户代码运行时间/（用户代码运行时间 + 垃圾收集运行时间），其中有两个参数需要关注一下：
```
# 一个大于0的毫秒数，收集器尽可能保证内存收集花费的时间不超过设定值
-XX:MaxGCPauseMillis
# 假设GCTimeRatio的值为n，那么系统将花费不超过1/(1+n)的时间用于垃圾收集
-XX:GCTimeRatio
```

![Parallel Scavenge + Parallel Old](https://raw.githubusercontent.com/little-motor/uml/master/Java/jvm/Parallel%20Scavenge%20%2B%20Parallel%20Old.jpg)

<center>Parallel Scavenge + Parallel Old</center>

## 5.4 Serial Old收集器（老年代）
Serial Old是Serial收集器的老年代版本，同样是一个单线程收集器，使用“标记——整理”算法，主要是给Client模式下的虚机使用，或者是CMS发生Concurrent Mode Failure时作为后备预案。
## 5.5 Parallel Old收集器（老年代）
使用多线程和“标记——整理”算法，与Parallel Scavenge搭配使用，注重“吞吐量优先”。
## 5.6 CMS收集器（老年代）
Concurrent Mark Sweep是一种以获取最短回收停顿时间为目标的垃圾收集器，非常适合重视响应速度的情况。CMS收集器是基于“标记——清楚”算法实现的，整个过程分为4步：
- 初始标记（Stop The World）
- 并发标记 (与用户线程一起并发工作)
- 重新标记（Stop The World）
- 并发清除（与用户线程一起并发工作）
从总体上来看，CMS收集器的内存回收过程是与用户线程一起并发执行的，所以停顿时间较短。但是CMS也存在3个明显的问题：
1. 对CPU资源敏感，CMS的默认启用线程数为（CPU数量 + 3）/ 4，当CPU数量少于4个时CMS占用当cpu资源反倒会比较大，影响吞吐量。
2. CMS无法处理浮动垃圾（Floating Garbage），可能出现"Concurrent Mode Failure"失败而导致另一次Full GC的产生。浮动垃圾是指在CMS并发清理阶段产生的新垃圾，这些垃圾只能由下次的并发清理来执行。<code>-XX:CMSInitiatingOccupancyFraction</code>的值来设置老年代内存占用率来触发CMS垃圾收集。这个值如果设置的太高就容易由于浮动垃圾太多而导致"concurrent Mode Failure",这时就会通过Serial Old收集器来重新进行老年代垃圾收集，产生长时间停顿。
3. “标记——清除”算法容易产生大量的空间碎片。当老年代还有很大当空间剩余但是无法找到足够大当连续空间来分配当前对象时，不得不触发一次Full GC。CMS提供如下两个命令来优化这个问题
```
# 在进行FullGC时开启内存碎片合并整理
-XX:+UseCMSCompactAtFullCollection(默认开启)
# 执行n次不压缩的Full GC后，跟着来一次带压缩的Full GC
-XX:CMSFullGCsBeforeCompaction 
```

![ParNew + CMS + Serial Old](https://raw.githubusercontent.com/little-motor/uml/master/Java/jvm/Parallel%20Scavenge%20%2B%20Parallel%20Old.jpg)

<center>ParNew + CMS + Serial Old</center>
## 5.7 G1收集器
G1垃圾收集器是在1.7版本中加入的，他的目标是在多核环境中缩短Stop-The-World停顿时间。他有如下几个特点：
- G1可以不与其他收集器配合就可以独立管理整个GC堆
- 采用“标记-整理”算法，有效减少内存空间碎片
- 可预测的停顿，能尽可能控制垃圾收集停顿时间在指定的时间内
  
G1将整个Java堆划分为多个大小相等的独立区域（Region），新生代、老年代都是一部分Region（不需要连续）的集合，这要他可以避免在Java堆中进行全区域的垃圾收集，并且优先收集回收价值最大的Region，这也是他名称的来由。G1收集器的运作大致可划分为以下几个步骤：
1. 初始标记(Initial Marking)
2. 并发标记(Concurrent Marking)
3. 最终标记(Final Marking)
4. 筛选回收(Live Data Counting and Evacuation)

![G1](https://raw.githubusercontent.com/little-motor/uml/master/Java/jvm/G1.jpg)
# 6. 垃圾收集器参数总结

参数 | 描述
--------|----------
 UseSerialGC | 虚拟机运行在Client模式下的默认值，打开此开关后，使用Serial + Serial Old的收集器组合进行内存回收
 UseParNewGC | 打开此开关后，使用ParNew + Serial Old的收集器组合进行内存回收
 UseConcMarkSweepGC | 打开此开关后，使用ParNew+ CMS + Serial Old的收集器组合进行内存回收。Serial Old收集器将作为CMS收集器出现Concurrent Mode Failure失败后的后备收集器使用
 UseParallelGC | 虚拟机运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge + Serial Old (PS Mark Sweep)的收集器组合进行内存回收
 UserParallelOldGC | 打开此开关后，使用Parallel Scavenge + Parallel Old的收集器组合进行内存回收
 SurvivorRatio | 新生代中Eden区域与Survivor区域的容量比值，默认为8，代表Eden: Survivor = 8:1
 PretenureSizeThreshold | 直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配
 MaxTenuringThreshold | 晋升到老年代的对象年龄。每个对象在坚持过一次Minor GC之后，年龄就增加1，当超过这个参数值时就进入老年代
 UseAdaptiveSizePolicy | 动态调整Java堆中各个区域的大小以及进入老年代的年龄
 HandlePromotionFailure | 是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden和Survivor区的所有对象都存活的极端情况
 ParallelGCThreads | 设置并行GC时进行内存回收的线程数
 GCTimeRatio | GC时间占总时间的比率，默认值是99， 即允许1%的GC时间。仅在使用Parallel Scavenge收集器时生效
 MaxGCPauseMillis | 设置GC的最大停顿时间。仅在使用Parallel Scavenge收集器时生效
 CMSInitiatingOccupancyFraction | 设置CMS收集器在老年代时间被使用多少后触发垃圾收集。默认值为68%，仅在使用CMS收集器时生效
 UseCMSCompactAtFullCollection | 设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片整理。仅在使用CMS收集器时生效
 CMSFullGCsBeforeCompaction | 设置CMS收集器在进行若干次垃圾收集后再启动一次内存碎片整理，仅在使用CMS收集器时生效