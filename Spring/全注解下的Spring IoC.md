[toc]
# 1. 引言
Spring最成功的是其提出的理念，而不是技术本身，他所依赖的两个核心理念，一个是控制反转(Inversion of Control,IoC)，另一个是面向切面编程(Aspect Oriented Programming,AOP)。IoC容器是Spring的核心，可以说Spring是一个基于IoC容器编程的框架。
Spring Boot并不建议使用XML，而是通过注解的描述生成对象。Spring不仅要生成各种对象，同时还提供了依赖注入功能，使得我们可以通过描述来管理各个对象之间的关系。在Spring中把每一个需要管理的对象称为Spring Bean(简称Bean)，而Spring是管理这些Bean的容器，被我们称为Spring IoC容器（简称IoC容器），IoC容器需要具备两个基本功能：
- 通过描述管理Bean，包括发布和获取Bean
- 通过描述完成Bean之间的依赖关系
```
public interface BeanFactory {

	
	String FACTORY_BEAN_PREFIX = "&";

        //多个getBean的方法
	Object getBean(String name) throws BeansException;

	<T> T getBean(String name, Class<T> requiredType) throws BeansException;

	Object getBean(String name, Object... args) throws BeansException;

	<T> T getBean(Class<T> requiredType) throws BeansException;
	
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);
	
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
    
        //是否包含Bean
	boolean containsBean(String name);

        //是否单例模式
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	//是否原型模式
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	//是否类型匹配
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
	
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

        //获取Bean的类型
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	//获取Bean的别名
	String[] getAliases(String name);

}
```
值得注意的是多个getBean()方法，这也是IoC容器最重要的方法之一，他的意义是从IoC容器中获取Bean，他可以按类型或者名称获取Bean。
默认情况下Spring IoC容器的Bean都是以单例存在的，也就是用getBean()方法返回的都是同一个对象，与isSingleton()方法相反的是isPrototype()方法，用来判断是否为原型模式，如果返回的是true，那么使用getBean()方法返回Bean的时候，Spring IoC容器就会创建一个新的Bean返回给一个调用者。
为了进一步完善IoC容器的功能，在BeanFactory的基础上，还设计了一个更为高级的子接口ApplicationContext。
#2. 装配Bean
## 2.1 通过@Configuration方式
@Configuration代表这是一个Java配置文件，Spring容器会根据他来生成IoC容器去装配Bean
@Bean(String name)会将被其注解的方法返回的实例装配到IoC容器中,name属性定义这个Bean的名称，默认与标准方法名相同。
## 2.2 通过@Component方式
Spring允许通过扫描装配Bean到IoC容器中，对于扫描装配而言使用的注解是@Component和@ComponentScan。@Component标明哪个类被扫描进入Spring IoC容器，而@ComponentScan则是标明采用何种策略去扫描装配Bean。
@Component(String value)设置Bean的名称，默认为类名首字母小写。
@ComponentScan默认扫描当前注解类所在的包和子包,可以通过basePackages,basePackageClasses等修改扫描位置。
# 3. 依赖注入(Dependency Injection)
@Autowird是依赖注入中使用最多的注解之一，关于依赖注入这种设计模式的详细解释请至[传送门](https://baike.baidu.com/item/%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC/1158025?fr=aladdin)，他注入的机制最基本的一条是根据类型查找Bean，即BeanFactory接口中的
```
<T> T getBean(Class<T> requiredType) throws BeansException;
```
需要注意的是@Autowired是一个默认必须找到对应Bean的注解，如果不能确定其标注属性一定存在，可以将required=false,他首先会根据类型查找Bean，如果对应的Bean不是唯一的，那么他会根据属性名称和Bean的名称进行匹配。
## 3.1 消除歧义性——@Primary和@Qualifier
@Primary告诉Spring IoC容器发现多个同样类型的Bean时，优先使用此注解的Bean
```
...
@Component
@Primary
public class demo{
	...
}
```
@Qualifier通过配置项value字符串定义需要的Bean name，与@Autowired组合使用，底层是通过BeanFactory借口的geatBean方法
```
<T> T getBean(String name,Class<T> requiredType) throws BeansException;
```
使用方法为
```
@Autowired
@Qualifier("name")
public ClassName instance = null;
```
# 4. 生命周期
Spring IoC管理Bean的生命周期大致分为4个部分：
Bean定义、Bean初始化、Bean的生存期、Bean的销毁。
## 4.1 Bean定义过程
Spring会通过配置比如@ComponentScan扫面带有@Component注解的类定位资源，解析之后会将定义信息保存起来，然后将Bean定义发布到IoC容器中，注意这个过程中只有Bean的定义而没有实例生成。
![Spring初始化Bean](https://raw.githubusercontent.com/little-motor/uml/master/Spring/SpringInitBean.png)
<center>Spring 初始化Bean</center>
可以通过设置@ComponentScan中的lazyInit来实现延迟加载，默认情况下为false在注入前已经实例化。
## 4.2 Spring Bean生命周期
![Spring Bean生命周期](https://raw.githubusercontent.com/little-motor/uml/master/Spring/Spring%20Bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

这里有几个需要注意的地方：
接口BeanNameAware,BeanFactoryAware,ApplicationContextAware,InitializingBean,DisposableBean分别对应单个Bean的生命周期中的各个阶段：定义，初始化和销毁。而方法postProcessBeforeInitialization和postProcessAfterInitialization是接口BeanPostProcessor中的两个方法，针对的是全部的Bean生效
```
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
	return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
	return bean;
    }
}
```
名为...Aware结尾的接口可以感知到其前面的含义的变化并注入到Bean实例中，例如BeanNameAware便可以感知到Bean的name并注入
```
public interface BeanNameAware extends Aware {
    void setBeanName(String name);
}
```
可以通过图中的注解执行自定义的方法。
