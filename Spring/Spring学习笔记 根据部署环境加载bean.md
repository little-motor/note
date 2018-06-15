# 1.引言
在开发过程中遇到的一大挑战就是将应用程序从一个环境部署到另一个环境，Spring为环境相关的bean提供了很好的解决方案。在这个过程中Spring会根据环境决定该创建哪个bean和不创建哪个bean。不过Spring并不是在构建的时候做出这样的决策，而是等到运行时再来确定。
这样的结果就是同一个部署单元（可能是WAR文件）能够适用于所有的环境，没有必要进行重新构建。
# 2.配置profile bean
在3.1版本，Spring引入了bean profile的功能。在Java配置中，可以使用@Profile注解指定某个bean属于哪一个profile。在应用部署到相应的环境中时，只要确保相应的profile处于激活状态就可以执行相关的bean。部分代码如下
```
package com.myapp;
import ...

@Configuration
public class DataSourceConfig{
  
  @Bean
  @Profile("dev") //开发环境使用这个bean，通过嵌入式数据库获得DataSource
  public DataSource embeddedDataSource(){
    return ...
  }

  @Bean
  @Profile("prod") //生产环境使用这个bean，通过JNDI从容器获取DataSource
  public DataSource jndiDataSource()
  {
    return ...
  }
}
```
注意，没有指定profile的bean始终都会创建，与激活哪个profile没有关系。
同时XML中也可以通过&lt;beans>元素的profie属性进行设置，这里就不过多的讲解了。那么问题来了：我们该怎样激活某个profile呢？
#3.激活profile
Spring在确定哪个profile处于激活状态时，需要依赖两个独立的属性：spring.profiles.active
spring.profiles.default
如果设置了spring.profiles.active属性的话，那么它的值就会用来确定哪个profile是激活的。但如果没有设置spring.profiles.active属性,Spring就会查找spring.profiles.default的值。如果两个都没有设置的话，那就没有激活的profile，因此只会创建那些没有定义在profile中的bean。
有多种方式来设置这两个属性：
* **作为DispatcherServlet的初始化参数；**
* **作为Web应用的上下文参数；**
* **作为JNDI条目；**
* **作为环境变量；**
* **作为JVM的系统属性**
* **在集成测试类上，使用@ActiveProfiles注解设置**
例如在Web应用中，设置spring.profiles.default的web.xml文件会如下所示：
```
<?xml version="1.0" encodind="UTF-8"?>
<web-app version="2.5"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
  http://java.sun.com/sml/ns/javaee/web-app_2_5.xsd>

  <!-- 为上下文设置默认的profile -->
  <context-param>
    <param-name>spring.profile.default</param-name>
  </context-param>

  <!-- 为Servlet设置默认的profile -->
  <servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    <servlet-class>
    <init-param>
      <param-name>spring.profile.default</param-name>
      <param-value>dev</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

</web-app>
```
当应用程序部署到QA、生产或其他环境之中时，负责部署的人根据情况使用系统属性、环境变量、或JNDI设置spring.profiles.active即可。当设置spring.profiles.active以后，置于spring.profiles.default设置成什么值就已经无所谓了。
# 4.条件化的bean
在Spring4之后通过@Conditional注解，可以实现更加复杂的条件化配置，如果给定的条件计算结果为true，就会创建这个bean，否则的化，这个bean就会被忽略。示例如下
```
@Bean
//条件化的创建bean
@Conditional(MagicExistsCondition.class)
publc MagicBean magicBean(){
  return new MagicBean();
}
```
@Conditional给定了一个Class，他指明了条件（本例中也就是MagicExistsCondition）,
@Conditional将会通过Condition接口进行对比：
```
public interface Condition{
  boolean matcher(ConditionContext context,AnnotatedTypeMetadata metadata);
}
```
设置给@Conditional的类可以是任意实现了Condition接口的类型，如果matches()方法返回turn就创建这个bean，反之，不会创建。下面是MagicExistsCondition类
```
package ...

import ...

public class MagicExistsCondition implements Condition{
  @Override
  public boolean matches(
    ConditionContext context,AnnotatedTypeMetadata metadata
  ){
    Environment env = context.getEnvironment();
    //假设此时需要的属性为“magic”
    return env.containsProperty("magic");
  }
}
```
他通过给定的ConditonContext对象得到Environment对象，并使用这个对象检查环境中是否存在名为magic对象的环境属性（属性的值是什么无所谓，只是举例，只要属性存在即可满足条件）
AnnotatedTypeMetadata则能够让我们检查带有@Bean注解的方法上还有什么其他的注解。
同时@Profile注解在Spring4中也进行了重构，使其基于@Conditional和Condition实现，@Profile注解部分源码如下
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile{
  String[] value();
}
```
@Profile本身也使用了@Conditional注解，并且引用ProfileCondition作为Condition实现。ProfileCondition实现了Condition接口，并且在作出决策的过程中，考虑了ConditionContext和AnnotatedTypeMetadata中的多个因素。