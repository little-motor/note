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