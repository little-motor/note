[toc]
# 1. 引言
以下是RestTemplate官方解释：
用于同步客户端HTTP访问的Spring的中心类。它简化了与HTTP服务器的通信，并加强了RESTful原则。它处理HTTP连接，留下应用程序代码来提供url(带有可能的模板变量)并提取结果。
注意:默认情况下，RestTemplate依赖于标准JDK工具来建立HTTP连接。您可以通过setRequestFactory属性切换到使用不同的HTTP库，比如Apache HttpComponents、Netty和OkHttp。
该模板的主要入口点是以六个主要HTTP方法命名的方法，此外，exchange和execute方法是上述方法的通用版本，可以用于支持其他的、不太频繁的组合(例如HTTP PATCH、HTTP PUT with response body等)。但是请注意，使用的底层HTTP库也必须支持所需的组合。
对于每个HTTP方法，都有三个变体:两个接受URI模板字符串和URI变量(数组或映射)，而第三个接受URI。注意，对于URI模板，假定编码是必要的，例如，restTemplate.getForObject(“http://example.com/hotel list”)变成了“http://example.com/hotel%20list”。这也意味着，如果URI模板或URI变量已经编码，就会发生双重编码，例如http://example.com/hotel%20list变成http://example.com/hotel%2520list)。为了避免这种情况，可以使用URI方法变体来提供(或重用)以前编码的URI。要准备这样一个完全控制编码的URI，请考虑使用org.springframework.web.util.UriComponentsBuilder。
在内部，模板使用HttpMessageConverter实例将HTTP消息转换为pojo和从pojo转换为HTTP消息。默认情况下，主要mime类型的转换器是注册的，但是您也可以通过setMessageConverters注册其他转换器。
该模板使用org.springframework.http.client.SimpleClientHttpRequestFactory和DefaultResponseErrorHandler分别作为创建HTTP连接或处理HTTP错误的默认策略。这些默认值可以分别通过setRequestFactory和setErrorHandler重写。
# 2. 使用RestTemplate请求后端
在这里需要记住的是RestTemplated的底层是通过类HttpURLConnection实现的
## 2.1 资源请求getForObject
在请求过程中传递的是JSON数据，RestTemplate内部会将其转换为Java对象。
```
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {}

public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException {}

public <T> T getForObject(URI url, Class<T> responseType) throws RestClientException {}
```
url为请求的路径，responseType为返回类型，uriVariables为边长请求参数，以下面两个示例做说明。
当请求参数较少时
```
User user = restTemplate.getForObject("http://localhsot:8080/user/{id}", User.class, id)
```
当参数较多时，可以使用map保存参数，需要注意的是map的key需要和url里面的参数名相同。
```
Map<String, String> params = new HashMap<>();
params.put("name","a");
params.put("sex","m");
...
User user = restTemplate.getForObject("http://localhsot:8080/user/{name}/{set}/...", User.class, params)
```

## 2.2 资源请求getForEntity
getForEntity返回一个ResponseEntity<T>实例，他的请求参数也同2.1中的getForObject方法一样有三种形式
```
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {}

public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException {}

public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType) throws RestClientException {}
```
其中常用的一种形式如下：
```
Map<String, String> params = new HashMap<>();
params.put("name","a");
params.put("sex","m");
...
User user = restTemplate.getForObject("http://localhsot:8080/user/{name}/{set}/...", User.class, params)
```
ResponseEntity<T>是HttpEntity<T>的子类，HttpEntity里面包含了了body和httpHeaders，而其子类ResponseEntity则多了返回状态。
## 2.3 Post请求
同上面一样，Post方法有postForObject和postForEntity，其方法如下：
```
public <T> T postForObject(String url, Object request, Class<T> responseType, Object... uriVariables)
	throws RestClientException {}

public <T> T postForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)
	throws RestClientException {}

public <T> T postForObject(URI url, Object request, Class<T> responseType) throws RestClientException {
```
我们通过postForEntity方法返回ResponseEntity实例之后可以通过查看他里面的状态属性判断请求的成功与否，示例代码如下：
```
ResponseEntity<User> userEntity = restTemplate.postForEntity("http://localhost/user", request, User.class);
//查看返回的状态码
int status = userEntity.getStatusCodeValue();
```
# 3. exchange方法
exchange和execute方法是上述方法的通用版本，可以用于支持其他的、不太频繁的组合(例如HTTP PATCH、HTTP PUT with response body等)，他们的使用非常灵活，但是同样的他的易读性不如前面提到的方法，所以这里面单独提出来讲解，通常情况下推荐使用*forEntity()方法。exchange有8个方法：
```
public <T> ResponseEntity<T> exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType, Object... uriVariables) throws RestClientException {}

public <T> ResponseEntity<T> exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException {}

public <T> ResponseEntity<T> exchange(URI url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType) throws RestClientException {}

public <T> ResponseEntity<T> exchange(String url, HttpMethod method, HttpEntity<?> requestEntity,
ParameterizedTypeReference<T> responseType, Object... uriVariables) throws RestClientException {}

public <T> ResponseEntity<T> exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType, Map<String, ?> uriVariables) throws RestClientException {}

public <T> ResponseEntity<T> exchange(URI url, HttpMethod method, HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType) throws RestClientException {}

public <T> ResponseEntity<T> exchange(RequestEntity<?> requestEntity, Class<T> responseType) throws RestClientException {}

public <T> ResponseEntity<T> exchange(RequestEntity<?> requestEntity, ParameterizedTypeReference<T> responseType) throws RestClientException {}
```
可以发现参数还是比较相似的，就不一一注明了，excute有三个方法：
```
public <T> T execute(String url, HttpMethod method, RequestCallback requestCallback, ResponseExtractor<T> responseExtractor, Object... uriVariables) throws RestClientException {}

public <T> T execute(String url, HttpMethod method, RequestCallback requestCallback, ResponseExtractor<T> responseExtractor, Map<String, ?> uriVariables) throws RestClientException {}

public <T> T execute(URI url, HttpMethod method, RequestCallback requestCallback, ResponseExtractor<T> responseExtractor) throws RestClientException {}
```
# 4. 小结
先简单的总结到这里，下次有机会研究一下背后的底层实现。
