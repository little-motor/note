# Java基础 static关键字
## 1. 修饰成员变量和方法，让它们变为类的所属，而不是对象的所属
1. 当修饰成员变量时，其在内存中位于静态存储区。
2. static修饰成员方法最大的作用，就是可以使用"类名.方法名"的方式操作方法，避免了先要new出对象的繁琐和资源消耗，不过它也有使用的局限，一个static修饰的类中，不能使用非static修饰的成员变量和方法
## 2. 静态块
首先我们需要知道一个对象的初始化过程
1. 当new一个对象时，static修饰的成员变量首先被初始化，随后是普通成员，最后调用类自己的构造方法完成初始化。
2. 更进一步来讲，被static修饰的方法在第一次被调用时，即使没有new对象，其内部所有static修饰的方法都会完成初始化，并且只需要初始化一次即可。
```
class Book{
    public Book(String msg) {
        System.out.println(msg);
    }
}

public class Person {

    Book book1 = new Book("book1成员变量初始化");
    static Book book2 = new Book("static成员book2成员变量初始化");
    
    public Person(String msg) {
        System.out.println(msg);
    }
    
    Book book3 = new Book("book3成员变量初始化");
    static Book book4 = new Book("static成员book4成员变量初始化");
    
    public static void funStatic() {
        System.out.println("static修饰的funStatic方法");
    }
    
    public static void main(String[] args) {
        Person.funStatic();
        System.out.println("****************");
        Person p1 = new Person("p1初始化");
    }


    /**Output
     * static成员book2成员变量初始化
     * static成员book4成员变量初始化
     * static修饰的funStatic方法
     * ***************
     * book1成员变量初始化
     * book3成员变量初始化
     * p1初始化
     */
     

}
```
静态块是指，可以将static修饰的成员变量统一放在一个以static开始，用花括号包裹起来的块状语句中，将上面的例子稍微修改一下，结果并无二致。
```
class Book{
    public Book(String msg) {
        System.out.println(msg);
    }
}

public class Person {

    Book book1 = new Book("book1成员变量初始化");
    static Book book2;
    
    static {
        book2 = new Book("static成员book2成员变量初始化");
        book4 = new Book("static成员book4成员变量初始化");
    }
    
    public Person(String msg) {
        System.out.println(msg);
    }
    
    Book book3 = new Book("book3成员变量初始化");
    static Book book4;
    
    public static void funStatic() {
        System.out.println("static修饰的funStatic方法");
    }
    
    public static void main(String[] args) {
        Person.funStatic();
        System.out.println("****************");
        Person p1 = new Person("p1初始化");
    }

    /**Output
     * static成员book2成员变量初始化
     * static成员book4成员变量初始化
     * static修饰的funStatic方法
     * ***************
     * book1成员变量初始化
     * book3成员变量初始化
     * p1初始化
     */

}
```
## 3. 静态导包
以下面代码为例
```
/* PrintHelper.java文件 */
package test;

public class PrintHelper {

    public static void print(Object o){
        System.out.println(o);
    }
}
```
```
/* App.java文件 */

import static test.PrintHelper.*;

public class App 
{
    public static void main( String[] args )
    {
        print("Hello World!");
    }

    /**Output
     * Hello World!
     */
}
```
而在App.java文件中，我们使用了static关键字导入PrintHelper类，它的作用就是将PrintHelper类中的所有类方法直接导入。不同于非static导入，采用static导入包后，在不与当前类的方法名冲突的情况下，直接可以采用"方法名"去调用类方法，就好像是该类自己的方法一样使用即可。

## 4. 总结
static是java中非常重要的一个关键字，而且它的用法也很丰富，主要有四种用法：
1. 用来修饰成员变量，将其变为类的成员，从而实现所有对象对于该成员的共享；
2. 用来修饰成员方法，将其变为类方法，可以直接使用“类名.方法名”的方式调用，常用于工具类；
3. 静态块用法，将多个类成员放在一起初始化，使得程序更加规整，其中理解对象的初始化过程非常关键；
4. 静态导包用法，将类的方法直接导入到当前类中，从而直接使用“方法名”即可调用类方法，更加方便。
 
参考文章：[[java]static关键字的四种用法](https://www.cnblogs.com/dotgua/p/6354151.html?utm_source=itdadao&utm_medium=referral)

