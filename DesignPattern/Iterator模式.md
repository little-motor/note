[toc]
## 1. 简介
Iterator模式用于在数据集合中按照顺序遍历集合，也称为迭代器。
## 2. 示例程序
示例程序作用是将书(Book)放置到书架(BookShelf)中，并将书名按顺序显示出来,uml图如下
![uml](http://on-img.com/chart_image/5b6cfdb4e4b0f8477dabf6ba.png)

### 2.1 Collection接口
实现这个接口的类可以成为保存多个元素的集合
```
public interface Collection{
  public abstract Iterator iterator();
}
```
### 2.2 Iterator接口
最简单的接口定义如下
```
public interface Iterator{
  public abstract boolean hasNext();
  public abstract Object next();
}
```
### 2.3 Book类
```
public class Book{
  private String name;

  public Book(String name){
    this.name = name;
  }

  public String getName(){
    return name;
  }
}
```
### 2.4 BookShelf类
表示书架的类，由于需要对集合操作实现了Collection接口，同时实现了Collection接口的iterator方法。
```
public class BookShelf implements Collection{

  private Book[] books;
  private int last = 0;

  public BookShelf(int maxsize){
    this.books = new Book[maxsize];
  }

  public Book getBookAt(int index){
    return books[index];
  }

  public void appendBook(Book book){
    this.books[last] = book;
    last++;
  }

  public int getLength(){
    return last;
  }

  /**
   * 返回遍历书架所有书的迭代器
   */
  public Iterator iterator(){
    return new BookShelfIterator(this);
  }
}
```
### 2.5 BookShelfIterator类
这个类用于遍历书架上面的书，他实现了Iterator接口
```
public class BookShelfIterator implements Iterator{

  private BookShelf bookShelf;
  private int index;

  public BookShelfIterator(BookShelf bookShelf){
    this.bookShelf = bookShelf;
    this.index = 0;
  }

  public boolean hasNext(){
    if(index < bookShelf.getLength()){
      return true;
    } else{
      return false;
    }
  }

  /**
  * next方法返回当前指向的值后index指向下一个元素
  */
  public Object next(){
    Book book = bookShelf.getBookAt(index);
    index++;
    return book;
  }
}
```
### 2.5 Main类
```
public class Main {
  public static void main(String[] args) {
    Book book;
    BookShelf bookShelf  = new BookShelf(4);
    bookShelf.appendBook(new Book("A"));
    bookShelf.appendBook(new Book("B"));
    bookShelf.appendBook(new Book("C"));
    bookShelf.appendBook(new Book("D"));
    
    Iterator iterator = bookShelf.iterator();
    while(iterator.hasNext()) {
      book = (Book)iterator.next();
      System.out.println(book.getName());
    }
  }
}
```
## 3. Iterator模式
### 3.1 Iterator（迭代器）
定义按顺序逐个遍历元素的接口（API）。
### 3.2 ConcreteIterator（具体的迭代器）
负责实现Iterator角色所定义的接口（API）。
### 3.3 Collection（集合）
该角色负责定义创建Iterator角色的接口（API），实际上在Java中用的专门的Iterable接口实现，此处是在忽略细节突出主旨的情况下使用。
### 3.4 ConcreteCollection（具体集合）
该角色负责实现Collection角色所定义的接口（API），他会创建具体的Iterator角色，即ConcreteIterator角色。
## 4. 小结
引入iterator后可以将遍历与实现分离开来。设计模式的作用就是帮助我们编写可复用的类。所谓“可复用”，就是将类实现为“组件”，当一个组件发生改变时，不需要对其他组件进行修改或者只需要很小的修改即可应对。
### 4.1 注意“next”方法
next方法返回当前元素并指向下一个元素
### 4.2 注意“hasNext”方法
他的作用是用来确定接下来是否可以调用next方法。
### 4.3 迭代器的种类

- 从最后向前遍历
- 可以向前也可以向后遍历
- 指定下标进行“跳跃式”遍历
### 4.4 相关的设计模式

- Visitor模式
- Composite模式
- Factory Mothod模式