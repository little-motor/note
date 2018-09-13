[toc]
## 1. 引言
通常，在构造复杂结构的物体时，很难一气呵成，我们需要首先构造组成这个物体的各个部分，然后分阶段将他们组装起来，Builer模式也是这个原理。
## 2. 示例程序
我们使用Builder模式编写“文档”，他具有如下结构：

- 含有一个标题
- 含有几个字符串
- 含有条目项目

抽象类Builder定义了决定文档结构的方法，Director类使用这些方法编写具体的文档，Builder类的子类决定了编写文档的具体方法。

- TextBuilder类：使用普通字符串编写文档
- HTMLBuilder类：使用HTML编写文档

### 2.1 类的一览表
名字 | 说明
----------|---------
 Builder | 定义文档结构的方法的抽象类
 Director|编写一个文档的类
 TextBuilder|使用纯文本编写文档
 HTMLBuilder|使用HTML编写文档
 Main|测试程序行为的类
### 2.2 示例程序类图
![uml](https://raw.githubusercontent.com/little-motor/uml/master/Builder%E6%A8%A1%E5%BC%8F.png)
### 2.3 Builder类
```
/**
 * 声明编写文档的抽象类，close是完成文档编写的方法
 * @author littlemotor
 * @since 18.9.13
 */
public abstract class Builder {
  public abstract void makeTitle(String title);
  public abstract void makeString(String str);
  public abstract void makeItems(String[] items);
  public abstract void close();
}
```
### 2.4 Director类
使用Builder类中声明的方法来编写文档，Director类的构造函数的参数是抽象类Builder的实现类，construct方法是编写文档的方法。
```
/**
 * 调用Builder实现类中的方法，完成文档的编写
 * @author littlemotor
 * @since 18.8.13
 */
public class Director {
  private Builder builder;
  
  public Director(Builder builder) {
    this.builder = builder;
  }
  
  //构造文档的方法
  public void constract() {
    builder.makeTitle("Greeting");
    builder.makeString("从早上到下午");
    builder.makeItems(new String[] {
        "早上好",
        "下午好"
    });
    builder.makeString("晚上");
    builder.makeItems(new String[] {
        "晚上好",
        "晚安"
    });
    builder.close();
  }
}
```
### 2.5 HTMLBuilder类
Builder子类，使用HTML编写文档
```
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * Builder子类，输出HTML格式文档
 * @author littlemotor
 * @version 18.8.13
 */
public class HTMLBuilder extends Builder{
  private String filename;
  private PrintWriter writer;
  
  @Override
  public void makeTitle(String title) {
    filename = title + ".html";
    try {
      writer = new PrintWriter(new FileWriter(filename));
    } catch(IOException e) {
      e.printStackTrace();
    }
    writer.println("<html><head><title>" + title + "</title></head><body>");
    writer.println("<h1>" + title + "</h1>");
  }

  @Override
  public void makeString(String str) {
    writer.println("<p>" + str + "</p>" );
  }
  
  @Override
  public void makeItems(String[] items) {
    for(int i = 0; i < items.length; i++) {
      writer.println("<li>" + items[i] + "</li>");
    }
    writer.println("</ul>");
  }

  @Override
  public void close() {
    writer.println("</body></html>");
    writer.close();
  }
  
  public String getResult() {
    return filename;
  }
}
```
### 2.6 TextBuilder类
使用纯文本编写文档并返回结果
```
/**
 * Builer实现类，使用纯文本编写文档
 * @author littlemotor
 * @version 18.8.13
 */
public class TextBuilder extends Builder{
  private StringBuffer buffer = new StringBuffer();
  
  @Override
  public void makeTitle(String title) {
    buffer.append("====================");
    buffer.append("「" + title + "」");
    buffer.append("\n");
  }

  @Override
  public void makeString(String str) {
    buffer.append(str + "\n");
    buffer.append("\n");
  }

  @Override
  public void makeItems(String[] items) {
    for(int i = 0; i < items.length; i++) {
      buffer.append("." + items[i] + "\n");
    }
    buffer.append("\n");
  }

  @Override
  public void close() {
    buffer.append("==================");
  }

  public String getResult() {
    return buffer.toString();
  }
}
```
### 2.7 BuilderMain类
测试类，使用下面命令编写相应格式的文档：
```
java BuilderMain plain:编写纯文本文档
java BuilderMain html:编写HTML格式文档
```
代码如下：
```
public class BuilderMain {
  public static void main(String[] args) {
    if(args.length != 1) {
      usage();
      System.exit(0);
    }
    if(args[0].equals("plain")) {
      TextBuilder textbuilder = new TextBuilder();
      Director director = new Director(textbuilder);
      director.constract();
      String result = textbuilder.getResult();
      System.out.println(result);
    }
    else if(args[0].equals("html")) {
      HTMLBuilder htmlbuilder = new HTMLBuilder();
      Director director = new Director(htmlbuilder);
      director.constract();
      String filename = htmlbuilder.getResult();
      System.out.println(filename + "文档编写完成。");
      
    } else {
      usage();
      System.exit(0);
    }
  }
  
  public static void usage() {
    System.out.println("Usage: java BuilderMain plain     编写纯文本文档");
    System.out.println("Usage: java BuilderMain html      编写HTML文档");
  }
}
```
## 3. Builder模式中的角色
### 3.1 Builder（建造者）
Builder角色定义生成实例的接口（API），准备了用于生成实例的方法。
### 3.2 ConcreteBuilder（具体的建造者）
实现Builder角色的接口的类，定义了实际被调用的方法，并且还定义了最终生成结果的方法。
### 3.3 Director(监工)
Director角色负责使用Builder角色的接口（API）生成示例。
## 4. 小结
让组件具有可替换性才会具有更高的价值，组装的具体过程被隐藏在Director角色中。