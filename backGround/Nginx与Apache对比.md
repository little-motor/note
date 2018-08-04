[toc]
## 1. Nginx服务器
nginx是一个高性能的HTTP和反向代理服务器，同时也是一个 IMAP/POP3/SMTP 代理服务器。Nginx 以事件驱动的方式编写，所以有非常好的性能，同时也是一个非常高效的反向代理、负载平衡。
Nginx做为HTTP服务器，有以下几项基本特性：

- 处理静态文件，索引文件以及自动索引，打开文件描述符缓冲
- 无缓存的反向代理加速，简单的负载均衡和容错
- FastCGI，简单的负载均衡和容错
- 模块化的结构。包括 gzipping, byte ranges, chunked responses,以及 SSI-filter 等 filter。
- 支持 SSL 和 TLSSNI
### 1.1 Nginx架构
Nginx默认是多进程方式，也支持多线程方式。Nginx 在启动后，会有一个 master 进程和多个worker进程。master进程主要用来管理worker进程，包含：接收来自外界的信号，向各 worker 进程发送信号，监控 worker 进程的运行状态，当 worker 进程退出后(异常情况下)，会自动重新启动新的 worker 进程。多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求，只可能在一个worker进程中处理，一个worker进程，不可能处理其它进程的请求。
![构架](http://wiki.jikexueyuan.com/project/nginx/images/chapter-2-1.png)
## 2. Apache服务器
Apache HTTP Server（简称Apache）是Apache软件基金会的一个开放源代码的网页服务器软件，它快速、可靠并且可通过简单的API扩充，将Perl／Python等解释器编译到服务器中。 
### 2.1 特性
Apache支持许多特性，大部分通过编译的模块实现。这些特性从服务器端的编程语言支持到身份认证方案。一些通用的语言接口支持Perl，Python，Tcl，和PHP。流行的认证模块包括mod_access，mod_auth和mod_digest。其他的例子有SSL和TLS支持（mod_ssl），代理服务器（proxy）模块，很有用的URL重写（由mod_rewrite实现），定制日志文件（mod_log_config），以及过滤支持（mod_include和mod_ext_filter）。
## 3. Apache和Nginx的区别
### 3.1 Nginx相对Apache的优点

- 轻量级，比apache占用更少的内存及资源
- 静态处理，Nginx静态处理性能比Apache高3倍以上
- 抗并发，nginx处理请求是异步非阻塞的，而apache则是阻塞型的，在高并发下nginx能保持低资源低消耗高性能。

### 3.2 Apache相对Nginx的优点

- 模块超多，基本想到的都可以找到
- 超稳定

