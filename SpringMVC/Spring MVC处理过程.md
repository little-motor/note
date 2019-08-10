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
## 4.1 doService
DispatcherServlet是Spring MVC最核心的类，整个处理过程的顶层设计都在这里。其处理方法的入口在doService在其内部又交给了doDispatcher进行具体的处理，doService方法大概过程是首先判断是否包含请求，如果是则对request的Attribut做个快照并设置一些属性，等doDispatch处理完之后进行还原。代码如下：
```
/**
	 * Exposes the DispatcherServlet-specific request attributes and delegates to {@link #doDispatch}
	 * for the actual dispatching.
	 */
	@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			attributesSnapshot = new HashMap<>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

        //实现redirect时的数据保存
		if (this.flashMapManager != null) {
			FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
			if (inputFlashMap != null) {
				request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
			}
			request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
			request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
		}

		try {
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}
```
其中的flashmap方便direct时候的保存一些数据而不用暴露在url中，我们只需要在redirect之前将需要传递的参数写入OUTPUT_FLASH_MAP_ATTRIBUTE
```
((FlashMap)((ServletReqeustAttributes)(RequestContextHolder.getRequestAttributes())
    .getAttribute(DispatcherServlet.OUTPUT_FLASH_MAP_ATTRIBUTE))).put("name","littlemotor");
```
之后就会被设置到INPUT_FLASH_MAP_ATTRIBUTE属性里，然后再放到model里，当然Spring提供了更方便的方法，在控制器中通过注入RedirectAttributes实例，通过addAttribute(key,value),addFlashAttribute(key,value)，可以达到同样的效果，第一种是在redirect时参数拼接到url中，第二种是在redirect时参数保存到flashMap中，例如下面的方式：
```
@PostMapping("submit")
public String submit(RedirectAttribute attr){
    ((FlashMap)((ServletReqeustAttributes)(RequestContextHolder.getRequestAttributes())
    .getAttribute(DispatcherServlet.OUTPUT_FLASH_MAP_ATTRIBUTE))).put("id","xxx");
    //与上面的等效
    attr.addFlashAttribute("id","xxx");
    //zh-cn会拼接到uri上面
    attr.add.addAttribute("local","zh-cn");
    return "redirect:uri";
}
```
inputFlashMap用于保存上次请求中转发过来的属性，outputFlashMap用于保存本次请求中需要转发的属性，FlashMapManager用于管理他们。
## 4.2 doDispatch
doDispatch方法也比较简介，其最核心的代码就4句
1. 根据request找到Handler
2. 根据Handler找到对应的HandlerAdapter
3. 用HandlerAdapter处理Handler
4. 调用processDispatchResult方法处理上面每一步得到的结果
```
/**
	 * Process the actual dispatching to the handler.
	 * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
	 * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
	 * to find the first that supports the handler class.
	 * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
	 * themselves to decide which methods are acceptable.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @throws Exception in case of any kind of processing failure
	 */
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        //外层捕获处理渲染页面异常
		try {
			ModelAndView mv = null;
			Exception dispatchException = null;
            //内层捕获处理请求处理异常
			try {
                //检查是不是上传请求
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
                //根据request找到Handler
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
                //根据Handler找到对应的HandlerAdapter
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
                //用HandlerAdapter处理Handler
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            //处理上面处理之后的结果
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```
首先需要理解一下三个概念：
1. Handler：每一个被@RequestMapping标注的方法就是一个Handler，用于处理实际请求
2. HandlerMapping：用来查找Handler
3. HandlerAdapter：他是一个适配器，因为Handler的形式是灵活的，可以是类也可以是方法，而servlet的处理方式是固定的，都是以reqeust和response为参数的方法

doDispatcher大体可以分为两部分：处理请求和渲染页面，开头定义了几个变量：
- HttpServletRequest processedRequest，实际处理时所用的request，如果不是上传请求则直接使用接收到的request，否则封装为上传类型的MultipartHttpServletRequest
- HandlerExecutionChain mappedHandler：处理请求的处理器链（包含处理器和对应的Interceptor）
- boolean multipartRequestParsed: 是不是上传请求的标志
- ModelAndView mv：封装Model和View的容器
- Excetion dispatchException:处理请求过程中抛出的异常，并不包含渲染过程中抛出的异常
### 4.2.1 getHandler
getHandler方法获取Handler处理器链，其中使用到了HandlerMapping返回值为HandlerExecutionChain类型，其中有与当前request相匹配的HandlerInterceptor和Handler，执行时依次执行Interceptor的PreHandler，然后执行Handler，返回时执行Interceptor的postHandler方法。
```
	/**
	 * Return the HandlerExecutionChain for this request.
	 * <p>Tries all handler mappings in order.
	 * @param request current HTTP request
	 * @return the HandlerExecutionChain, or {@code null} if no handler could be found
	 */
	@Nullable
	protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```
### 4.2.2 getHandlerAdapter
通过HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());获取HandlerAdapter，从类型名就可以知道这是一个适配器模式，mappedHandler.getHandler()返回的是object类型，因为handler的类型是多样的，在下面的getHandlerAdapter方法中通过遍历handlerAdapters，找到适配的atapter并返回。
```
/**
	 * Return the HandlerAdapter for this handler object.
	 * @param handler the handler object to find an adapter for
	 * @throws ServletException if no HandlerAdapter can be found for the handler. This is a fatal error.
	 */
	protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
```
之后分别进行preHandle方法，HandlerAdapter使用Handler处理请求并得到modelAndView，如果需要异步直接返回，否则执行postHandle方法，这样内层的try块就执行完成了。
### 4.2.3 processDispatchResult
processDispatchResult处理前面返回的结果，包括处理异常，渲染页面，触发Interceptor的afterCompletion方法三部分内容。
```
/**
	 * Handle the result of handler selection and handler invocation, which is
	 * either a ModelAndView or an Exception to be resolved to a ModelAndView.
	 */
	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		boolean errorView = false;

		//如果请求过程中有异常则处理异常
		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// 渲染页面
		if (mv != null && !mv.wasCleared()) {
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}

		//如果启动了异步处理则返回
		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}

		//发出请求处理完成的通知，触发Interceptor的afterCompletion
		if (mappedHandler != null) {
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
	}
```
### 4.2.4 异常捕获
- 内层捕获的是对请求进行处理的过程中抛出的异常，即被捕获并传递给dispatcherException变量的异常，然后由processDispatchResult方法进行处理。
- 外层主要是处理渲染页面时抛出的异常，主要处理processDispatchResult方法抛出的异常。
### 4.2.5 finally
在最后的finally中判断是否请求启动了异步处理，如果启动了则调用相应的异步处理拦截器，否则如果是上传请求则删除上传请求过程中产生的临时资源。
# 5. 小结
![doDispatch流程](https://raw.githubusercontent.com/little-motor/uml/master/Spring/springmvc/doDispatch%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)

整个过程大致如下：
1. HttpServletBean没有参与实际请求处理
2. FrameworkServlet将不同的请求合并到processRequest方法统一处理，processReqeust方法做了三件事情：
   - 调用doService模板方法处理请求
   - 将当前请求的LocaleContext和ServletReqeustAttribes在处理请求前设置到LocaleContextHolder和ReqeustContextHolder，并在请求处理完成后恢复
   - 请求处理完后发布ServletRequestEvent消息
3. DispatcherServlet：doService设置了一些属性并将请求交给doDispatch方法具体处理。