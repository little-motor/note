[toc]
# 1. 引言
Servlet API提供了一系列的事件和实践监听接口，监听器接口可以分为三类：ServletContext,HttpSession和ServletRequest,他们都继承自java.util.EventListener标记接口。
# 2. 监听器接口和注册
主要有以下这些接口
- javax.servlet.ServletContextListener:能够响应ServletContext声明周期事件，提供了ServletContext创建之前和关闭之前调用的方法。
- javax.servlet.ServletContextAttributeListener:他能够响应ServletContext范围的属性添加、删除、替换事件。
- javax.servlet.http.HttpSessionListener:能够响应HttpSession创建、超时和失效时间。
- javax.servlet.http.HttpSessionAttributeListener:能响应HttpSession范围属性的添加、删除和替换事件
- javax.servlet.http.HttpSessionActivationListener:HttpSession激活或者失效时被调用
- javax.servlet.http.HttpSessionBindingListener：使对象在绑定到会话或从会话解除绑定时得到HttpSessionBindingEvent事件通知。
- javax.servlet.ServletRequestListener:响应ServletRequest的创建或删除
- javax.servlet.ServletRequestAttributeListener: 能响应ServletRequest范围的属性值添加、删除、修改事件
## 2.1 编写监听器格式
有两种注册监听器的方法，第一种是使用WebListener注解
```
@WebListener
public class ListenerClass implements ListenerInterface{

}
```
第二种方式是在部署描述文档中增加一个listener元素
```
<listener>
  <listener-class>fully-qualified listener class</listener-class>
</listener>
```
# 3. ServletContext监听器
ServletContext监听器接口有两个：ServletContextListener和ServletContextAttributeListener。
## 3.1 ServletContextListener
ServletContextListener能对ServletContext的创建和销毁作出响应
```
/** 
 * Interface for receiving notification events about ServletContext
 * lifecycle changes.
 *
 * <p>In order to receive these notification events, the implementation
 * class must be either declared in the deployment descriptor of the web
 * application, annotated with {@link javax.servlet.annotation.WebListener},
 * or registered via one of the addListener methods defined on
 * {@link ServletContext}.
 *
 * 他们的加载顺序是按照声明顺序确定的
 */
@WebListener
public class MyListener implements ServletContextListener {

    /**
     * Receives notification that the web application initialization
     * process is starting.
     *
     * 这个方法运行在filter和servlet初始化之前
     * that is being initialized
     */
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("before initialization");
    }
    /**
     * Receives notification that the ServletContext is about to be
     * shut down.
     *
     * 在此方法收到通知前，所有的servlet和filter已经被销毁
     *
     */
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("after initialization");
    }
}
```
ServletContextListener包含的两个方法都会从容器获得一个ServletContextEvent实例，他是EventObject的子类，定义了访问ServletContext的getServletContext()方法。
```
public class ServletContextEvent extends java.util.EventObject { 

    /** Construct a ServletContextEvent from the given context.
     *
     * @param source - the ServletContext that is sending the event.
     */
    public ServletContextEvent(ServletContext source) {
        super(source);
    }
    
    /**
     * Return the ServletContext that changed.
     *
     * @return the ServletContext that sent the event.
     */
    public ServletContext getServletContext () { 
        return (ServletContext) super.getSource();
    }
}
```
## 3.2 ServletContextAttributeListener
当一个ServletContext范围的属性被添加、删除或者替换时，此接口的实现类会接收到消息，这个接口定义了如下三个方法
```
void attributeAdded(ServletContextAttributeEvent event);
void attributeRemoved(ServletContextAttributeEvent event);
void attributeReplaced(ServletContextAttributeEvent event);
```
ServletContextAttributeEvent继承自ServletContextEvent，并且增加了两个方法，分别用户获得该属性的名称和值
```
/** 
 * Event class for notifications about changes to the attributes of
 * the ServletContext of a web application.
  */

public class ServletContextAttributeEvent extends ServletContextEvent { 

    private String name;
    private Object value;

    /**
     * @param source the ServletContext whose attribute changed
     * @param name the name of the ServletContext attribute that changed
     * @param value the value of the ServletContext attribute that changed
     */
    public ServletContextAttributeEvent(ServletContext source,
            String name, Object value) {
        super(source);
        this.name = name;
        this.value = value;
    }
	
    /**
     * Gets the name of the ServletContext attribute that changed.
     *
     * @return the name of the ServletContext attribute that changed
     */
    public String getName() {
        return this.name;
    }
	
    /**
     * Gets the value of the ServletContext attribute that changed.
     *
     * <p>If the attribute was added, this is the value of the attribute.
     * If the attribute was removed, this is the value of the removed
     * attribute. If the attribute was replaced, this is the old value of
     * the attribute.
     *
     * @return the value of the ServletContext attribute that changed
     */
    public Object getValue() {
        return this.value;   
    }
}
```
# 4. SessionListener
一共有四个HttpSession相关的监听器接口：HttpSessionListener,HttpSessionActivationListener,HttpSessionAttributeListener和HttpSessionBindingListener，他们都在javax.servlet.http包中
## 4.1 HttpSessionListener
当HttpSession创建或者销毁时，容器会通知所有的HttpSessionListener，这个接口有两个方法
```
void sessionCreated(HttpSessionEvent event);
void sessionDestroy(HttpSessionEvent event);
```
HttpSessionEvent是继承自EventObject的类，可以调用HttpSessionEvent的getSession()方法获取当前的HttpSession
```
/**
 * This is the class representing event notifications for changes to
 * sessions within a web application.
 */
public class HttpSessionEvent extends java.util.EventObject {

     /**
     * Construct a session event from the given source.
     */
    public HttpSessionEvent(HttpSession source) {
        super(source);
    }

    /**
     * Return the session that changed.
     */
    public HttpSession getSession () { 
        return (HttpSession) super.getSource();
    }
}
```
## 4.2 HttpSessionAttributeListener
他响应的是HttpSession范围属性的添加、删除和替换，包含三个方法
```
void attributeAdded(HttpSessionBindingEvent event);
void attributeRemoved(HttpSessionBindingEvent event);
void attributeReplaced(HttpSeesionBindingEvent event);
```
HttpSessionBindingEvent是HttpSessionEvent的子类，可以获得HttpSession的同时实现了另外两个方法,可以获得对应属性的名称和值。
```
String getName();
Object getValue();
```
## 4.3 HttpSessionActivationListener
在分布式环境下，会用多个容器来进行负载均衡，有可能需要将session保存起来，在容器之间传递，例如在一个容器内存不足时，会把很少用到的对象转存到其他容器上，这时候容器就会通知所有HttpSessionActivationListener接口实现类
```
    /** 通知会话即将被钝化*/
    public void sessionWillPassivate(HttpSessionEvent se); 
    /** 通知会话刚刚被激活*/
    public void sessionDidActivate(HttpSessionEvent se);
```
## 4.4 HttpSessionBindingListener
在发生下面三种情况之一时，触发事件
- 执行session.invalidate()时。
- session超时，自动销毁时。
-  执行session.setAttribute("keyName", Object)或session.removeAttribute("keyName")

这个接口的实现类可以用于统计实时登陆人数，包含两个方法：
```
 public void valueBound(HttpSessionBindingEvent event);
 public void valueUnbound(HttpSessionBindingEvent event);
```
