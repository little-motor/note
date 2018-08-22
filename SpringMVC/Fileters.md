[toc]
## 1. 引言
Filter是拦截Request请求的对象，在用户的请求访问资源前处理ServletRequest以及ServletResponse，他可用于日志记录、加解密、Session检查、图像文件保护等。
## 2. Filter API
Filter相关接口包括Filter、FilterConfig、FilterChain。
### 2.1 Filter接口
包含三个生命周期init、doFilter、destroy，servlet容器初始化Filter时，会触发Filter的init方法，FilterConfig实例由Servlet容器传入init方法中。
```
void init(FilterConfig filterConfig)
```
Servlet容器每次处理Filter相关资源时，会调用相关Filter实例的doFilter方法
```
void doFilter(ServletRequest request, Servlet Response, FilterChanin filterChain)
```
在Filter的doFilter的实现中，最后一行需要调用FilterChain中的doChain方法
```
filterChain.doFilter(request, response)
```
一个资源可能被多个Filter关联到（Filter链条），这时Filter.doFilter()方法将触发Filter链条中下一个Filter。**只有在Filter链条中的最后一个Filter里调用FilterChain.doFilter()才会触发处理资源方法，否则Request请求终止。**
Filter接口最后一个方法是destroy，在servlet容器要销毁时触发
```
void destroy();
```
## 3. Filter配置
Filter配置需要如下步骤：
1. 确认哪些资源需要使用这个Filter拦截器
2. 配置Filter的初始化参数值
3. 给Filter取一个名称，方便识别

FilterConfig接口允许通过他的getServletContext方法访问ServletContext
```
ServletContext getServletContext();                             //返回ServletContext
java.lang.String getFilterName();                               //返回Filter名字
java.util.Enumeration<java.lang.String> getInitParameterNames() //返回初始化参数名
java.lang.String getInitParameter(java.lang.String parameterName)//返回参数
```
配置方式有两种，一种是@WebFilter注解方法，另一种是通过XML注册方式，以@WebFilter注解为例，下面是可选参数

属性 | 描述| 
---------|----------
asyncSupported | Filter是否支持异步操作
 description | Filter描述
 dispatcherTypes | Filter生效范围
 displayName|Filter显示名
 filterName|Filter名称
 initParams|Filter初始化参数
 urlPatterns|Filter生效的URL路径
 value|Filter生效的URL路径
```
@WebFilter(filterName = "Security Filter", urlPatterns = "/*",
  initParams = {
    @WebInitParam(name = "A", value = "1")
    @WebInitParam(name = "B", value = "2")
  })
```