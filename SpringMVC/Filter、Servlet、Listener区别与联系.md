# Servlet、Filter、Listener区别与联系
# 1.Servlet
## 1.1 Servlet接口
Servlet运行在Servlet容器中，其生命周期由容器来管理。Servlet接口总共提供五个接口，其生命周期通过Servlet接口中的init( )、service( )和destory( )方法来表示。
```
    public interface Servlet {  
        //1.创建
        public void init(ServletConfig config) throws ServletException;  

        //Servlet获得ServletContext运行环境从而与Servlet容器通信
        public ServletConfig getServletConfig(); 

        //2.服务，包含客户请求参数，与servlet处理后的响应参数
        public void service(ServletRequest req, ServletResponse res)  
        throws ServletException, IOException;  
      
        public String getServletInfo();  

        //3.销毁，容器检测到Servlet对象应该被移除时，该对象会被Java垃圾收集器回收
        public void destroy();  
    }  
```
## 1.2 Servlet生命周期

![Servlet生命周期](https://img-blog.csdn.net/20180625200429982?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5Mzg1MTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
#2.Filter
Filter可认为是Servlet的一种“变种”，它主要用于对用户请求进行预处理，也可以对HttpServletResponse进行后处理，是个典型的处理链。它与Servlet的区别在于：它不能直接向用户生成响应。完整的流程是：Filter对用户请求进行预处理，接着将请求交给 Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。 
## 2.1 Filter接口
```
public interface Filter {  
  //用于完成Filter的初始化  
  public void init(FilterConfig filterConfig) throws ServletException; 

  //实现过滤功能  
  public void doFilter ( ServletRequest request, ServletResponse response,   
    FilterChain chain ) throws IOException, ServletException;  
        
  //用于销毁Filter前，完成某些资源的回收  
  public void destroy();  
} 
```
## 2.2 Filter生命周期

web.xml 中声明的每个 filter 在每个虚拟机中仅仅只有一个实例。
#### 2.2.1 加载和实例化
Web 容器启动时，即会根据 web.xml 中声明的 filter 顺序依次实例化这些 filter。Web 容器调用 init(FilterConfig filterConfig) 并传递FilterConfig对象来初始化过滤器。FilterConfig 对象可以得到 ServletContext 对象，以及在 web.xml 中配置的过滤器的初始化参数。实例化和初始化的操作只会在容器启动时执行，而且只会执行一次。
####  2.2.2 doFilter
当客户端请求目标资源的时候，容器会筛选出符合 filter-mapping 中的 url-pattern 的 filter，并按照声明 filter-mapping 的顺序依次调用这些 filter 的 doFilter 方法。在这个链式调用过程中，可以调用 chain.doFilter(ServletRequest, ServletResponse) 将请求传给下一个过滤器(或目标资源)，
#### 2.2.3 销毁
Web 容器调用 destroy 方法指示过滤器的生命周期结束。在这个方法中，可以释放过滤器使用的资源。
## 2.3 Filter链
在一个web应用中，可以开发编写多个Filter，这些Filter组合起来称之为一个Filter链。web服务器根据Filter在web.xml文件中先后顺序依次调用，当第一个Filter的doFilter方法被调用时，web服务器会创建一个代表Filter链的FilterChain对象传递给该方法。在doFilter方法中，开发人员如果调用了FilterChain对象的doFilter方法，则web服务器会检查FilterChain对象中是否还有filter，如果有，则调用第2个filter，如果没有，则调用目标资源。    
![Filter处理过程](https://img-blog.csdn.net/20180625202922512?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5Mzg1MTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  
#3.Listener
Servlet,Filter都是针对url之类的，而Listener是响应事件的操作的，如session的创建，session.setAttribute的发生，或者在启动服务器的时候将你需要的数据加载到缓存等，在这样的事件发生时做一些事情。
## 3.1 Listener分类

不同功能的Listener 需要实现不同的 Listener  接口，一个Listener也可以实现多个接口，这样就可以多种功能的监听器一起工作。常用监听器：

#### 3.1.1生命周期监听器：
监听ServletContext、HttpSession、ServletRequest三个对象创建和销毁

ServletContextListener接口：
```
//监听ServletContext的生命周期变化
javax.servlet.ServletContextListener
```
方法：
```
//在ServletContext对象创建后调用
void contextInitialized(ServletContextEvent sce)
//在ServletContext对象销毁前调用
void contextDestroyed(ServletContextEvent sce)
```                    
HttpSessionListener接口：
```
//监听HttpSession对象的生命周期变化
javax.servlet.http.HttpSessionListener 
```     
方法:
```
//在HttpSession对象创建以后调用
void sessionCreated(HttpSessionEvent se)
//在HttpSession对象销毁前调用
void sessionDestroyed(HttpSessionEvent se)
```
同样的Listener还有ServletRequestListener接口
#### 3.1.2 属性监听器：            
ServletContextAttributeListener接口：              
```
 //ServletContext属性变化监听器
 javax.servlet.ServletContextAttributeListener
```               
 方法：
```
//当向application域中添加属性时调用
void attributeAdded(ServletContextAttributeEvent scab)

//当从application域中移除属性时调用
void attributeRemoved(ServletContextAttributeEvent scab)

//当application域中一个属性被替换时调用
void attributeReplaced(ServletContextAttributeEvent scab)
```        
另外还有HttpSession的属性变化监听器
javax.servlet.http.HttpSessionAttributeListener
                
ServletRequest属性变化监听器
javax.servlet.ServletRequestAttributeListener
               
#### 3.1.3 监听Session中指定类的实例属性变化的监听器
HttpSessionBindingListener
HttpSessionActivationListener

# 参考资料
##### [Filter，Servlet，Listener区别与联系](https://zhidao.baidu.com/question/625228544295061444.html)
#####[ Servlet、Filter、Listener深入理解](https://blog.csdn.net/sunxianghuang/article/details/52107376)
#####[Listener监听器与Filter过滤器](https://www.cnblogs.com/libingbin/p/5985647.html)
