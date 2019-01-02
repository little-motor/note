[toc]
## 1. 引言
在FactoryMethod模式中，父类决定实例的生产方式，但并不决定所要生成的具体的类，具体的处理全部交给子类负责。这样就可以将生成实例的框架(framework)和负责生成实例的类解藕。
主要分为普通工厂模式（SimpleFactory）和抽象工厂（AbstractFactory）模式
## 2. 普通工厂模式
Product和Factory类属于framework包，这两个类组成了生成实例的框架。
IDCard类和IDCardFactory类负责实际的加工处理，他们属于idcard包。

- 生成实例的框架(framework)
- 加工处理(idcard)
![uml](http://on-img.com/chart_image/5b74db54e4b067df5a0b8bee.png)

### 2.1 Product类
在这个框架中定义了产品是“任意的可以use的”东西
```
package framework;

public abstract class Product {
  public abstract void use();
}
```
### 2.2 Factory类
定义工厂是用来“调用create方法生成Product实例”，具体的实现内容根据FactoryMethod模式适用的场景不同而不同，但是，只要是FactoryMethod模式，在生成实例时就一定会使用TemplateMethod模式。
```
package framework;

/**
 * 调用create方法生成Product实例，只要是FactoryMethod模式，
 * 在生成实例时就一定会用到TemplateMethod模式。
 * 
 * @author littlemotor
 * @since 18.8.15
 */
public abstract class Factory {
  /**
   * 在抽象类中实现了逻辑方法，利用create方法生产和注册产品，
   * 根据不同的场景会有不同的实现
   * @param owner
   * @return
   */
  public final Product create(String owner) {
    Product p = createProduct(owner);
    registerProduct(p);
    return p;
  }
  
  protected abstract Product createProduct(String owner);
  
  protected abstract void registerProduct(Product p);
}
```
### 2.3 IDCard类
在负责加工处理这一边(idcard包)，IDCard类是产品Product类的子类。
```
package idcard;

import framework.Product;

/**
 * IDCard是Product类的子类
 * @author littlemotor
 * @since 18.8.15
 */
public class IDCard extends Product{

  private String owner;
  
  public IDCard(String owner) {
    System.out.println("制作" + owner + "的ID卡");
    this.owner = owner;
  }
  
  @Override
  public void use() {
    System.out.println("使用" + owner + "的ID卡");
  }
  
  public String getOwner() {
    return owner;
  }
}
```
### 2.4 IDCardFactory类
```
package idcard;

import java.util.ArrayList;
import java.util.List;

import factoryMethod.framework.Factory;
import factoryMethod.framework.Product;

public class IDCardFactory extends Factory{

  private List owners = new ArrayList();
  
  @Override
  protected Product createProduct(String owner) {
    return new IDCard(owner);
  }

  @Override
  protected void registerProduct(Product product) {
    owners.add(((IDCard)product).getOwner());
  }
  
  public List getOwners() {
    return owners;
  }
}
```
### 2.5 Main类
在Main类中调用framework包和idcard包来制作和使用IDCard。
```
package idcard;

import factoryMethod.framework.Factory;
import factoryMethod.framework.Product;

public class Main {
  public static void main(String[] args) {
    Factory factory = new IDCardFactory();
    Product card1 = factory.create("小明");
    Product card2 = factory.create("小红");
    Product card3 = factory.create("小刚");
    card1.use();
    card2.use();
    card3.use();
  }
}
```
## 3. 抽象工厂模式 
抽象工厂模式可以向客户端提供一个接口，使得客户端在不指定产品具体情况下，创建多个产品族中的产品对象。对于普通工厂他解决了一类对象创建问题，但是有时候对象很复杂，有几十种分为几个类别，所以通过抽象工厂把工厂分为好几个，便于工厂产品规则的维护。
![uml](http://on-img.com/chart_image/5b801176e4b0534c9bbb1640.png)
此处我们以车场为例，旗下有各种车型的车场，每种车厂会生产多种型号的车，当客户提交订单后，对他来说工厂是一个抽象概念，并不真正存在，而是通过各个分厂实现的，这个虚拟工厂称为抽象工厂，他的各个工厂称为具体工厂。
### 3.1 ProductFactory接口
```
public interface ProductFactory{
  public Product createProduct(String productNo);
}
```
### 3.2 抽象工厂实现
这里假设有三个具体工厂，其伪代码如下
```
public class ProductFactory1 implements ProductFactory{
  @Override
  public Product createProduct(String productNo){
    Product product = ...//工厂1生成产品对象1的规则
    return product;
  }
} 

public class ProductFactory2 implements ProductFactory{
  @Override
  public Product createProduct(String productNo){
    Product product = ...//工厂2生成产品对象2的规则
    return product;
  }
} 

public class ProductFactory3 implements ProductFactory{
  @Override
  public Product createProduct(String productNo){
    Product product = ...//工厂3生成产品对象3的规则
    return product;
  }
} 
```
将这三个具体工厂放在一个类中，其代码如下
```
public class ProductFactories implements ProductFactory{
  @Override
  public ProductFactory createProduct(String productNo){
    char ch = productNo.charAt(0);
    Factory factory = null
    if(ch == '1'){
      factory = new ProductFactory1();
    } else if(ch == '2'){
      factory = new ProductFactory2();
    } else if(ch == '3'){
      factory = new ProductFactory3();
    }
    if (factory == null){
      return factory.createProduct(productNo)
    }
    return null;
  }
}
```
这样通过ProductFactories根据不同的情况调用ProductFactory1、ProductFactory2、ProductFactory3具体工厂。
## 4. 总结
父类（框架）这一方的Creator角色和Product角色的关系与子类（具体加工）这一方的ConcreteCreator角色和ConcreteProduct角色的关系是平行的。在这个示例程序中，我们没有必要修改framework包中的任何内容，就可以创建出其他的“产品”和“工厂”。
同时也体会到在分析设计模式时，不应当将其中一个类单独拿出来分析，必须着眼于类和接口之间的相互关系。
### 4.1 Product（产品）
Product是抽象类属于框架这一方，定义了FactoryMethod模式中生成实例所持有的接口（API），具体的处理则由子类ConcreteProduct角色决定。
### 4.2 Creator（创建者）
Creator角色属于框架这一方，负责生成Product角色的抽象类，定义生成实例的逻辑方法。
### 4.3 ConcreteProduct（具体产品）
### 4.4 ConcreteCreator（具体创建者）
