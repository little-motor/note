[toc]
## 1. 引言
Servlet技术的核心是Servlet，Servlet API有以下4个Java包：
- javax.servlet
- javax.servlet.http
- javax.servlet.annotation
- javax.servlet.descriptor

Servlet容器将Servlet类载入内存，并在Servlet实例上调用具体的方法，每个Servlet类型只能有一个实例，按照惯例Servlet类的名称要以Servlet作为后缀。
## 2. Servlet接口
```
package javax.servlet;

import java.io.IOException;

public interface Servlet {

  public void init(ServletConfig config) throws ServletException;

  public ServletConfig getServletConfig();

  public void service(ServletRequest req, ServletResponse res)
	throws ServletException, IOException;

  public String getServletInfo();

  public void destroy();
}
```
init,service和destroy是生命周期方法，getServletInfo和getServetConfig是非生命周期方法。
- init: 当Servlet第一次被请求时，Servlet会调用这个方法，执行初始化动作，Servlet容器会传入一个ServletConfig，一般来说会将Servlet赋值给一个类级别变量
- service：每当请求Servlet时，Servlet容器会调用这个方法
- destroy：当要销毁Servlet时，Servlet会调用这个方法，用于回收资源并且在必要情况下同步内存的状态到持久层中
- getServletInfo: 返回Servlet信息，例如作者、版本等，注意此方法应该返回的是纯文本，而不是任何标记语言。
- getServletConfig: 返回init方法中传入的ServletConfig，ServletConfig应该被保存在servlet实现类中的类级变量中。

### 2.1 一个简单的Servlet应用程序
```
package servlet;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;

/**
 * Servlet温故而知新
 * @author littlemotor
 * @date 18.11.13
 */
@WebServlet(name = "MyServlet", urlPatterns = {"/my"})
public class MyServlet implements Servlet {

    ServletConfig servletConfig;


    /**
     * Called by the servlet container to indicate to a servlet that the
     * servlet is being placed into service.
     *
     * <p>The servlet container calls the <code>init</code>
     * method exactly once after instantiating the servlet.
     * The <code>init</code> method must complete successfully
     * before the servlet can receive any requests.
     *
     * <p>The servlet container cannot place the servlet into service
     * if the <code>init</code> method
     * <ol>
     * <li>Throws a <code>ServletException</code>
     * <li>Does not return within a time period defined by the Web server
     * </ol>
     *
     * @param config a <code>ServletConfig</code> object
     *               containing the servlet's
     *               configuration and initialization parameters
     * @throws ServletException if an exception has occurred that
     *                          interferes with the servlet's normal
     *                          operation
     * @see UnavailableException
     * @see #getServletConfig
     */
    @Override
    public void init(ServletConfig config) throws ServletException {

        this.servletConfig = config;
    }

    /**
     * Returns a {@link ServletConfig} object, which contains
     * initialization and startup parameters for this servlet.
     * The <code>ServletConfig</code> object returned is the one
     * passed to the <code>init</code> method.
     *
     * <p>Implementations of this interface are responsible for storing the
     * <code>ServletConfig</code> object so that this
     * method can return it. The {@link GenericServlet}
     * class, which implements this interface, already does this.
     *
     * @return the <code>ServletConfig</code> object
     * that initializes this servlet
     * @see #init
     */
    @Override
    public ServletConfig getServletConfig() {
        return servletConfig;
    }

    /**
     * Called by the servlet container to allow the servlet to respond to
     * a request.
     *
     * <p>This method is only called after the servlet's <code>init()</code>
     * method has completed successfully.
     *
     * <p>  The status code of the response always should be set for a servlet
     * that throws or sends an error.
     *
     *
     * <p>Servlets typically run inside multithreaded servlet containers
     * that can handle multiple requests concurrently. Developers must
     * be aware to synchronize access to any shared resources such as files,
     * network connections, and as well as the servlet's class and instance
     * variables.
     * More information on multithreaded programming in Java is available in
     * <a href="http://java.sun.com/Series/Tutorial/java/threads/multithreaded.html">
     * the Java tutorial on multi-threaded programming</a>.
     *
     * @param request the <code>ServletRequest</code> object that contains
     *            the client's request
     * @param response the <code>ServletResponse</code> object that contains
     *            the servlet's response
     * @throws ServletException if an exception occurs that interferes
     *                          with the servlet's normal operation
     * @throws IOException      if an input or output exception occurs
     */
    @Override
    public void service(ServletRequest request, ServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.println("<html>" +
                "<head></head>" +
                "<body>hello,world</body>");
    }

    /**
     * Returns information about the servlet, such
     * as author, version, and copyright.
     *
     * <p>The string that this method returns should
     * be plain text and not markup of any kind (such as HTML, XML,
     * etc.).
     *
     * @return a <code>String</code> containing servlet information
     */
    @Override
    public String getServletInfo() {
        return null;
    }

    /**
     * Called by the servlet container to indicate to a servlet that the
     * servlet is being taken out of service.  This method is
     * only called once all threads within the servlet's
     * <code>service</code> method have exited or after a timeout
     * period has passed. After the servlet container calls this
     * method, it will not call the <code>service</code> method again
     * on this servlet.
     *
     * <p>This method gives the servlet an opportunity
     * to clean up any resources that are being held (for example, memory,
     * file handles, threads) and make sure that any persistent state is
     * synchronized with the servlet's current state in memory.
     */
    @Override
    public void destroy() {

    }
}

```
### 2.2 ServletRequest
对于每一个HTTP请求，Servlet容器都会创建一个ServletRequest实例，封装了这个请求的信息，并将他传给Service方法
```
public String getParameter(String name);  //返回指定参数名的值
public int getContentLength()             //请求主体的字节数
public String getContentType()            //返回主体的MIME类型
public String getProtocol()               //返回HTTP请求的协议名称和版本
```
### 2.3 ServletResponse
表示一个Servlet响应，在调用Servlet的Service方法之前，Servlet容器首先创建一个ServletResponse，并将它作为第二个参数传给Service方法，ServletResponse隐藏了向浏览器发送响应的复杂过程。
首先应该调用setContentType方法，设置响应内容类型为"text/html",调用getWriter方法返回一个可以向客户端发送文本的PrintWriter，还有getOutputStream方法，可以发送二进制数据。
### 2.4 ServletConfig
Servlet容器初始化Servlet时会向init方法传入一个ServletConfig实例封装了@WebServlet或者xml文件传递给Servlet的配置信息，初始参数为key-vealue形式存储在HashMap中。
```
String getInitParameter(String name)  //获取初始参数值
Enumeration<String> getInitParameterNames()  //返回所有参数名称枚举类型
```
### 2.5 ServletContext
负责定义一系列和容器通讯的方法，例如获取MIMI类型，调度器请求，写日志。每个虚拟机的每个web应用只有唯一的context，对于分布式web应用来说每个虚拟机都会有一个上下文，所以他们不能用来存储全局信息，可以用外部数据库代替，ServletContext对象包含于ServletConfig当中。