[toc]
# 1. 引言
Queue： 基本上，一个队列就是一个先入先出（FIFO）的数据结构，优先队列除外其更具comparator或者comparable实现进行比较。
Queue接口与List、Set同一级别，都是继承了Collection接口。LinkedList实现了Deque接口。
在并发队列上JDK提供了两套实现，一个是以ConcurrentLinkedQueue为代表的高性能队列非阻塞，一个是以BlockingQueue接口为代表的阻塞队列，无论哪种都继承自Queue。

在最后两节会深入当了解两个queue的实现类，PriorityQueue和ConcurrentLinkedQueue。

Queue家族当继承关系如下所示：

![继承关系](https://github.com/little-motor/uml/blob/master/Java/dataStructure/queue%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB?raw=true)
# 2. 阻塞队列与非阻塞队列
阻塞队列与普通队列的区别在于，当队列是空的时，从队列中获取元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其他的线程往空的队列插入新的元素。同样，试图往已满的阻塞队列中添加新元素的线程同样也会被阻塞，直到其他的线程使队列重新变得空闲起来，如从队列中移除一个或者多个元素，或者完全清空队列。
# 3. 子类简述
## 3.1 未实现阻塞接口的
- LinkedList : 实现了Deque接口，受限的队列
- PriorityQueue ： 优先队列，本质维护一个有序列表（内部维护一个最小堆的一维数组）。可自然排序亦可传递 comparator构造函数实现自定义排序。
- ConcurrentLinkedQueue：基于链表 线程安全的队列。增加删除O(1) 查找O(n)
## 3.2 实现阻塞接口的
实现blockqueue接口的五个阻塞队列，其特点：当队列满或者空时，线程阻塞，直到到有空间或者插入元素时，才进行操作。
- ArrayBlockingQueue： 基于数组的有界队列
- LinkedBlockingQueue： 基于链表的无界队列
- ProiporityBlockingQueue：基于优先次序的无界队列
- DelayQueue：基于时间优先级的队列
- SynchronousQueue：内部没有容器的队列 较特别 --其独有的线程一一配对通信机制

# 4. 基本操作方法
操作 | 抛出异常 | 返回特定值
---------|----------|---------
 入队 | add(e) | offer(e)——false
 出队 | remove() | poll()——null
 检查 | element() | peek()——null

 另外在阻塞队列中还有两个阻塞方法：
```
    put 添加一个元素 如果队列已满 则阻塞
    take 删除并返回队列头部元素 如果队列为空 则阻塞
```
# 5. PriorityQueue
## 5.1 简介
优先级队列，是0个或多个元素的集合，集合中的每个元素都有一个权重值，每次出队都弹出优先级最大或最小的元素。一般来说，优先级队列使用堆排序来实现。这里简单介绍下用到的知识，堆排序依赖于完全二叉树。
若设二叉树的深度为h，除第h层外，其它各层 (1～h-1) 的结点数都达到最大个数，第h层所有的结点都连续集中在最左边，这就是完全二叉树。
在完全二叉树之上，又引入一个新的概念，叫做最小/大堆，堆是一种经过排序的完全二叉树，其中任一非终端节点的数据值均不大于（或不小于）其左子节点和右子节点的值。
## 5.2 源码分析
### 5.2.1 主要属性
```java
    // 默认容量
    private static final int DEFAULT_INITIAL_CAPACITY = 11;
    // 存储元素的地方
    transient Object[] queue; // non-private to simplify nested class access
    // 元素个数
    private int size = 0;
    // 比较器
    private final Comparator<? super E> comparator;
    // 修改次数
    transient int modCount = 0; // non-private to simplify nested class access
```
（1）默认容量是11；
（2）queue，元素存储在数组中，这跟我们之前说的堆一般使用数组来存储是一致的；
（3）comparator，比较器，在优先级队列中，也有两种方式比较元素，一种是元素的自然顺序，一种是通过比较器来比较；
（4）modCount，修改次数，有这个属性表示PriorityQueue也是fast-fail的，像ArrayList、HashMap中都有一个属性叫modCount，每次对集合的修改这个值都会加1，在遍历前记录这个值到expectedModCount中，遍历中检查两者是否一致，如果出现不一致就说明有修改，则抛出ConcurrentModificationException异常。
### 5.2.2 入列
入队有两个方法，add(E e)和offer(E e)，在PriorityQueue实现中两者是一致的，add(E e)也是调用的offer(E e)。
```java
    public boolean add(E e) {
        return offer(e);
    }


    public boolean offer(E e) {
        // 不支持null元素
        if (e == null)
            throw new NullPointerException();
        modCount++;
        // 取size
        int i = size;
        // 元素个数达到最大容量了，扩容
        if (i >= queue.length)
            grow(i + 1);
        // 元素个数加1
        size = i + 1;
        // 如果还没有元素
        // 直接插入到数组第一个位置
        // 这里跟我们之前讲堆不一样了
        // java里面是从0开始的
        // 我们说的堆是从1开始的
        if (i == 0)
            queue[0] = e;
        else
            // 否则，插入元素到数组size的位置，也就是最后一个元素的下一位
            // 注意这里的size不是数组大小，而是元素个数
            // 然后，再做自下而上的堆化
            siftUp(i, e);
        return true;
    }

    //自下而上堆化
    private void siftUp(int k, E x) {
        // 根据是否有比较器，使用不同的方法
        if (comparator != null)
            siftUpUsingComparator(k, x);
        else
            siftUpComparable(k, x);
    }

    @SuppressWarnings("unchecked")
    private void siftUpComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        while (k > 0) {
            // 找到父节点的位置
            // 因为元素是从0开始的，所以减1之后再除以2
            int parent = (k - 1) >>> 1;
            // 父节点的值
            Object e = queue[parent];
            // 比较插入的元素与父节点的值
            // 如果比父节点大，则跳出循环
            // 否则交换位置
            if (key.compareTo((E) e) >= 0)
                break;
            // 与父节点交换位置
            queue[k] = e;
            // 现在插入的元素位置移到了父节点的位置
            // 继续与父节点再比较
            k = parent;
        }
        // 最后找到应该插入的位置，放入元素，最终形成一个最小堆
        queue[k] = key;
    }
```
（1）入队不允许null元素；
（2）如果数组不够用了，先扩容；
（3）如果还没有元素，就插入下标0的位置；
（4）如果有元素了，就插入到最后一个元素往后的一个位置（实际并没有插入哈）；
（5）自下而上堆化，一直往上跟父节点比较；
（6）如果比父节点小，就与父节点交换位置，直到出现比父节点大为止；
（7）由此可见，PriorityQueue是一个小顶堆。
### 5.2.3 扩容
```
    private void grow(int minCapacity) {
        // 旧容量
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        // 旧容量小于64时，容量翻倍
        // 旧容量大于等于64，容量只增加旧容量的一半
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
        // overflow-conscious code
        // 检查是否溢出
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 创建出一个新容量大小的新数组并把旧数组元素拷贝过去
        queue = Arrays.copyOf(queue, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
### 5.2.4 出队
出队有两个方法，remove()和poll()，remove()也是调用的poll()，只是没有元素的时候抛出异常。
```java
    public E remove() {
        // 调用poll弹出队首元素
        E x = poll();
        if (x != null)
            // 有元素就返回弹出的元素
            return x;
        else
            // 没有元素就抛出异常
            throw new NoSuchElementException();
    }

    @SuppressWarnings("unchecked")
    public E poll() {
        // 如果size为0，说明没有元素
        if (size == 0)
            return null;
        // 弹出元素，元素个数减1
        int s = --size;
        modCount++;
        // 队列首元素
        E result = (E) queue[0];
        // 队列末元素
        E x = (E) queue[s];
        // 将队列末元素删除
        queue[s] = null;
        // 如果弹出元素后还有元素
        if (s != 0)
            // 将队列末元素移到队列首
            // 再做自上而下的堆化
            siftDown(0, x);
        // 返回弹出的元素
        return result;
    }

    //自上而下堆化
    private void siftDown(int k, E x) {
        // 根据是否有比较器，选择不同的方法
        if (comparator != null)
            siftDownUsingComparator(k, x);
        else
            siftDownComparable(k, x);
    }

    @SuppressWarnings("unchecked")
    private void siftDownComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>)x;
        // 只需要比较一半就行了，因为叶子节点占了一半的元素
        int half = size >>> 1;        // loop while a non-leaf
        while (k < half) {
            // 寻找子节点的位置，这里加1是因为元素从0号位置开始
            int child = (k << 1) + 1; // assume left child is least
            // 左子节点的值
            Object c = queue[child];
            // 右子节点的位置
            int right = child + 1;
            if (right < size &&
                ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
                // 左右节点取其小者
                c = queue[child = right];
            // 如果比子节点都小，则结束
            if (key.compareTo((E) c) <= 0)
                break;
            // 如果比最小的子节点大，则交换位置
            queue[k] = c;
            // 指针移到最小子节点的位置继续往下比较
            k = child;
        }
        // 找到正确的位置，放入元素
        queue[k] = key;
    }
