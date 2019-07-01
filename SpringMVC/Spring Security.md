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
### 2.1.1 使用数据库定义用户认证服务
```
public class WebSecurityConfigurerAdapterImpl extends WebSecurityConfigurerAdapter{

    @Autowired
    private DataSource dataSource = null;

    //sql语句

    @Override
    protected void configurer(AuthenticationManagerBuilder auth) throws Exception{
        //密码编码器
        PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        auth.jdbcAuthentication()
            //密码编码器
            .passwordEncoder(passwordEncoder)
            //数据源
            .dataSource(dataSource)
            //查询用户，自动判断密码是否一致
            .usersByUsernameQuery(pwdQuery)
            //赋予权限
            .authoritiesByUsernameQuery(roleQuery);
    }
}
```
### 2.1.2 使用自定义用户认证服务
Spring security提供了一个UserDetailsService接口，通过它可以获取用户信息，这个接口只有一个loadUserByUsername方法需要实现。
```
@Service
public class UserDetailsServiceImpl implements UserDetailsService{
    //注入接口服务
    @Autowired
    private UserRoleService userRoleService = null;

    @Override
    @transactional
    public UserDetails loadByUsername(String userName){
        //获取数据库用户信息
        //转换为UserDetails对象返回
        ...
    }
}
```
之后需要注册这个UserDetailsServiceImpl
```
public class WebSecurityConfigurerAdapterImpl extends WebSecurityConfigurerAdapter{

    @Autowired
    private UserDetailsService userDetailsService = null;

    protected void configurer(AuthenticationManagerBuilder auth) throws Exception{
        //密码编码器
        PasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        //设置用户密码服务和密码编辑器
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder);
    }
}
```
## 2.2 限制请求
WebSecurityConfigurerAdapter的configurer(HttpSecurity http)方法能够实现对于不同角色（用户）赋予不同权限的功能、限定请求范围以及认证方式等。HttpSecurity允许使用Ant风格或者正则式的路径限定安全请求
```
/**
* 使用ant风格配置限定模板
*/
@Override
protected void configure(HttpSecurity http) throws Exception{
    http.
        //限定"/user/welcome"和"/user/details"请求赋予角色ROLE_USER或者ROLE_ADMIN
        .antMathers("/user/welcome","/user/details").hasAnyRole("USER","ADMIN")
        //限定"/admin/"下所有请求权限赋予角色ROLE_ADMIN
        .antMathers("/admin/**").hasAuthority("ROLE_ADMIN")
        //其他路径允许签名后访问
        .anyRequest().permitAll()

        //允许匿名访问
        .and.anonymous()

        //采用默认登陆页面和认证方式
        .and().formLogin()
        .and().httpBasic();
}
```
这里需要注意的是把具体的配置放在前面，把不具体的放在后面，spring security会才有先配置的优先级高的原则。
hasAnyRole方法默认加入前缀"ROLE"，而hasAuthority方法则不会，他们都表示对应都路径只有用户分配了对应的角色才能访问。
关于权限常用的方法如下：

