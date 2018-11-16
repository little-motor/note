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
public interface ServletContextListener extends EventListener {

    /**
     * Receives notification that the web application initialization
     * process is starting.
     *
     * <p>All ServletContextListeners are notified of context
     * initialization before any filters or servlets in the web
     * application are initialized.
     *
     * @param sce the ServletContextEvent containing the ServletContext
     * that is being initialized
     */
    public void contextInitialized(ServletContextEvent sce);

    /**
     * Receives notification that the ServletContext is about to be
     * shut down.
     *
     * <p>All servlets and filters will have been destroyed before any
     * ServletContextListeners are notified of context
     * destruction.
     *
     * @param sce the ServletContextEvent containing the ServletContext
     * that is being destroyed
     */
    public void contextDestroyed(ServletContextEvent sce);
}

```
