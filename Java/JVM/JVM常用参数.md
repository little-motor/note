[toc]
# 1. 引言
VM使用的一些常用参数，方便随时查询使用
# 2. 查询
```
#VM详细设置参数
java -XX:+PrintFlagsFinal -version 
#VM简要设置参数
java -XX:+PrintCommandLineFlags -version
#GC详细信息
java -XX:+PrintGCDetails -version
```
# 3. 内存设置
```
#设置初始化堆大小为6m（也可以设置为k，下同）
-Xms6m
#设置最大的堆内存大小为80m
-Xmx80m
#设置栈大小为10m
-Xss10m
#触发元空间GC的阈值
-XX:MaxMetaspaceSize
#元空间最大值
-XX:MaxMetaspaceSize
#直接内存大小（如不指定默认同堆最大值）
-XX:MaxDirectMemorySize
```
# 4. 垃圾收集设置
## 4.1 ParNew(新生代垃圾收集器)
```
#使用ParNew和CMS组合
-XX:+UseConcMarkSweepGC
#强制使用ParNew
-XX:+UseParNewGC
#限制垃圾收集的线程数
-XX:ParallelGCThreads
```
## 4.3 G1收集器（全代）


选项和默认值 | 描述
---------|----------
 -XX:+UseG1GC | 使用垃圾优先(G1,Garbage First)收集器
 -XX:MaxGCPauseMillis=n | 设置垃圾收集暂停时间最大值指标。这是一个软目标，Java虚拟机将尽最大努力实现它
 -XX:InitiatingHeapOccupancyPercent=n | 触发并发垃圾收集周期的整个堆空间的占用比例。它被垃圾收集使用，用来触发并发垃圾收集周期，基于整个堆的占用情况，不只是一个代上(比如：G1)。0值 表示’do constant GC cycles’。默认是45
-XX:NewRatio=n 	| 年轻代与年老代的大小比例，默认值是2
-XX:SurvivorRatio=n  |	eden与survivor空间的大小比例，默认值8
-XX:MaxTenuringThreshold=n | 	最大晋升阈值，默认值15
-XX:ParallerGCThreads=n  |	设置垃圾收集器并行阶段的线程数量。默认值根据Java虚拟机运行的平台有所变化
-XX:ConcGCThreads=n  |	并发垃圾收集器使用的线程数量，默认值根据Java虚拟机运行的平台有所变化
-XX:G1ReservePercent=n  |	为了降低晋升失败机率设置一个假的堆的储备空间的上限大小，默认值是10
-XX:G1HeapRegionSize=n 	|  使用G1收集器，Java堆被细分成一致大小的区域。这设置个体的细分的大小。这个参数的默认值由工学意义上的基于堆的大小决定
```
# 使用G1收集器
-XX:+UseG1GC
# 设置垃圾收集暂停时间最大值指标。这是一个软目标，Java虚拟机将尽最大努力实现它
-XX:MaxGCPauseMillis=200
```
# 5. 垃圾收集器参数总结

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