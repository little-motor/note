[toc]
# 1. 引言
线程组相比于线程，重点会从如何利用线程“做到”我们想做的事情，转变为“如何做的更好”，主要问题包括：线程运行过程中一旦抛出未捕获异常我们如何得知并应对的可靠性问题；如何提高线程利用率；如何将这些线程统一管理，从“散兵游勇”提升为“正规军”。
# 2. 线程组
线程组ThreadGroup类可以用来表示一组相似或相关的线程，线程与线程组的关系可以类比为文件和文件夹的关系————一个文件总是位于特定的文件夹之中，而一个文件夹可以包含多个文件和文件夹。同时，线程组之间可以存在父子关系。多数情况下我们可以忽略这一概念。
# 3. 可靠性：线程的未捕获异常与监控
Thrad类内部有一个未捕获异常接口，用于处理线程的run方法抛出未被捕获的异常(UncaughtException)
```java
    @FunctionalInterface
    public interface UncaughtExceptionHandler {
        /**
         * Method invoked when the given thread terminates due to the
         * given uncaught exception.
         * <p>Any exception thrown by this method will be ignored by the
         * Java Virtual Machine.
         */
        void uncaughtException(Thread t, Throwable e);
    }
```
这是一种JVM捕获线程异常的机制，当线程运行过程中抛出未捕获异常时，JVM首先会去调用线程关联的UncaughtExceptionHandler实例，如果没有会去调用关联的线程组（线程组也实现了这个接口），如果没有就调用getDefaultUncaughtExceptionHandler。
# 4. 线程工厂
在JDK1.5开始，Java标准库本身就支持创建线程的工厂方法，ThreadFactory接口是工厂方法设计模式的一个实例。使用new方法好比手工制作，ThreadFactory的newThread好比工厂采用标准化的流程进行生产，同时将生产和使用分离。
```
public interface ThreadFactory {

    /**
     * Constructs a new {@code Thread}.  Implementations may also initialize
     * priority, name, daemon status, {@code ThreadGroup}, etc.
     *
    Thread newThread(Runnable r);
}
```
# 5. 线程的高效利用：线程池
线程是一个昂贵的资源，其开销主要有以下几个方面：
- 线程的创建与启动开销，与普通对象相比，Java线程还占用额外的存储空间————栈空间
- 线程调度时上下文切换的开销
- 线程销毁开销
## 5.1 线程池简述
线程池本身也是一个对象，客户端提交任务并缓存在工作队列中，线程池内部的各个工作者线程不断的消费任务。java.util.ThreadPoolExecutor类就是一个线程池，客户端代码可以调用ThreadPoolExecutor.submit方法向其提交任务。
```java
  Future<?> submit(Runnable task);
```
线程池内部维护的工作者线程的数量被称为该线程池的线程池大小，其中有三种类型状态：
- Current Pool Size 当前线程池大小
- Core Pool Size 核心线程池大小
- Maximum Pool Size 最大线程池大小
他们三者的大小关系依此递增。
以下面一个参数最多的构造器为例
```java
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) 
                              {...}
                        
```
- corePoolSize和maximumPoolSize前面已经讲过
- keepAliveTime和unit合在一起用于指定线程池中空闲（Idle）线程的最大存活时间
- workQueue是被称为工作队列的阻塞队列，他相当于生产者-消费者中的传输通道
- threadFactory指定用于创建工作者线程的线程工厂
- handler用于工作队列满之后，提交任务抛出的RejectedExecution。
## 5.2 RejectedExecutionHandler
当线程池大小达到核心线程池大小时，新提交的任务会存入工作队列，缓存的任务会被工作者线程取出并消费，当工作队列满后，线程池会继续创建新的线程（这部分属于非核心线程），直到达到最大线程数。当线程池饱和（Saturated）时，即工作队列满并且当前线程数达到最大线程数时，客户端试图提交当任务会被拒绝并抛出异常，线程池关联的RejectedExecutionHandler的rejectedExecution方法会被调用。
```java
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```
TheradPoolExecutor自身提供几种实现

| 实现类                                 | 所实现的处理策略                                         |
| -------------------------------------- | -------------------------------------------------------- |
| ThreadPoolExecutor.AbortPolicy         | 直接抛出异常（默认）                                     |
| ThreadPoolExecutor.DiscardPolicy       | 丢弃当前拒绝的任务（不抛出异常）                         |
| ThreadPoolExecutor.DiscardOldestPolicy | 将工作队列中最老的任务丢弃，然后重新尝试接纳被拒绝的任务 |
| ThreadPoolExecutor.CallerRunsPolicy    | 在客户端线程中执行被拒绝的任务                           |
## 5.3 多余线程生存时间
在当前线程池大小超过线程池核心大小的时候，超过线程池核心大小的部分工作者线程空闲（即工作者队列中没有待处理的任务）时间达到keepAliveTime所指定的时间后会被清理掉。
## 5.4 关闭线程池
- shutdown() 已提交的任务会继续执行，不可继续提交新的任务
- awaitTermination(long timeout, TimeUnit unit) 等待一定时间来关闭线程池
- shutdownNow() 立即结束当前线程池内部的任务
## 5.5 任务的处理结果、异常处理
ThreadPoolExecutor的父接口提供了提交Callable实例的方法，通过返回的Future未来任务，可以查询其返回结果。由下来列出的三个submit方法可以发现，提交的任务可以为Callable实例或Runnable实例。
简单介绍一下Callabel，更多内容可以前往[异步任务与Executor框架](https://blog.csdn.net/qq_39385118/article/details/102826341)。
Callable接口也是对任务的抽象，任务的处理逻辑在call方法中实现，call方法的返回值代表相应任务的处理结果，其执行的任务可以抛出异常并在future.get()中感知。Future接口实例可以看作是提交给线程池执行的任务的处理结果句柄（Handle），其常用的几个方法如下
```java
   //方法可以获取任务的执行结果，如果任务抛出异常，get也会抛出异常
   V get() throws InterruptedException, ExecutionException;

   //指定获取结果的等待时间
   V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;

   //来判断任务是否结束，需要注意的是正常执行或抛出异常时，都会让isDone返回true。
   boolean isDone();     

   //mayInterruptIfRunning用于设置是否给执行线程发送中断，不管mayInterruptIfRunning的值是true还是false，
   //如果任务还没有开始执行，那么就会停止掉。如果任务已经执行了。那么下次任务就不会执行了。
   //但是如果任务里面有用到while (!Thread.interrupted())
   //那么本次任务会一直执行，只有mayInterruptIfRunning=true马上中断线程才能停止任务。
   boolean cancel(boolean mayInterruptIfRunning);

   boolean isCancelled();
  get()。同时我们可以通过futere.isDone()

```
同时Executors.callable(Runnable task, T result)能够将Runnable接口转换为Callable接口实例。
```java
    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);
```
需要注意的是当调用Future.get()方法时，如果任务未执行完毕，获取该处理结果会导致等待并由此引起上下文切换，所以应该尽可能早的提交任务，尽可能晚的获取处理结果，或者用Future.isDone()判断是否完成。