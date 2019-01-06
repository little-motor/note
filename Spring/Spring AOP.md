[toc]
# 1. 引言
日志、安全和事务管理都很重要，但他们不应该成为对象主动参与的行为，AOP(Aspect Oriented Programming)面向切面编程就是让应用对象只关注自己所针对的业务领域问题，散布于应用中多处的功能被称为横切关注点(cross-cutting concern)，这些横切关注点从概念上是与应用的业务逻辑相分离的（但是往往会直接嵌入到应用的业务逻辑之中），把这些横切关注点与业务逻辑相分离正是面向切面编程所要解决的问题。
# 2. 约定编程（底层原理）
抛开AOP的概念，先来看一个约定编程的实例，他和Spring AOP有异曲同工之妙
## 2.1 简易接口
```
package designPattern.aop.interf;

public interface HelloService{
    public void sayHello(String name);
}
```
## 2.2 简易接口实现类
```
package designPattern.aop;

import designPattern.aop.interf.HelloService;

public class HelloServiceImp implements HelloService {

    @Override
    public void sayHello(String name){
        if(name == null || name.trim() == ""){
            throw new RuntimeException("parameter is null");
        }
        System.out.println("hello" + name);
    }
}
```
## 2.3 拦截器接口
```
package designPattern.aop.interf;

import designPattern.aop.Invocation;

import java.lang.reflect.InvocationTargetException;

public interface Interceptor{
    //事前方法
    public boolean before();

    //事后方法
    public void after();

    /**
     * 取代原有事件
     * @param invocation  回调参数，可以通过他的proceed方法，回调原有事件
     */
    public Object around(Invocation invocation)throws InvocationTargetException, IllegalAccessException;

    //事后返回方法，事件没有发生异常执行
    public void afterReturning();

    //事后异常方法，当事件发生异常后执行
    public void afterThrowing();

    //是否使用around方法取代原有方法
    boolean useAround();
}
```
Invocation源代码如下
```
package designPattern.aop;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Invocation {

    private Object target;
    private Method method;
    private Object[] params;

    public Invocation(Object target, Method method, Object[] params){
        this.target = target;
        this.method = method;
        this.params = params;
    }

    //反射方法
    public Object proceed() throws InvocationTargetException, IllegalAccessException{
        return method.invoke(target, params);
    }

    public Object getTarget() {
        return target;
    }

    public void setTarget(Object target) {
        this.target = target;
    }

    public Method getMethod() {
        return method;
    }

    public void setMethod(Method method) {
        this.method = method;
    }

    public Object[] getParams() {
        return params;
    }

    public void setParams(Object[] params) {
        this.params = params;
    }
}

```
##  2.4 开发自己的拦截器
```
package designPattern.aop;

import designPattern.aop.interf.Interceptor;

import java.lang.reflect.InvocationTargetException;

/**
 * @author littlemotor
 * @date 19.1.2
 */
public class InterceptorImp implements Interceptor {
    @Override
    public boolean before(){
        System.out.println("before ...");
        return true;
    }

    @Override
    public boolean useAround(){
        return true;
    }

    @Override
    public void after(){
        System.out.println("after ...");
    }

    @Override
    public Object around(Invocation invocation) throws InvocationTargetException, IllegalAccessException{
        System.out.println("around before ...");
        Object object = invocation.proceed();
        System.out.println("around after ...");
        return object;
    }

    @Override
    public void afterReturning(){
        System.out.println("afterReturning ...");
    }

    @Override
    public void afterThrowing(){
        System.out.println("afterThrowing ...");
    }
}

```
约定是核心同时也是AOP的本质，关于动态代理的知识请至[传送门](https://blog.csdn.net/qq_39385118/article/details/83859043)查看我的另一篇文章，当调用proxy对象的方法时，我们约定如下
1. 使用proxy调用方法时会执行拦截器的before方法。
2. 如果拦截器的useAround方法返回true，则执行拦截器around方法，而不调用target对象对应的方法，但around方法的参数invocation对象存在proceed方法，可以调用target对象对应的方法；如果useAround方法返回false，则直接调用target对象的事件方法。
3. 无论怎样，在完成之前的事情后，都会执行拦截器的after方法。
4. 在执行around方法或回调target的事件方法时，可能会发生异常，也可能不发生异常。如果发生异常，就执行拦截器的afterThrowing方法，否则就执行afterReturning方法。
   

![AOP约定流程](https://raw.githubusercontent.com/little-motor/uml/master/AOP/AOP%E9%A2%84%E5%AE%9A%E6%B5%81%E7%A8%8B.png)
<center>约定流程图 </center>

## 2.5 ProxyBean实现
我们现在需要做的就是将服务类和拦截方法织入到对应的流程，此处会涉及到动态代理的知识，由于篇幅有限请至传送门查看[动态代理](https://blog.csdn.net/qq_39385118/article/details/83859043)
```
/*
 * @littlemotor
 * @date
 */

package designPattern.aop;

import designPattern.aop.interf.Interceptor;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * aop底层实现的学习，通过jdk动态代理生成代理对象，同时通过invoke织入方法
 * @author littlemotor
 * @date 19.1.2
 */
public class ProxyBean implements InvocationHandler {

    private Object targe = null;
    private Interceptor interceptor = null;

    public static Object getProxyBean(Object targe, Interceptor interceptor) {
        ProxyBean proxyBean = new ProxyBean();
        //保存被代理对象
        proxyBean.targe = targe;
        //保存拦截器
        proxyBean.interceptor = interceptor;
        //生成代理对象
        Object proxy = Proxy.newProxyInstance(targe.getClass().getClassLoader(), targe.getClass().getInterfaces(), proxyBean);
        return proxy;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //异常标记
        boolean exceptinFlag = false;
        Object returnObj = null;

        Invocation invocation = new Invocation(targe,method,args);
        try{
            if (this.interceptor.before()){
                returnObj = this.interceptor.around(invocation);
            }else{
                returnObj = method.invoke(targe, args);
            }
        } catch (Exception e){
            //产生异常
            exceptinFlag = true;
        }
        this.interceptor.after();
        if (exceptinFlag) {
            this.interceptor.afterThrowing();
        } else {
            this.interceptor.afterReturning();
            return returnObj;
        }
        return null;
    }
}
```
## 2.6 main方法
```
/*
 * @littlemotor
 * @date
 */

package designPattern.aop;

import designPattern.aop.interf.HelloService;

/**
 * 检验AOP实例，当sayHello方法的参数不同是显示和执行过程也不同
 * @author littlemotor
 * @date 19.1.2
 */
public class AOPmain {
    public  static void main(String[] args){
        HelloService helloService = new HelloServiceImp();
        //按约定获取proxy
        HelloService proxy = (HelloService)ProxyBean.getProxyBean(helloService, new InterceptorImp());
        //执行代理对象的拦截过程
        proxy.sayHello("little motor");

        System.out.println("#######参数为null的情况###########");
        proxy.sayHello(null);
    }
}
```
实现结果如下
```
around before ...
hellolittle motor
around after ...
after ...
afterReturning ...
#######参数为null的情况###########
before ...
around before ...
after ...
afterThrowing ...
```
# 3. Spring AOP详解
前面并没有讲AOP的概念，只是通过一个动态代理的实例了解了AOP织入的本质，Spring AOP也是一种约定流程的编程，可以将代码织入事先约定的流程中。AOP可以减少大量重复工作，是对面向对象编程(OOP)的补充，比如在数据库的事务处理中，其流程图如下所示

![事务流程默认实现](https://github.com/little-motor/uml/blob/master/AOP/%E4%BA%8B%E7%89%A9%E6%B5%81%E7%A8%8B%E7%9A%84%E9%BB%98%E8%AE%A4%E5%AE%9E%E7%8E%B0.png?raw=true)
<center>事务流程默认实现</center>
在Spring中通过@Transactional注解即可实现，其大致原理就是Spring将sql执行语句织入到类似于上图的流程中。

## 3.1 AOP术语和流程
Spring AOP是一种基于方法的AOP，他只能用于方法上。
- 连接点(join point)：对应的是具体被拦截的方法，AOP通过动态代理技术将它织入到对应的流程中
- 切点(point cut)：有时候切面不单单应用于单个方法，也可能是多个类的不同方法，这时，可以通过正则表达式和指示器的规则去定义，从而适配连接点。切点就是提供提供这个功能的。
- 通知(advice)：约定流程中的执行方式，分为前置通知(before advice),后置通知(after advice),环绕通知(around advice),事后返回通知(afterReturning advice)和异常通知(afterThrowing advice),他会根据预定织入流程。
- 目标对象(target)：即被代理的对象，例如上面的HelloServiceImp实例就是目标对象，他被代理了。
- 引入(introduction)：引入新的类和方法，增强现有的Bean功能。
- 织入(weaving)：通过动态代理技术，为原有服务对象生成代理对象，然后将与切点定义匹配的连接点拦截，并按照约定将各类通知织入约定流程的过程。
- 切面(aspect)：是一个可以定义切点、各类通知和引入的内容，Spring AOP将通过他的信息来增强Bean的功能或者将对应的方法织入流程。

备注一些，Spring AOP中实现动态代理的方式有JDK动态代理和CGLIB动态代理，不同版本中默认的实现方式不同，在Spring Boot 2.1.1中默认使用CGLIB实现动态代理，具体设置可以查看类AopAutoConfiguration的设置，通过application.yml可以修改
```
#默认true时使用cglib,false使用jdk动态代理，AopAutoConfiguration中默认使用true
spring:
  aop:
    proxy-target-class: false  #修改为jdk动态代理
```
## 3.2 AOP开发流程详解
### 3.2.1 确定连接点
用户服务接口
```
package cn.littlemotor.web.aspect.service.interf;

import cn.littlemotor.web.model.User;

/**
 * 用于切面中的接口，在imp包中被实现，然后可以被动态代理包装为面的一部分
 * @author littlemotor 
 * @date 19.1.5
 */

public interface UserService {
    public void printUser(User user);
}
```
用户服务接口实现类
```
package cn.littlemotor.web.aspect.service.imp;

import cn.littlemotor.web.aspect.service.interf.UserService;
import cn.littlemotor.web.model.User;
import org.springframework.stereotype.Component;

/**
 * 用户接口实现类
 * @author littlemotor
 * @date 19.1.6
 */
@Component
public class UserServiceImpl implements UserService {
    @Override
    public void printUser(User user) {
        System.out.println("hi: " + user.getName());
    }
}
```
### 3.2.2 开发切面
以printUser方法作为连接点创建一个切面类
```
package cn.littlemotor.web.aspect;

import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * 面向切面编程终端切面部分
 * @author littlemotor
 * @date 19.1.5
 */
@Component
@Aspect
public class MyAspect {

    @Pointcut("execution(* cn.littlemotor.web.aspect.service.imp.UserServiceImpl.printUser(..))")
    public void pointCut(){
    }

    @Before("pointCut()")
    public void before(){
        System.out.println("before");
    }

    @After("pointCut()")  //无论有没有抛出错误都会执行
    public void after(){
        System.out.println("after");
    }

    @AfterReturning("pointCut()")
    public void afterReturning(){
        System.out.println("afterReturning");
    }

    @AfterThrowing("pointCut()")
    public void afterThrowing(){
        System.out.println("afterThrowing");
    }
}
```
Spring通过@Aspect注解来声明该类为一个切面，并且可以在内部定义各类通知和切点，@Pointcut通过正则表达式的方式定义什么时候启用AOP，别的通知可以直接引用被@Pointcut注解的方法名。
简单的分析一下切点中的正则表达式：
execution(* cn.littlemotor.web.aspect.service.imp.UserServiceImpl.printUser(..))
- excution表示在执行的时候，拦截里面的正则匹配的方法
- * 表示任意返回类型的方法
- cn.littlemotor.web.aspect.service.imp.UserServiceImpl表示目标对象的全限定名
- printUser指定目标对象的方法
- (..)表示任意参数进行匹配
  
AspectJ关于Spring AOP切点的指示器如下

项目类型 | 描述
---------|----------
 arg() | 限定连接点方法参数
@args() | 通过连接点方法参数上的注解进行限定
 execution() | 用于匹配是连接点的执行方法
 this()|限制连接点匹配AOP代理Bean引用为指定类型
 target|目标对象(即被代理对象)
 @target()|限制目标对象配置了指定的注解
 within|限制连接点匹配指定的类型
 @within()|限定连接点带有匹配注解类型
 @annotation()|限定带有指定注解的连接点

### 3.2.3 测试AOP
在一个Web开发环境中进行测试，controller如下
```
package cn.littlemotor.web.controller;

import cn.littlemotor.web.aspect.service.imp.UserServiceOnlyClass;
import cn.littlemotor.web.aspect.service.interf.UserService;
import cn.littlemotor.web.model.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * 测试aop切面编程
 * @author littlemotor
 * @date 19.1.5
 */
@Controller
@RequestMapping("/user")
public class UserController {


    @Autowired
    private UserService userService = null;

    @Autowired
    private UserServiceOnlyClass userServiceOnlyClass = null;

    @RequestMapping("/print")
    @ResponseBody
    public User printUser(int id, String name, String note) {

        User user = new User();
        user.setId(id);
        user.setName(name);
        user.setNote(note);
        userService.printUser(user);
        userServiceOnlyClass.printUser(user);
        return user;
    }
}
```

Spring Boot配置启动文件
```
package cn.littlemotor.web;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 *
 */
@SpringBootApplication(scanBasePackages = {"cn.littlemotor.web.aspect","cn.littlemotor.web.controller"})
public class WebApplication {


	public static void main(String[] args) {
		SpringApplication.run(WebApplication.class, args);
	}

}
```
启动程序后在浏览器中输入localhost:8080/user/print?id=1&name=littlemotor&note=test，就可以在控制台中看到输出结果，这就是一个简单的AOP织入过程。
```
before
hi: littlemotor
after
afterReturning
```

## 3.3 AOP补充
### 3.3.1 环绕通知
其使用场景是需要大幅度修改原有目标对象的服务逻辑时，否则建议使用其他通知，同时他也提供了回调原有目标对象方法的能力。
```
@Around("pointCut()")
public void around(ProceedingJoinPoint jp) throws Throwable {
    System.out.println("around before ...");
    //回调目标对象的原有方法
    jp.proceed();
    System.out.println("around after ...");
}
```
### 3.3.2 引入
前面的方法print只能打出用户的姓名，我们还想要打印出用户的note信息，但是假设这个方法不能由UserServiceImpl提供，而需要接口UserNote的printNote方法提供