```
（1）将队列首元素弹出；
（2）将队列末元素移到队列首；
（3）自上而下堆化，一直往下与最小的子节点比较；
（4）如果比最小的子节点大，就交换位置，再继续与最小的子节点比较；
（5）如果比最小的子节点小，就不用交换位置了，堆化结束；
（6）这就是堆中的删除堆顶元素；
### 5.2.5 取队列首元素
取队首元素有两个方法，element()和peek()，element()也是调用的peek()，只是没取到元素时抛出异常。
```java
    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    public E peek() {
        return (size == 0) ? null : (E) queue[0];
    }
```
### 5.2.6 优先队列总结
（1）PriorityQueue是一个小顶堆；
（2）PriorityQueue是非线程安全的；
（3）PriorityQueue不是有序的，只有堆顶存储着最小的元素；
（4）入队就是堆的插入元素的实现；
（5）出队就是堆的删除元素的实现；

# 6. ConcurrentLinkedQueue
## 6.1 简介
ConcurrentLinkedQueue只实现了Queue接口，并没有实现BlockingQueue接口，所以它不是阻塞队列，也不能用于线程池中，但是它是线程安全的，可用于多线程环境中。
## 6.2 源码分析
### 6.2.1 主要属性
```java
    // 链表头节点
    private transient volatile Node<E> head;
    // 链表尾节点
    private transient volatile Node<E> tail;
