[toc]
## 1. 引言
由于HTTP协议是无状态的，所以用cookie机制在客户端保持状态，而session机制在服务器端保持状态。同时我们也看到，由于采用服务器端保持状态的方案在客户端也需要保存一个标识，所以session机制可能需要借助于cookie机制来达到保存标识的目的，但实际上它还有其他选择。 
## 2. cookie机制
cookie的内容主要包括：名字，值，过期时间，路径和域。正常的cookie分发时通过扩展HTTP协议来实现的，服务器通过在HTTP的响应头中加上一行特殊的指示以提示浏览器按照指示生成相应的cookie，另外纯粹的客户端脚本如JavaScript或者VBScript也可以生成cookie。 
而cookie的使用是由浏览器按照一定的原则在后台自动发送给服务器的。浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于等于将要请求的资源所在的位置，则把该cookie附在请求资源的HTTP请求头上发送给服务器。
## 3. session机制
Session 对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的 Web 页之间跳转时，存储在 Session 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。 
当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了session标识，称为session id，如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。 
保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发送给服务器。
### 3.1 cookie被禁用的解决方法
1. URL重写
把session id直接附加在URL路径的后面，附加方式也有两种，一种是作为URL路径的附加信息，表现形式为http://...../xxx;jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764 
另一种是作为查询字符串附加在URL后面，有利于把session id的信息和正常程序参数区分开来，表现形式为http://...../xxx?jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764为了在整个交互过程中始终保持状态，就必须在每个客户端可能请求的路径后面都包含这个session id。
2. 表单隐藏字段
```
<form name="testform" action="/xxx"> 
    <input type="hidden" name="jsessionid" value="ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764"> 
    <input type="text"> 
    </form> 
```