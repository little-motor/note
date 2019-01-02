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