```
就这两个主要属性，一个头节点，一个尾节点。
### 6.2.2 主要内部类
```java
    private static class Node<E> {
        volatile E item;
        volatile Node<E> next;
    }
```
典型的单链表结构，非常纯粹。
### 6.2.3 主要构造方法
```java
    public ConcurrentLinkedQueue() {
        // 初始化头尾节点
        head = tail = new Node<E>(null);
    }

    public ConcurrentLinkedQueue(Collection<? extends E> c) {
        Node<E> h = null, t = null;
        // 遍历c，并把它元素全部添加到单链表中
        for (E e : c) {
            checkNotNull(e);
            Node<E> newNode = new Node<E>(e);
            if (h == null)
                h = t = newNode;
            else {
                t.lazySetNext(newNode);
                t = newNode;
            }
        }

        if (h == null)
            h = t = new Node<E>(null);
        head = h;
        tail = t;
    }

```
### 6.2.4 入队
因为它不是阻塞队列，所以只有两个入队的方法，add(e)和offer(e)。因为是无界队列，所以add(e)方法也不用抛出异常了。
```java
    public boolean add(E e) {
        return offer(e);
    }

    public boolean offer(E e) {
        // 不能添加空元素
        checkNotNull(e);
        // 新节点
        final Node<E> newNode = new Node<E>(e);
        // 入队到链表尾
        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            // 如果没有next，说明到链表尾部了，就入队
            if (q == null) {
                // CAS更新p的next为新节点
                // 如果成功了，就返回true
                // 如果不成功就重新取next重新尝试
                if (p.casNext(null, newNode)) {
                    // 如果p不等于t，说明有其它线程先一步更新tail
                    // 也就不会走到q==null这个分支了
                    // p取到的可能是t后面的值
                    // 把tail原子更新为新节点
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.
                    // 返回入队成功
                    return true;
                }
            }
            else if (p == q)
                // 如果p的next等于p，说明p已经被删除了（已经出队了）
                // 重新设置p的值
                p = (t != (t = tail)) ? t : head;
            else
                // t后面还有值，重新设置p的值
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
```
入队整个流程还是比较清晰的，这里有个前提是出队时会把出队的那个节点的next设置为节点本身。
（1）定位到链表尾部，尝试把新节点放到后面；
（2）如果尾部变化了，则重新获取尾部，再重试；
### 6.2.5 出队
```Java
    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    public E poll() {
        restartFromHead:
        for (;;) {
            // 尝试弹出链表的头节点
            for (Node<E> h = head, p = h, q;;) {
                E item = p.item;
                // 如果节点的值不为空，并且将其更新为null成功了
                if (item != null && p.casItem(item, null)) {
                    // 如果头节点变了，则不会走到这个分支
                    // 会先走下面的分支拿到新的头节点
                    // 这时候p就不等于h了，就更新头节点
                    // 在updateHead()中会把head更新为新节点
                    // 并让head的next指向其自己
                    if (p != h) // hop two nodes at a time
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    // 上面的casItem()成功，就可以返回出队的元素了
                    return item;
                }
                // 下面三个分支说明头节点变了
                // 且p的item肯定为null
                else if ((q = p.next) == null) {
                    // 如果p的next为空，说明队列中没有元素了
                    // 更新h为p，也就是空元素的节点
                    updateHead(h, p);
                    // 返回null
                    return null;
                }
                else if (p == q)
                    // 如果p等于p的next，说明p已经出队了，重试
                    continue restartFromHead;
                else
                    // 将p设置为p的next
                    p = q;
            }
        }
    }

    // 更新头节点的方法
    final void updateHead(Node<E> h, Node<E> p) {
        // 原子更新h为p成功后，延迟更新h的next为它自己
        // 这里用延迟更新是安全的，因为head节点已经变了
        // 只要入队出队的时候检查head有没有变化就行了，跟它的next关系不大
        if (h != p && casHead(h, p))
            h.lazySetNext(h);
    }
```
### 6.2.6 ConcurrentLinkedQueue总结：
（1）ConcurrentLinkedQueue不是阻塞队列；
（2）ConcurrentLinkedQueue不能用在线程池中；
（3）ConcurrentLinkedQueue使用（CAS+自旋）更新头尾节点控制出队入队操作；
ConcurrentLinkedQueue与LinkedBlockingQueue对比？
（1）两者都是线程安全的队列；
（2）两者都可以实现取元素时队列为空直接返回null，后者的poll()方法可以实现此功能；
（3）前者全程无锁，后者全部都是使用重入锁控制的；
（4）前者效率较高，后者效率较低；
（5）前者无法实现如果队列为空等待元素到来的操作；
（6）前者是非阻塞队列，后者是阻塞队列；
（7）前者无法用在线程池中，后者可以；

转自：
https://segmentfault.com/a/1190000019345930?utm_source=tag-newest
https://mp.weixin.qq.com/s/kpBAIRoMvqPzC-wfLELP3Q
https://www.jianshu.com/p/d9e8c8cd22af