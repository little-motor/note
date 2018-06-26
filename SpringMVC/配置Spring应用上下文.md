#配置Spring应用上下文
Spring自带了多种类型的应用上下文，下面罗列几个最有可能遇到的
* AnnotationConfigApplicationContext:从一个或多个基于Java的配置类中加载Spring应用上下文。
* AnnotationConfigWebApplicationContext:从一个或多个基于Java的配置类中加载Spring Web应用上下文。
* ClassPathXmlApplicationContext:从类路径下的一个或多个XML配置中加载上下文定义，把应用上下文的定义文件作为类资源。
* FileSystemXmlApplicationContext:从文件系统下的一个或多个XML配置文件中加载上下文定义。
* XmlWebApplicationContext:从Web应用下的一个或多个XML配置文件中加载上下文定义。

例如从Java配置(Demo类)中加载应用上下文，可以使用AnnotationConfigApplicationContext：
```
ApplicationContext context = new 
                     AnnotationConfigApplicationContext(com.main.Demo.class);
```
