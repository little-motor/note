 [toc]
## 1. 引言
如果想让交流220V为12V笔记本充电，需要使用电源适配器，在程序世界中，也经常会存在现有程序无法使用，需要做适当变换之后才能使用的情况。这种用于填补“现有的程序”和“所需的程序”之间差异的模式就是Adapter模式。Adapter模式也被称为Wrapper模式（包装器）
Adapter模式有以下两种

- **类适配器模式（使用继承的适配器）**
- **对象适配器模式（使用委托的适配器）**
## 2. 类适配器模式
### 2.1 示例程序
示例程序是输出（hello）和*hello*的简单程序。在Banner（有广告横幅的意思）类中，有将字符串用括号括起来的showWithParen方法和将字符串用*号括起来的showWithAster方法。假设这个Banner类是类似引言中“220V交流电”。
假设Print接口中声明了两种方法，弱化字符串显示（加括号）的printWeak方法，和强调字符串显示（加*号）的printStrong方法。假设这个Pring接口类似于笔记本“12V直流电需求”。
现在要做的事情是使用Banner类编写一个实现了Print接口的类，也就是做一个“220V交流电”转“12V直流电”的适配器，扮演适配器的角色是PrintBanner类，在该类中使用showWithParen方法实现printWeak，使用showWithAster方法实现printStrong。

 <center>|电源的比喻 | 示例程序
---------|----------|---------
 实际情况 | 交流220V | Banner类(showWithParen、showWithAster)
 变换装置 | 适配器 | PrintBanner类
需求 | 直流12V | Print接口(printWeak、printStrong)
![UML](http://on-img.com/chart_image/5b6f9c23e4b08d3622a97033.png)
<center>类适配器模式</center>
### 2.2 Banner类
```
/**
 * 假设这个是被适配的类
 * @author littlemotor
 * @since 18.8.12
 */
public class Banner {
  
  private String string;
  
  public Banner(String string) {
    this.string = string;
  }
  
  public void showWithParen() {
    System.out.println("(" + string + ")");
  }
  
  public void showWithAster() {
    System.out.println("*" + string + ")");
  }
  
}
```
### 2.3 Print接口
```
/**
 * 代表需求的接口
 * @author littlemotor
 * @since 18.8.12
 */
public interface Print {
  
  public void printWeak();
  
  public void printStrong();
}
```
### 2.4 PrintBanner类
PrintBanner类扮演适配器角色，他继承了Banner类，同时实现了Print接口的需求。
```
/**
 * 适配器类，用Banner类的方法实现Print接口方法
 * @author littlemotor
 * @since 18.8.12
 */
public class PrintBanner extends Banner implements Print {

  public PrintBanner(String string) {
    super(string);
  }

  //用showWithParen方法实现printWeak方法
  @Override
  public void printWeak() {
    showWithParen();
  } 

  //用showWithAster方法实现showWithAster方法
  @Override
  public void printStrong() {
    showWithAster();
  }

}
```
###  2.5 Main类
扮演请求的角色，调用Print接口方法。
```
public class Main {
  public static void main(String[] args) {
    
    Print p = new PrintBanner("hello");
     
    p.printStrong();
    p.printWeak();
  }
}
```
在Main类中，我们将PrintBanner类的实例保存在了Pirnt类型的变量中，对于Main类的代码而言Banner类的方法是被完全隐藏起来的，就像笔记本在12V电压下正常工作但他并不知道这是220V电压转换而成的。这样的好处是可以在不用对Main类进行修改的情况下修改PrintBanner类的具体实现。
## 3. 对象适配器模式
### 3.1 关于委托
在Java里委托就是指将某个方法中的实际处理交给其他实例的方法。PrintBanner类继承了Print类并在构造函数中生成了Banner类的实例，然后在showWeak和showStrong方法中调用Banner实例对应的方法。
![uml](http://on-img.com/chart_image/5b6fb556e4b0edb75104aade.png)

<center>对象适配器模式</center>
### 3.2 Print类
```
/**
 * 代表需求的抽象类
 * @author littlemotor
 * @since 18.8.12
 */
public abstract class Print {
  
  public abstract void printWeak();
  
  public abstract void printStrong();
}
```
### 3.3 PrintBanner类
```
/**
 * 适配器类，用Banner类实例方法实现Print抽象类方法
 * @author littlemotor
 * @since 18.8.12
 */
public class PrintBanner extends Print {

  Banner banner;
  
  public PrintBanner(String string) {
    banner = new Banner(string);
  }
  
  //用Banner实例方法实现printWeak方法
  @Override
  public void printWeak() {
    banner.showWithParen();
  }

  //用Banner实例方法实现showWithAster方法
  @Override
  public void printStrong() {
    banner.showWithAster();
  }

}
```
## 4. 小结
Adapters模式用于填补具有不同接口（API）的两个类之间的缝隙，他包含四个角色
### 4.1 Target(对象)
该角色负责定义所需的方法，本例中为Print
### 4.2 Client(请求者)
该角色负责使用Target角色所定义的方法，本例中为Main
### 4.3 Adaptee(被适配)
是一个持有既定方法的角色，本例中为Banner
### 4.4 Adapter(适配)
Adapter模式的主人公，在类适配器模式中，Adapter角色通过继承使用Adaptee角色，而在对象适配器模式中，Adapter角色通过委托来实现Adaptee角色。
## 5. 什么时候用Adapter模式
### 5.1 利用现有代码
很多时候我们并非从零开始编程，经常会用到现有的类。特别是现有的类已经被充分测试过Bug很少，而且已经用于其他软件之中时，我们更愿意将这些类作为组件重复利用。通过该模式可以很方便的创建我们需要的方法群，当出现Bug时代码排查也会相对简单。
### 5.2 版本升级与兼容性
可以使用Adapter模式使新旧版本兼容，可以让新版本扮演Adaptee角色，旧版本扮演Target角色，接着编写一个扮演Adapter角色的类，让他使用新版本中的类来实现旧版本中的方法。
### 5.3 注意
需要注意的是当Adaptee角色和Target角色功能完全不同时，Adapter模式是无法使用的，就像是220V的电源无法转换为自来水一样。
## 6. 相关的设计模式
### 6.1 Bridge模式
### 6.2 Decorator模式

