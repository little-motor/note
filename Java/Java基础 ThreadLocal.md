[toc]
参考：
https://blog.csdn.net/lufeng20/article/details/24314381
https://www.cnblogs.com/coshaho/p/5127135.html
https://www.jianshu.com/p/98b68c97df9b
# 1. 引言
ThreadLocal是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程保存需要使用的变量副本而不互相干扰。
一句话概括Synchronized与ThreadLocal的区别：Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。所以ThreadLocal的应用场合，最适合的是按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到。
数据隔离的秘诀其实是这样的，Thread有个TheadLocalMap类型的属性，叫做threadLocals，该属性用来保存该线程本地变量。这样每个线程都有自己的数据，就做到了不同线程间数据的隔离，保证了数据安全。
采用jdk1.8源码进行深挖一下TheadLocal和TheadLocalMap

![ThreadLocal原理](https://raw.githubusercontent.com/little-motor/uml/master/Java/ThreadLocal.jpg)
# 2. ThreadLocal是什么
从上面的结构图，我们已经窥见ThreadLocal的核心机制：
- 每个Thread线程内部都有一个ThreadLocal.ThreadLocalMap类型的threadLocals属性
- Map里面将ThreadLocal对象作为key，线程的变量副本为value
- 但是，Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。
  
所以对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。
# 3. 原理
Thread中的ThreadLocal.ThreadLocalMap属性
```
public
class Thread implements Runnable {
    /*...其他属性...*/
 
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
 
    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;

    ...
```

ThreadLocal类核心方法如下：
```
//用于保存当前线程的副本变量值
public void set(T value)
//用于获取当前线程的副本变量值
public T get()
//当前线程初始化副本变量值
protected T initialValue()
//移除当前线程的副本变量值
public void remove()
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier)
private T setInitialValue()

```
## 3.1 get方法
```
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 * 返回当前线程的变量副本，如果值为空，那么调用initialValue方法初始化
 * @return the current thread's value of this thread-local
 */
public T get() {
    //返回当前线程
    Thread t = Thread.currentThread();
    //getMap(t)返回t.threadLocals属性
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    //如果threadLocals为空则初始化
    //最终默认结果为threadLocals=new ThreadLocalMap(this, null)
    return setInitialValue();
}
```
步骤：
1. 获取当前线程的ThreadLocalMap对象threadLocals
2. 从map中获取线程存储的K-V Entry节点。
3. 从Entry节点获取存储的Value副本值返回。
4. map为空的话初始化map
## 3.2 set方法
```
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 * 向threadLocals存储值
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```
步骤：
1. 获取当前线程的成员变量threadLocals赋值给map
2. 若map非空，则重新将ThreadLocal和新的value副本放入到map中。
3. 若map空，则对线程的成员变量ThreadLocalMap进行初始化创建，并将ThreadLocal和value副本放入map中。












ThreadLocal，连接ThreadLocalMap和Thread。来处理Thread的ThreadLocal.TheadLocalMap类型的threadLocals属性，包括init初始化属性赋值、get对应的变量，set设置变量等。通过当前线程，获取线程上的ThreadLocalMap属性，对数据进行get、set等操作。

　　ThreadLocalMap，用来存储数据，采用类似hashmap机制，存储了以threadLocal为key，需要隔离的数据为value的Entry键值对数组结构。

　　ThreadLocal，有个ThreadLocalMap内部类，存储的数据就放在这儿。
ThreadLocal、ThreadLocal、Thread之间的关系可以概括为:ThreadLocalMap是ThreadLocal内部类，由ThreadLocal创建，Thread有ThreadLocal.ThreadLocalMap类型的threadLocals属性。


# 4. 简单用法
```
/**
 * ThreadLocal用法
 */
public class MyThreadLocal {
    private static final ThreadLocal<Object> threadLocal = new ThreadLocal<Object>() {
        /**
         * ThreadLocal没有被当前线程赋值时或当前线程刚调用remove方法后调用get方法，返回此方法值
         */
        @Override
        protected Object initialValue() {
            System.out.println("调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！");
            return null;
        }
    };

    public static void main(String[] args) {
        new Thread(new MyIntegerTask("IntegerTask1")).start();
        new Thread(new MyStringTask("StringTask1")).start();
        new Thread(new MyIntegerTask("IntegerTask2")).start();
        new Thread(new MyStringTask("StringTask2")).start();
    }

    public static class MyIntegerTask implements Runnable {
        private String name;

        MyIntegerTask(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                // ThreadLocal.get方法获取线程变量
                if (null == MyThreadLocal.threadLocal.get()) {
                    // ThreadLocal.et方法设置线程变量
                    MyThreadLocal.threadLocal.set(0);
                    System.out.println("线程" + name + ": 0");
                } else {
                    int num = (Integer) MyThreadLocal.threadLocal.get();
                    MyThreadLocal.threadLocal.set(num + 1);
                    System.out.println("线程" + name + ": " + MyThreadLocal.threadLocal.get());
                    if (i == 3) {
                        MyThreadLocal.threadLocal.remove();
                    }
                }
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    public static class MyStringTask implements Runnable {
        private String name;

        MyStringTask(String name) {
            this.name = name;
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                if (null == MyThreadLocal.threadLocal.get()) {
                    MyThreadLocal.threadLocal.set("a");
                    System.out.println("线程" + name + ": a");
                } else {
                    String str = (String) MyThreadLocal.threadLocal.get();
                    MyThreadLocal.threadLocal.set(str + "a");
                    System.out.println("线程" + name + ": " + MyThreadLocal.threadLocal.get());
                }
                try {
                    Thread.sleep(800);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
}


//运行结果
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！
线程IntegerTask1: 0
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！
线程StringTask1: a
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！
线程IntegerTask2: 0
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！
线程StringTask2: a
线程StringTask1: aa
线程StringTask2: aa
线程IntegerTask1: 1
线程IntegerTask2: 1
线程StringTask1: aaa
线程StringTask2: aaa
线程IntegerTask1: 2
线程IntegerTask2: 2
线程StringTask1: aaaa
线程StringTask2: aaaa
线程IntegerTask1: 3
线程IntegerTask2: 3
线程StringTask1: aaaaa
线程StringTask2: aaaaa
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！
调用get方法时，当前线程共享变量没有设置，调用initialValue获取默认值！
线程IntegerTask1: 0
线程IntegerTask2: 0
```
# 5. 注意事项
## 5.1 ThreadLocal弱引用问题
关于ThreadLocalMap<ThreadLocal, Object>内部entry弱引用问题：

当线程没有结束，但是entry已经被回收，则可能导致线程中存在ThreadLocalMap&lt;ThreadLocal, Object>的键值对，造成内存泄露。为了防止此类情况的出现，我们有两种手段。

1、使用完线程共享变量后，显示调用ThreadLocalMap.remove方法清除线程共享变量；

2、JDK建议ThreadLocal定义为private static，这样ThreadLocal的弱引用问题则不存在了。













# 4. ThreadLocal源代码简要分析





set设置当前线程对应的线程局部变量的值。先取出当前线程对应的threadLocalMap，如果不存在则用创建一个，否则将value放入以this，即threadLocal为key的映射的map中，其实threadLocalMap内部和hashMap机制一样，存储了Entry键值对数组，后续会深挖threadLocalMap。

initialValue返回该线程局部变量的初始值。该方法是一个protected的方法，显然是为了让子类覆盖而设计的。这个方法是一个延迟调用方法，在线程第1次调用get()或set(Object)时才执行，并且仅执行1次。ThreadLocal中的缺省实现直接返回一个null。

withInitial提供一个Supplier的lamda表达式用来当做初始值，java8引入。

setInitialValue设置初始值。在get操作没有对应的值时，调用此方法。private方法，防止被覆盖。过程和set类似，只不过是用initialValue作为value进行设置。



get该方法返回当前线程所对应的线程局部变量。和set类似，也是先取出当前线程对应的threadLocalMap，如果不存在则用创建一个，但是是用inittialValue作为value放入到map中，且返回initialValue，否则就直接从map取出this即threadLocal对应的value返回。

remove将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是JDK 5.0新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。需要注意的是，如果remove之后又调用了get，会重新初始化一次，即再次调用initialValue方法，除非在get之前调用set设置过值。