方法 | 含义
---------|----------
access(String) | 参数为Spring表达式，如果返回true则允许访问
anonymous() | 允许匿名访问
authorizeRequests() | 限定通过签名的请求
anyRequest() | 限定任意的请求
hasAnyRole(String...) | 将访问权限赋予多个角色（角色会自动加入前缀"ROLE_"）
hasRole(String) | 将访问权限赋予一个角色（角色会自动加入前缀"ROLE_"）
permitAll() | 无条件允许访问
and() | 连接词，并取消之前限定前提规则
not() | 对其他方法的访问采取求反
fullyAuthenticated() | 如果是完整验证(并非Remember me)，则允许访问
denyAll() | 无条件不允许任何访问
hasIpAddress(String) | 如果是给定的IP地址则允许访问
rememberme() | 用户通过Remember me功能验证就允许访问
hasAuthority(String) | 如果是给定用户就允许访问(不自动加入前缀"ROLE_")
hasAnyAuthority(String...) | 如果是给定角色中的任意一个就允许访问（不自动加入前缀"ROLE_")
## 2.3 自定义登陆页面
可以通过覆盖WebSecurityConfigurerAdapter的configure(HttpSecurity http)方法让登陆页面指向对应的请求路径和启用“记住我”功能。
```
@Override
protected void configure(HttpSecurity http) throws Exception{
    http.
        //限定"/admin/"下所有请求权限赋予角色ROLE_ADMIN
        .antMathers("/admin/**").hasAuthority("ROLE_ADMIN")
        //启用remember me功能
        .and().rememberMe().tokenValiditySeconds(86400).key("remember-me-key")
        //通过签名后可以访问任何请求
        .and().authorizeRequests().antMatcher("/**").permitAll()
        //设置登陆页面和默认的跳转路径
        .and().formLogin().loginPage("/login/page")
            .defaultSuccessUrl("/welcome");
}
```
# 3. 防止夸站点请求伪造(Cross-Site Request Forgery,CSRF)
csrf的工作原理，首先浏览器请求安全的网站，在登陆后浏览器就记录一些信息以cookie的形式保存，然后在不关闭浏览器的情况下，用户可能访问一个危险网站，危险网站通过获取cookie信息来伪造用户的请求，进而请求安全网站。
CsrfFilter过滤器用来防止CSRF攻击，可以通过下面方式关闭
```
//但是不建议这么做
http.csrf().disable().authorizeRequests()...
```
需要注意的是，match的method为除了"GET", "HEAD", "TRACE", "OPTIONS"以外的所有方法。通过源代码可知其验证方式大概是首次访问页面时首先生成并在服务器保存csrfToken，第二次如果是post，那么他会验证这个一起传过来的csrfToken与原来的是否一致。
```
//CsrfFilter中的核心方法

@Override
protected void doFilterInternal(HttpServletRequest request,
        HttpServletResponse response, FilterChain filterChain)
                throws ServletException, IOException {
    request.setAttribute(HttpServletResponse.class.getName(), response);
    //根据request从session载入csrfToken
    CsrfToken csrfToken = this.tokenRepository.loadToken(request);
    //判断csrfToken是否为空
    final boolean missingToken = csrfToken == null;
    //如果csrfToken为空那么生成新的csrfToken并保存
    if (missingToken) {
        csrfToken = this.tokenRepository.generateToken(request);
        this.tokenRepository.saveToken(csrfToken, request, response);
    }
    //获取token的方式有多种可以通过request的getAttribute也可以通过session
    //他们的key分别为"org.springframework.security.web.csrf.CsrfToken"、"_csrf"和"org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository.CSRF_TOKEN"
    request.setAttribute(CsrfToken.class.getName(), csrfToken);
    request.setAttribute(csrfToken.getParameterName(), csrfToken);
    //判断http方法是否为"GET", "HEAD", "TRACE", "OPTIONS"，如果是就不检查
    if (!this.requireCsrfProtectionMatcher.matches(request)) {
        filterChain.doFilter(request, response);
        return;
    }
    //获取request里面携带的csrfToken，首先检查header，之后检查body
    String actualToken = request.getHeader(csrfToken.getHeaderName());
    if (actualToken == null) {
        actualToken = request.getParameter(csrfToken.getParameterName());
    }
    //根据检验结果返回
    if (!csrfToken.getToken().equals(actualToken)) {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Invalid CSRF token found for "
                    + UrlUtils.buildFullRequestUrl(request));
        }
        if (missingToken) {
            this.accessDeniedHandler.handle(request, response,
                    new MissingCsrfTokenException(actualToken));
        }
        else {
            this.accessDeniedHandler.handle(request, response,
                    new InvalidCsrfTokenException(csrfToken, actualToken));
        }
        return;
    }

    filterChain.doFilter(request, response);
}
```
