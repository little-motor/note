[toc]
## 1. 引言
Java反射技术应用广泛，他能够配置：类的全限定名、方法和参数，完成对象的初始化，甚至是反射某些方法，这样就可以大大增强Java的可配置性。Java的反射内容繁多，包括对象构建、反射方法、注解、参数、接口等，这里主要学习对象构建和方法的反射调用。
## 2. 通过反射构建对象
### 2.1 在无参构造函数的情况下
示例程序如下 
```
package reflect;

/**
 * 反射技术的学习，这里主要实践对象构建和方法的反射
 * 调用，此处为无参构造函数的情况
 * @author littlemotor
 * @since 18.8.25
 */
public class ReflectServiceImpl {
  
  public ReflectServiceImpl() {
    
  }
  
  public void sayHello(String name) {
    System.out.println("Hello " + name);
  } 
}
```
通过反射方法去构建他
```
package reflect;

import java.lang.reflect.InvocationTargetException;

public class ReflectMain {

  /**
   * 只有无参构造函数的情况下返回实例
   * @return ReflectServiceImpl
   */
  public static ReflectServiceImpl getInstance() {
    ReflectServiceImpl object = null;
    try {
      object = (ReflectServiceImpl)Class.forName("reflect.ReflectServiceImpl").newInstance();
        } catch(ExceptionInInitializerError | ClassNotFoundException 
            | IllegalAccessException | InstantiationException ex) {
          ex.printStackTrace();
        } 
    return object;   
  }  
  
  public static void main(String[] args) {
    ReflectServiceImpl test = getInstance();
    test.sayHello("littlemotor");
  }
}

//运行结果
//Hello littlemotor
```
其中实现反射的最主要代码就一行
```
object = (ReflectServiceImpl)Class.forName("reflect.ReflectServiceImpl").newInstance();
```
### 2.2 在有参构造函数的情况下
实例程序如下
```
package reflect;

/**
 * 反射练习，返回有参构造函数类的实例
 * @author littlemotor
 *
 */
public class ReflectServiceImpl2 {
  private String name;
  
  public ReflectServiceImpl2(String name) {
    this.name = name;
  }
  
  public void sayHello() {
    System.out.println("Hello " + name);
  }
}
```
通过反射方法去构建实例
```
package reflect;

import java.lang.reflect.InvocationTargetException;

public class ReflectMain {
  /**
   * 返回在有参构造函数的情况下通过反射返回实例
   * @return
   */
  public static ReflectServiceImpl2 getInstance2() {
    ReflectServiceImpl2 object = null;
    try {
      object = (ReflectServiceImpl2)Class.forName("reflect.ReflectServiceImpl2")
                .getConstructor(String.class).newInstance("littlemotor 2");
    } catch(ExceptionInInitializerError | InstantiationException | IllegalAccessException | ClassNotFoundException | IllegalArgumentException | InvocationTargetException | NoSuchMethodException | SecurityException ex) {
      ex.printStackTrace();
    }
    return object;
  }
  
  public static void main(String[] args) {
    ReflectServiceImpl2 test2 = getInstance2();
    test2.sayHello();
  }
}

//运行结果
//Hello littlemotor 2
```
其中反射获得类实例的代码
```
object = (ReflectServiceImpl2)Class.forName("reflect.ReflectServiceImpl2")
                .getConstructor(String.class).newInstance("littlemotor 2");
```
首先通过forName加载到类的加载器，然后通过getConstract方法，他的参数可以是多个，这里的定义为<code>String.class</code>，上一行代码实际就等于<code>object = new ReflectServiceImpl("littlemotor");</code>反射的优点是灵活同时降低程序的耦合度，缺点是比较慢。
## 3. 反射调用方法
在使用反射调用方法前要获取方法对象，示例程序如下
```
/**
   * 通过反射方法得到方法对象并调用执行
   * @param args
   */
  public static void invokeSayHelloMethod() {
    ReflectServiceImpl2 object = null;
    try {
      object = (ReflectServiceImpl2)Class.forName("reflect.ReflectServiceImpl2")
          .getConstructor(String.class).newInstance("littlemotor 3");
      //得到方法对象
      Method method = object.getClass().getMethod("sayHello");
      method.invoke(object);
    }catch(IllegalAccessException | InstantiationException | IllegalArgumentException | InvocationTargetException | NoSuchMethodException | SecurityException | ClassNotFoundException ex) {
      ex.printStackTrace();
    }
        
  }
  
  public static void main(String[] args) {
    invokeSayHelloMethod();
  }
}

//执行结果
//Hello littlemotor 3
```
这里面重要的就是三行程序，获得类的反射实例，再通过类的实例反射得到方法并调用
```
object = (ReflectServiceImpl2)Class.forName("reflect.ReflectServiceImpl2")
          .getConstructor(String.class).newInstance("littlemotor 3");
      //得到方法对象
Method method = object.getClass().getMethod("sayHello");
method.invoke(object);
```
## 4. 小结
产生实例的方法除了new和工厂方法外，还有灵活的反射方法，这个主要在框架中使用，缺点是比较慢一点，理解起来也稍微会费点脑经。