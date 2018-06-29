# Iterator迭代器总结
## 1. 简介：
Collection合集框架接口继承自Iterable接口，Iterable接口中定义了iterator方法，该方法返回一个iterator接口，Iterator接口为便利各种类型的合集中的元素提供了统一的方法。源码如下
```
public interface Iterable<T> {
    
    Iterator<T> iterator();

    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
    
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```
```
public interface Iterator<E> {
  
  boolean hasNext();

  E next();

  default void remove() {
        throw new UnsupportedOperationException("remove");
    }

  default void forEachRemaining(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    while (hasNext())
      action.accept(next());
    }
```
## 2. 方法
从iterator接口中我们看到iterator接口提供的方法有hasNext(),next(),remove()方法。
- 当创建完成指向某个集合或者容器的Iterator对象后，这时的指针其实指向的是第一个元素的上方，即指向一个空元素
- 当调用hasNext方法的时候，只是判断下一个元素的有无，并不移动指针
- 当调用next方法的时候，向下移动指针，并且返回指针指向的元素，如果指针指向的内存中没有元素，会抛出异常。
- remove方法删除的元素是指针指向的元素。如果当前指针指向的内存中没有元素，那么会抛出异常。所以remove方法一般和next方法配合使用，其删除的是最近next方法返回的元素。
