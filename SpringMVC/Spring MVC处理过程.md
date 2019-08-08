[toc]
# 1. 引言
在这里简单简单分析一个请求在Spring MVC中的处理过程
# 2. HttpServletBean
HttpServletBean主要参与了创建工作，并没有涉及请求的处理
# 3. FrameworkServlet
首先从Servlet接口的service方法开始，在FrameworkServlet中重写了service、doGet、doPost、doPut、doDelete、doOptions、doTrace方法等，所有这些方法都交给了processRequest方法进行统一处理。以doGet的代码为例
```
@Override
	protected void service(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
		if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
			processRequest(request, response);
		}
		else {
			super.service(request, response);
		}
	}

	@Override
	protected final void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		processRequest(request, response);
	}
```
这样做的好处是开发者如果对DispatcherServlet有特殊对定制需求可以覆盖doGet、doPost等方法，但是一般不必要修改内部。让我们来看一下processReqeust方法，他是FrameworkServlet类在处理请求过程中的核心方法
```
/**
	 * Process this request, publishing an event regardless of the outcome.
	 * <p>The actual event handling is performed by the abstract
	 * {@link #doService} template method.
	 */
	protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		long startTime = System.currentTimeMillis();
		Throwable failureCause = null;
        //通过ContextHolder获得请求参数
		LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
		LocaleContext localeContext = buildLocaleContext(request);

		RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
		asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

		initContextHolders(request, localeContext, requestAttributes);

		try {
            //真正的处理交给doService模版方法实现，在FrameworkServlet中是空实现，由DispatcherServlet重写
			doService(request, response);
		}
		catch (ServletException | IOException ex) {
			failureCause = ex;
			throw ex;
		}
		catch (Throwable ex) {
			failureCause = ex;
			throw new NestedServletException("Request processing failed", ex);
		}

		finally {
			resetContextHolders(request, previousLocaleContext, previousAttributes);
			if (requestAttributes != null) {
				requestAttributes.requestCompleted();
			}
			logResult(request, response, failureCause, asyncManager);
            //事件发布
			publishRequestHandledEvent(request, response, startTime, failureCause);
		}
	}
```
processRequest自己主要做了两件事：

1. 对LocaleContext和RequestAttributes的设置和恢复
2. 处理完后发布ServletRequestHandledEvent
## 3.1 LocaleContextHolder和RequestContextHolder
LocaleContext存放着本地化信息，RequestAttributes是Spring的一个接口，通过他的get/set/removeAttribute根据scope判断操作是request还是session。

这里顺便提一下LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext()其内部如下
```
	/**
	 * Return the LocaleContext associated with the current thread, if any.
	 * @return the current LocaleContext, or {@code null} if none
	 */
	@Nullable
	public static LocaleContext getLocaleContext() {
		LocaleContext localeContext = localeContextHolder.get();
		if (localeContext == null) {
			localeContext = inheritableLocaleContextHolder.get();
		}
		return localeContext;
	}
```
这里用到了ThreadLocal，通过他每个线程可以独立保存自己的内容，其核心原理是Thread内部有一个属性为ThreadLocal.ThreadLocalMap threadLocals，当ThreadLocal在get/set时首先拿到线程当threadLocals，然后以ThreadLocal实例作为key，所要保存的值为value，put进去，这样就将具体当值保存在线程自身上面。
LocaleContextHolder和RequestContextHolder可以方便在service层没有servlet时直接获得了Locale和RequestAttributes，顺便打个断点看了一下他们是在什么时候把这几个属性注入的，发现是在RequestContextFilter中的doFilterInternal方法里的initContextHolders(request, attributes)方法里
```
	@Override
	protected void doFilterInternal(
			HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		ServletRequestAttributes attributes = new ServletRequestAttributes(request, response);
        //注入Locale和RequestAttributes
		initContextHolders(request, attributes);

		try {
			filterChain.doFilter(request, response);
		}
		finally {
			resetContextHolders();
			if (logger.isTraceEnabled()) {
				logger.trace("Cleared thread-bound request context: " + request);
			}
			attributes.requestCompleted();
		}
	}

	private void initContextHolders(HttpServletRequest request, ServletRequestAttributes requestAttributes) {
		LocaleContextHolder.setLocale(request.getLocale(), this.threadContextInheritable);
		RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
		if (logger.isTraceEnabled()) {
			logger.trace("Bound request context to thread: " + request);
		}
	}
```
## 3.2 事件发布
publishRequestHandledEvent发布一个ServletRequestHandledEvent消息，可以通过监听这个事件记录一些事情，通过实现接口ApplicationListenner<ServletRequestHandledEvent>
# 4. DispatcherServlet
DispatcherServlet