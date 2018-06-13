# 1.通过Java代码装配bean
当需要将第三方库中的组件装配到应用时，就没有办法在这些类上面添加@Component和@Autowired注解，因此需要显示装配方式了，有两种方案：Java代码配置和XML配置，这里着重讲Java配置。
他的优势就是类型安全并且对重构友好，同时也要注意JavaConfig是配置代码，所以不应该包含任何业务逻辑。
# 2.创建配置类
参照上一篇文章[Spring学习笔记 自动化装配bean](https://blog.csdn.net/qq_39385118/article/details/80662265)当中的配置类
```
    package helloWorld;  
      
    import org.springframework.context.annotation.ComponentScan;  
    import org.springframework.context.annotation.Configuration;  
      
    @Configuration  
    @ComponentScan  
    public class HelloWorldConfig {  
      
    }  
```
因为此次关注的是Java代码显式配置，所以就将@Component移除了，此时由于bean没有被扫描到，所以程序会有错误，修改后如下
```
    package helloWorld;  
      
    import org.springframework.context.annotation.Configuration;  
      
    @Configuration  
    public class HelloWorldConfig {  
      
    }  
```
# 3.借助JavaConfig实现注入
这里的关键在于为其添加@Configuration和@bean注解，@Configuration表明这是一个配置类，该类应该包含创建Spring上下文中的bean的细节。@bean注解返回相应类实例的方法。
由于这是Java代码配置，所以实现依赖注入的方法多种多样，唯一的限制可能就是语法了，**只要方法能够返回需要的类的实例，Spring就会将他作为bean**，这里我介绍一种通常情况下的最佳方案，由于是需要注入MessagePrinter实例，代码如下
```
package helloWorld;  
      
    import org.springframework.context.annotation.Configuration;  
      
    @Configuration  
    public class HelloWorldConfig {  
      @bean
      public MessagePrinter message(){
        return new MessagePrinter();
      }
    }  
```
bean的ID与带@bean注解的方法名相同，如果需要注入多个类的实例那就写多个带@bean注解的方法，返回类型为需要的类，如果一个类初始化需要另一个类的实例作为构造参数，其形式可如下代码
```
package helloWorld;  
      
    import org.springframework.context.annotation.Configuration;  
      
    @Configuration  
    public class HelloWorldConfig {  
      @bean
      //这里的ClassName也要生成相应的bean
      public MessagePrinter message(ClassName name){ 
        return new MessagePrinter();
      }

      @bean
      //用相应的类去替换
      public ClassName methodName(){
        return new ClassName();
      }
    }  
```
# 4.小结
这样一来我们就可以将配置分散到多个配置类、XML文件以及自动扫描和装配bean之中，只要功能完整健全即可。