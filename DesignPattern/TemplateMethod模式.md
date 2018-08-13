[toc]
## 1. 引言
TemplateMethod模式是带有模版功能的模式，组成模板的抽象方法被定义在父类中，实现抽象方法的是子类。像这样在父类中定义流程的框架，在子类中实现具体处理的模式就称为TemplateMethod模式。
## 2. 实例程序
实例程序将字符和字符串循环显示5次的简单程序。在AbstractDisplay类中定义了display抽象方法，在该方法中定义了open、print、close方法，调用抽象方法的display方法就是模板方法。
![uml](http://on-img.com/chart_image/5b70f7d1e4b0be50eadb780d.png)
<center>示例程序类图</center>
### 2.1 AbstractDisplay类
处理过程为调用open方法，调用5次print方法，调用close方法
```
/**
 * 模版抽象类，定义open、print、close抽象方法，并定义dispaly
 * 模版方法
 * @author littlemotor
 * @since 18.8.13
 */
public abstract class AbstractDisplay {
  
  public abstract void open();
  public abstract void print();
  public abstract void close();
  
  public final void display() {
    open();
    for(int i = 0; i < 5; i++) {
      print();
    }
    close();
  }
}
```
### 2.2 CharDisplay类

方法名 | 处理
---------|----------
open | 显示字符串"<<"
 print | 显示构造函数接收的一个字符
close | 显示字符串">>"

```
/**
 * 以连续显示单个字符5次的方式实现模板方法
 * @author littlemotor
 * @since 18.8.13
 */
public class CharDisplay extends AbstractDisplay{

  private char ch;
  
  public CharDisplay(char ch) {
    this.ch = ch;
  }
  
  @Override
  public void open() {
    System.out.print("<<");
  }

  @Override
  public void print() {
    System.out.print(ch);
  }

  @Override
  public void close() {
    System.out.println(">>");
  }
}
```
### 2.3 StringDisplay类

方法名 | 处理
---------|----------
 open | 显示字符串"+--------+"
 pirnt | 显示构造函数接受的字符串，并在前后加上"&#124;"显示出来
close | 显示字符串"+--------+"
```
/**
 * 以连续显示5次字符串的方式实现模板方法
 * @author littlemotor
 * @since 18.8.13
 */
public class StringDisplay extends AbstractDisplay{

  private String string;
  private int width;
  
  public StringDisplay(String string) {
    this.string = string;
  }

  @Override
  public void open() {
    printLine();
  }

  @Override
  public void print() {
    for(int i = 0; i < 5; i++) {
      System.out.println("|" + string + "|");
    }
  }

  @Override
  public void close() {
    printLine();
  }
  
  public void printLine() {
    System.out.print("+");
    for(int i = 0; i < string.length(); i++) {
      System.out.print("-");
    }
    System.out.println("+");
  }
}
```
### 2.4 Main类
```
public class Main {
  public static void main(String[] args) {
    AbstractDisplay d1 = new CharDisplay('H');
    AbstractDisplay d2 = new StringDisplay("Hello,world");
    AbstractDisplay d3 = new StringDisplay("你好，世界");
    
    d1.display();
    d2.display();
    d3.display();
  }
}
```
## 3. 小结
抽象类的意义是我们可以决定抽象方法的名字，然后通过调用使用了抽象方法的模版方法去编写处理。虽然具体的处理内容是由子类决定的，不过在抽象类阶段确定处理流程非常重要。
总而言之抽象类可以使逻辑处理通用化，在父类模版方法中编写算法，无需在每个子类中再编写算法。
### 3.1 AbstractClass抽象类
该角色不仅负责实现模板方法，还负责声明在模板方法中使用到的抽象方法。
### 3.2 ConcreteClass具体类
该角色负责实现AbstractClass角色中定义的抽象方法，他实现的方法会在AbstractClass角色的模板方法中被调用