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

		// Let subclasses do whatever initialization they like.
		initServletBean();
	}
```