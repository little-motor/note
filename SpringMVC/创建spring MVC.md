[toc]
# 1. 引言
Spring MVC核心结构图如下所示
![DispatcherServlet结构图](https://raw.githubusercontent.com/little-motor/uml/master/Spring/springmvc/DispatcherServlet.png)
其中aware意为感知，实现XXXAware接口就会通过接口的唯一方法setXXX得到ApplicationContext或Environment等。EnvironmentCapable有惟一方法getEnvironment()。同时需要注意的是ApplicationContext为应用上下文，Environment封装了servletContext、servletConfig、JndiProperty、系统环境变量和系统属性等。
# 2. HttpServletBean
HttpServlet的简单扩展，它将配置参数(web.xml中servlet标记中的init-param条目)视为bean属性。
对于任何类型的servlet都是一个方便的超类。他实现了EnvironmentAware和EnvironmentCapable可以获取environment属性。这里主要关注其init()方法。
```
/**
	 * Map config parameters onto bean properties of this servlet, and
	 * invoke subclass initialization.
	 * @throws ServletException if bean properties are invalid (or required
	 * properties are missing), or if subclass initialization fails.
	 */
	@Override
	public final void init() throws ServletException {

		// Set bean properties from init parameters.
		PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
		if (!pvs.isEmpty()) {
			try {
				BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
				ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
				bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
				initBeanWrapper(bw);
				bw.setPropertyValues(pvs, true);
			}
			catch (BeansException ex) {
				if (logger.isErrorEnabled()) {
					logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
				}
				throw ex;
			}
		}

		// 由子类实现初始化
		initServletBean();
	}
```
可以看到子类通过重写initServletBean()方法来进一步初始化。
# 3. FrameworkServlet
```
/**
	 * Overridden method of {@link HttpServletBean}, invoked after any bean properties
	 * have been set. Creates this servlet's WebApplicationContext.
	 */
	@Override
	protected final void initServletBean() throws ServletException {
		getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
		if (this.logger.isInfoEnabled()) {
			this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
		}
		long startTime = System.currentTimeMillis();

		try {
			//初始化WebApplicationContext
			this.webApplicationContext = initWebApplicationContext();
			//空方法，子类可以重写
			initFrameworkServlet();
		}
		catch (ServletException ex) {
			...
```
在FrameworkServlet中重写了initServletBean()方法，其主要做了两件事
```
//初始化webApplicationContext
this.webApplicationContext = initWebApplicationContext();
//初始化FrameworkServlet，模板方法
initFrameworkServlet();
```
让我们来了解一下initWebApplicationContext方法
```
	/**
	 * Initialize and publish the WebApplicationContext for this servlet.
	 * <p>Delegates to {@link #createWebApplicationContext} for actual creation
	 * of the context. Can be overridden in subclasses.
	 */
	protected WebApplicationContext initWebApplicationContext() {
		//获取rootContext
		WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
		WebApplicationContext wac = null;
		//如果已经通过构造方法设置了WebApplicationContext
		if (this.webApplicationContext != null) {
			// A context instance was injected at construction time -> use it
			wac = this.webApplicationContext;
			if (wac instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
				if (!cwac.isActive()) {
					// The context has not yet been refreshed -> provide services such as
					// setting the parent context, setting the application context id, etc
					if (cwac.getParent() == null) {
						// The context instance was injected without an explicit parent -> set
						// the root application context (if any; may be null) as the parent
						cwac.setParent(rootContext);
					}
					//configureAndRefreshWebApplicationContext方法用于封装ApplicationContext数据并且初始化所有相关Bean对象。
					//它会从web.xml中读取名为contextConfigLocation的配置，这就是spring xml数据源设置，然后放到ApplicationContext中，
					//最后调用传说中的refresh方法执行所有Java对象的创建
					configureAndRefreshWebApplicationContext(cwac);
				}
			}
		}
		if (wac == null) {
			//从servlet context中获取webApplicationContext
			// No context instance was injected at construction time -> see if one
			// has been registered in the servlet context. If one exists, it is assumed
			// that the parent context (if any) has already been set and that the
			// user has performed any initialization such as setting the context id
			wac = findWebApplicationContext();
		}
		if (wac == null) {
			//如果从servlet context中找不到就创建一个新的
			// No context instance is defined for this servlet -> create a local one
			wac = createWebApplicationContext(rootContext);
		}

		if (!this.refreshEventReceived) {
			//如果没有refresh，在这里刷新
			// Either the context is not a ConfigurableApplicationContext with refresh
			// support or the context injected at construction time had already been
			// refreshed -> trigger initial onRefresh manually here.
			onRefresh(wac);
		}

		if (this.publishContext) {
			//将ApplicationContext保存到ServletContext中
			// Publish the context as a servlet context attribute.
			String attrName = getServletContextAttributeName();
			getServletContext().setAttribute(attrName, wac);
			if (this.logger.isDebugEnabled()) {
				this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
						"' as ServletContext attribute with name [" + attrName + "]");
			}
		}

		return wac;
	}
```
initWebApplicationContext方法做了三件事：
1. 获取spring的根容器rootContext
2. 设置webApplicationContext并根据情况调用onRefresh方法
3. 将webApplicationContext设置到ServletContext中

整个FrameworkServlet中重要的事情就是将创建出来的WebApplicationContext设置到ServletContext中，方便获取。