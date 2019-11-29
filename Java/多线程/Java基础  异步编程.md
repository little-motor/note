[toc]
# 1. 引言
同步任务的发起和执行是在同一条时间线上进行的，往往以为的阻塞，而异步任务的发起和执行在不同的时间线上。但是阻塞/非阻塞与同步/异步执行方式并没有完全的对应关系，同步/异步的说法也是相对的，他取决于任务的执行方式以及我们的观察角度。
# 2. Java Executor框架
## 2.1 Runnable、Callable接口
Runnable和Callable接口都是对任务处理逻辑对抽象，使得我们无需关注任务的具体处理逻辑。
```
//没有返回状态
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

//有返回状态
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```
## 2.2 Executor接口
Executor接口使得任务的提交能够与任务执行的具体细节解耦，能够给我们带来信息隐藏和关注点分离的好处。
```
public interface Executor {

    /**
     * 传入的Runnable实例将被执行，具体是在一个新的线程，或者线程池，
     * 或者与被调用者在同一线程，取决于Executor接口的具体实现
     */
    void execute(Runnable command);
}
```
Executor接口我们可以发现两个问题：
- 无法将任务处理结果返回给客户端
- 没有定义释放资源的方法
  
## 2.3 ExecutorService接口
ExecutorService接口继承自Executor接口，**ThreadPoolExecutor**是其默认实现，相对于上面提到的两点问题：
- 其定义了几个submit方法，这些方法能够接受Callable接口或者Runnable接口表示的任务，并返回相应的Future实例，从而使客户端在提交任务后可以获取任务执行结果
- 定义了shutdown()方法和shutdownNow()方法来释放资源
  
下面列举了一部分ExecutorService接口定义的方法
```
public interface ExecutorService extends Executor {

    // 提交一个callable实例，可以通过返回的future对象获取到返回结果
    <T> Future<T> submit(Callable<T> task);

    // 提交一个runnabel实例，执行成功返回的future get()方法返回null
    Future<?> submit(Runnable task);

    //阻塞方法，提交callable实例集合，当第一个执行成功后便返回结果，并终止其余线程执行
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    // 阻塞方法，提交callable实例集合，当所有执行成功后返回future实例集合
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    // 正在执行的任务会继续执行下去，没有被执行的则中断
    void shutdown();

    // 正在执行的任务停止，返回没被执行的任务
    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();
}
```
## 2.4 Executors实用工具类
Excecutors能够返回默认线程工厂，将Runnable实例转为Callable实例，并提供了一些能够返回ExecutorService实例的快捷方法。

方法 | 说明
---------|----------
 public static ExecutorService newCachedThreadPool() | 适用于执行大量耗时较短且提交比较频繁的任务。 
 public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) | 同上
 public static ExecutorService newFixedThreadPool(int nThreads) | 线程池大小等于最大线程池大小，该线程池中的工作者线程一般不会超时
 public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) | 同上
 public static ExecutorService newSingleThreadExecutor() | 适合单/多生产者——单消费者模式
 public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) | 同上 
- newCachedThreadPool()
  相当于new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
核心线程池大小为0，最大线程池大小不受限，工作者线程允许的最大空闲时间为60s，内部以SynchronousQueue为工作队列，但是SynchronousQueue内部并不维护用于存储队列元素的实际存储空间。在极端情况下，给该线程池每提交一个任务都会导致一个新的工作者线程被创建，因此适合适用于执行大量耗时较短且提交比较频繁的任务。
- public static ExecutorService newFixedThreadPool(int nThreads)
  相当于new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
  核心线程池大小与最大线程池大小均为nThreads，线程池中的空闲工作者线程不会被自动清理，注意使用完后需要主动关闭。

## 2.5 Future与FutureTask
无论是Runnable实例还是Callable实例所表示的任务，只要我们将其提交给线程池执行，那么这些任务就是异步任务。
- Runnable实例的优点是可以使用单独的工作线程执行，缺点是没有返回值
- Callable实例的优点是有返回值，但是缺点是只能用线程池执行，灵活性较低。

首先最基本的接口是Future，他代表了异步任务执行之后的返回信息
```
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    //阻塞方法，等任务执行完成后返回结果
    V get() throws InterruptedException, executionException;

    V get(long timeout, TimeUnit unit)throws InterruptedException, ExecutionException, TimeoutException;

}
```
FutureTask是Future接口的基本实现类，他融合了Runnable接口（继承）和Callable接口（组合）的优点，他的构造器可以将Callable实例转换为Runnable实例，同时也可以将FutureTask实例交给Executor执行。