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