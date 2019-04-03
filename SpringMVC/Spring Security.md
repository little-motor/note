[toc]
# 1. 引言
在Java web工程中，一般使用servlet过滤器(Filter)对请求进行拦截，然后在filter中通过自己的验证逻辑决定是否放行请求。Spring Security也是基于这个原理，在进入DispatcherServlet前可以对Spring MVC的请求进行拦截，然后验证决定是否放行。
当启用Spring Security，Spring IOC容器会创建一个名为springSecurityFilterChain的bean，他的类型为FilterChainProxy并实现了Filter接口，他是一个特殊的拦截器。另一方面，在Spring Security操作的过程中他会提供servlet过滤器DelegatingFilterProxy，这个对象上存在一个列表(List)，列表上存在用户验证的拦截器、跨站点请求伪造等拦截器，自定义拦截器等FilterChainProxy对象，以此提供多种拦截功能。
# 2. 使用WebSecurityConfigurerAdapter自定义
为了给FilterChainProxy对象加入自定义的初始化可以通过SecurityConfigurer接口配置SpringSecurity，对于web工程还提供了专门的接口WebSecurityConfigurer，并在此接口的基础上提供了一个抽象类WebSecurityConfigurerAdapter，开发者通过继承它就能得到Spring Security默认安全功能，也可以覆盖他的方法来自定义安全拦截方案。
```
/**
     * 配置用户签名服务，主要是user-details机制，还可以给用户赋予角色
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    }

    /**
     * 用来配置filter链
     * @param web
     * @throws Exception
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
    }

    /**
     * 配置拦截保护的请求，比如什么请求放行，什么请求需要验证，指定用户和角色与对应的url访问权限
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
    }
```
## 2.1 自定义用户认证信息
AuthenticationManagerBuilder可以通过内存签名，数据库签名和自定义签名三种方式。内存签名方便调试和快速开发，此处不再涉及。
同时需要注意的是Spring5的Security中都要求使用密码编辑器，否则会发生异常，所以需要PasswordEncoder(在spring security包中定义)接口实现类
常用的方法如下：

项目类型 | 描 述
---------|----------
 accountExpired(boolean) | 设置账号是否过期
 accountLocked(boolean) | 是否锁定账号
 credentialsExpired(boolean) | 定义证书是否过期
 disabled(boolean) | 是否禁用用户
 username(String) | 定义用户名
 authorities(GrantedAuthority...) | 赋予一个或多个权限
 password | 定义密码
 roles(String...) | 赋予角色，会自动加入前缀"ROLE_"