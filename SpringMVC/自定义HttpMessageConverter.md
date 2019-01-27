[toc]
# 1. 引言
当一个请求到来时，在处理器执行的过程中，他首先从HTTP请求和上下文环境来得到参数，如果是简易的参数他会以简单的转换器进行转换，如果是HTTP请求体需要转换，就会调用HttpMessageConverter接口的实现类对请求体的信息进行转换。
#### 代码清单1-1
```

package org.springframework.http.converter;

/**
 * Strategy interface that specifies a converter that can convert from and to HTTP requests and responses.
 */
public interface HttpMessageConverter<T> {

	/**
	 * Indicates whether the given class can be read by this converter.
	 */
	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

	/**
	 * Indicates whether the given class can be written by this converter.
	 */
	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

	/**
	 * Return the list of {@link MediaType} objects supported by this converter.
	 */
	List<MediaType> getSupportedMediaTypes();

	/**
	 * Read an object of the given type from the given input message, and returns it.
         * inputMessage是读入的原始信息，它正是SpringMVC的消息转换器所作用的“请求消息”的内部抽象，消息转换器
         * 从“请求消息”中按照规则提取消息，转换为方法形参中声明的对象
	 */
	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	/**
	 * Write an given object to the given output message.
         * 同理outputMessage是“响应消息”的内部抽象，他讲返回的Java对象按照定义的方法转换为原始字符串。
	 */
	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}
```

![httpmessageconvert](https://github.com/little-motor/uml/raw/master/Spring/springmvc/httpmessageconverter.png)
<center>请求体转换过程如图所示</center>
# 2. 原理
HttpMessageConverter<T>是对http请求体的顶层封装接口，他下面有很多抽象类和实现类负责对应不同类型的请求体转换，比如常用的StringHttpMessageConverter、StringHttpMessageConverter等等。
**当controller的方法参数上加入@RequestBody注解时**，http请求体(body)首先转化为HttpInputMessage对象并且在未传递到对应方法的参数(@RequestBody标注)之前首先通过HttpMessageConverter对应的实现类转化实例并作为方法的参数。
**同理，当controller的方法上加入@ResponseBody时**，返回的Java对象会通过合适的HttpMessageConverter实现类转化为HttpOutputMessage类型最后返回相应报文，即http文本。
# 3.RequestResponseBodyMethodProcessor
通过实现HttpMessageConverter接口或者AbstractHttpMessageConverter抽象类可以自定义一个请求体转换器，但是背后调用这些请求体转换器是由RequestResponseBodyMethodProcessor类实现的，在官方文档中介绍原话是
```
Resolves method arguments annotated with @RequestBody and handles return values from methods annotated with @ResponseBody by reading and writing to the body of the request or response with an HttpMessageConverter.

An @RequestBody method argument is also validated if it is annotated with @javax.validation.Valid. In case of validation failure, MethodArgumentNotValidException is raised and results in an HTTP 400 response status code if DefaultHandlerExceptionResolver is configured.
```
大概意思就是这个类负责处理注解了@RequestBody**方法参数**和方法上注解了@ResponseBody的**返回值**。通过HttpMessageConverter的请求和返回对象，同时@RequestBody注解的参数同时能由@Valid注解进行验证。

























 在文件MyMvcConfig中配置configureMessageConverters：重载会覆盖掉Spring MVC默认注册的多个HttpMessageConverter。
1. 在文件MyMvcConfig中配置extendMessageConverters：仅添加一个自定义的HttpMessageConverter，不覆盖默认注册的HttpMessageConverter。 