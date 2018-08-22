[toc]
## 1. 引言
Prototype原型模式是指根据实例原型、实例模型来生成新的实例。
## 2. 示例程序
### 2.1 类和接口一览表

包 | 类名 | 说明
---------|----------|---------
 framewrok | Product | 声明抽象方法use和createClone的接口
framework | Manager | 调用createClone方法复制实例的类
 <null>| MessageBox | 实现了use和createClone方法，将字符串放入方框中显示
 <null>|MessageUnderline|实现了use和createClone方法，将字符串下方添加一串字符显示
 <null>|Main|测试程序行为的类
### 2.2 UML图
![uml](http://on-img.com/chart_image/5b77bcdfe4b08d3622b390c8.png)
### 2.3 Product接口
use定义“使用”的方法。
createClone方法定义用于复制实例的方法。
```
package framework;

/**
 * 继承Cloneable接口，连接Manager和实现Product实现类之间的桥梁
 * @author littlemotor
 * @since  8.18
 */
public interface Product extends Cloneable{
  
  public abstract void use(String s);
  
  public abstract Product createClone();

}
```
### 2.4 Manager类
Manager并没有用到别的具体需要复制的类名，保证了与其他类之间的松耦合。
```
package framework;

import java.util.HashMap;

public class Manager {
  private HashMap showcase = new HashMap();
  
  public void register(String name, Product prototype) {
    showcase.put(name, prototype);
  }
  
  public Product create(String protoname) {
    Product p = (Product) showcase.get(protoname);
    return p.createClone();
  }
}
```
### 2.5 MessageBox类
createClone方法用于复制自己，在进行复制时，原来实例中字段的值也会被复制到新的实例中，之所以能够调用clone方法是因为父类中实现了Cloneable接口。
需要注意的是只有类自己（或他的子类）能够点用clone方法，其他类要求复制时是通过调用封装的createClone方法实现的。
```
import framework.Product;

/**
 * 显示字符的方式是用一个指定符包围
 * @author littlemotor
 * @since 18.8.18
 */

public class MessageBox implements Product {
  private char decochar;
  
  public MessageBox(char decochar) {
    this.decochar = decochar;
  }
  
  /**
   * 包围显示
   */
  @Override
  public void use(String s) {
    System.out.println("字符串" + s + "被字符" + decochar +"包围显示");
  }

  @Override
  public Product createClone() {
    Product p = null;
    try {
      p = (Product)clone();
    } catch(CloneNotSupportedException e) {
      e.printStackTrace();
    }
    return p;
  }

}
```
### 2.6 MessageUnderline类
这个类的显示方式是在字符串下面加一行指定的字符串
```
import framework.Product;

public class MessageUnderline implements Product {

  private char underChar;
  
  public MessageUnderline(char underChar) {
    this.underChar = underChar;
  }
  
  @Override
  public void use(String s) {
    System.out.println("字符串" + s + "下方显示下划线" + underChar);
  }

  @Override
  public Product createClone() {
    Product p = null;
    try {
      p = (Product)clone();
    } catch(CloneNotSupportedException e) {
      e.printStackTrace();
    }
    return p;
  }

}
```
### 2.7 Main类
```
import framework.Manager;
import framework.Product;

/**
 * 通过Manager类调用注册在Product接口下方的不同类的实例 生成新的实例体会原型模式
 * 
 * @author littlemotor
 * @since 18.8.18
 */
public class Main {
  public static void main(String[] args) {
    //准备
    Manager manager = new Manager();
    MessageUnderline messageUnderline = new MessageUnderline('~');
    MessageBox messageBox = new MessageBox('*');
    manager.register("message1", messageUnderline);
    manager.register("message2", messageBox);
    //生成
    Product p1 = manager.create("message1");
    p1.use("hello,world");
    Product p2 = manager.create("message2");
    p2.use("hello.world");
  }
}

//结果
字符串hello,world下方显示下划线~
字符串hello.world被字符*包围显示
```
## 3. 小结
在类中尽量不要出现别的类名是为了实现组件复用，以达到只有.class文件时也可以复用的目的。
在Prototype原型模式中有三个角色，分别是
### 3.1 Prototype（原型）
Product角色负责定义用于复制现有实例来生成新实例的方法。
### 3.2 ConcretePrototype（具体的原型）
ConcretePrototype负责实现复制现有实例并生成新实例的方法。
### 3.3 Client（使用者）
由Manager类扮演此角色，Client角色使用复制实例的方法生成新的实例。
## 4. 拓展——Java中的clone方法和Clonable接口
### 4.1 调用clone方法的基本要求
被复制对象的类或父类必须实现java.lang.Clonable接口或其子接口，则可以调用clone方法，clone方法返回值是新复制出的实例（clone方法内部的处理是分配与要复制的实例相同大小的内存空间，接着将要复制的实例中的字段的值复制到所分配的内存空间中去）
### 4.2 Cloneable接口
Cloneable接口被称为标记接口（marker interface），其内部并没有声明任何方法，只是用来标记“可以使用clone方法进行复制”的。
### 4.3 Clone方法进行的是浅复制
clone方法进行的复制只是将被复制实例的字段值直接复制到新的实例中，并没有考虑字段中所保存实例的内容，例如，当字段保存的是数组时，使用clone方法进行复制，只会复制该数组的引用，并不会一一复制数组中的元素。更详细的[深拷贝与浅拷贝的区别](https://blog.csdn.net/qq_39385118/article/details/81808399)请参阅另一篇文章。
