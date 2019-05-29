[toc]
参考：
https://zhuanlan.zhihu.com/p/24293062?refer=dreawer
# 1. 引言
SpringMVC提供的RequestContextHolder它是持有上下文的Request容器，使用方法如下:
```
//两个方法在没有使用JSF的项目中是没有区别的
RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
                                    //RequestContextHolder.getRequestAttributes();
//从session里面获取对应的值
String str = (String) requestAttributes.getAttribute("name",RequestAttributes.SCOPE_SESSION);
//获取HttpServletRequest
HttpServletRequest request = ((ServletRequestAttributes)requestAttributes).getRequest();
//获取HttpServletResponse
HttpServletResponse response = ((ServletRequestAttributes)requestAttributes).getResponse();
```
# 2. 原理
## 2.1 request和response怎么和当前请求挂钩?
首先分析RequestContextHolder这个类,里面有两个ThreadLocal保存当前线程下的request
```
//得到存储进去的request
private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
new NamedThreadLocal<RequestAttributes>("Request attributes");
//可被子线程继承的request
private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
new NamedInheritableThreadLocal<RequestAttributes>("Request context");
```
再看`getRequestAttributes()`方法,相当于获取当前线程的threadLocals属性中key为requestAttributesHolder或inheritableRequestAttributesHolder对应的RequestAttributes,这样就保证了每一次获取到的Request是该请求的request.
## 2.2 request和response是怎么设置进去的
这个涉及到spring MVC的初始化原理，下次单独讲解。