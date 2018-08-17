[toc]
## 1. 引言
Singleton模式也叫做单例模式，是为了确保只生成一个实例的模式。
## 2. 示例程序
![uml](http://on-img.com/chart_image/5b7631ffe4b067df5a0d8351.png)
### 2.1 Singleton类
Singleton类定义了static字段（类成员变量）singleton，并将其初始化为Singleton类的实例。Singleton类的构造函数是private，这是为了禁止从Singleton类外部调用构造函数。
```
/**
 * 只生成一个实例
 * @author littlemotor
 * @since 18.8.17
 */
public class Singleton {
  
  /**
   * 在内部完成初始化
   */
  private static Singleton singleton = new Singleton();
  
  /**
   * 限制外部不能调用构造函数
   */
  private Singleton() {
    System.out.println("生成一个单例");
  }
  
  public static Singleton getInstance() {
    return singleton;
  }
}
```
### 2.2 Main类
调用了Singleton模式，并将实例保存在obj1和obj2中，判断是否为同一实例。
```
/**
 * 调用单例模式类，验证单例模式
 * @author littlemotor
 * @since 18.8.17
 */
public class Main {
  public static void main(String[] args) {
    System.out.println("strat");
    Singleton obj1 = Singleton.getInstance();
    Singleton obj2 = Singleton.getInstance();
    if(obj1 == obj2) {
      System.out.println("obj1和obj2是相同的实例");
    }else {
      System.out.println("obj1和obj2不是相同的实例");
    }
    System.out.println("End.");
  }
}
```
## 3. 小结
Singleton模式的作用在于可以确保任何情况下都只能生成一个实例，为了达到这个目的，必须为构造函数设置为private。
### 3.1 Singleton
Singleton角色中有一个返回唯一实例的static方法，该方法总会返回同一个实例。