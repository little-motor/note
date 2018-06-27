# 新建Spring MVC项目并使用Java代码配置容器
### ——使用WebApplicationInitializer实现
## 1.引言：
在Servlet3.0+环境中，容器的配置支持多种方式，可以用纯Java代码方式配置（推荐），也可以用传统的XML文件配置，甚至可以将两个混合在一起配置，**这里我强烈推荐使用Java代码配置方式，因为相对更安全，简介明了，也更符合逻辑**。
在写这篇文章之前，我就遇到了这个麻烦，在学习《Spring in Action》的过程中，照着他写的Java配置代码竟然不能运行，然后一模一样的示例代码完全ojbk，我在网上也找了很多教程，大部分还是比较老的XML配置的方式，我觉得代码被一堆<>淹没的感觉很不好。废话不多说，以一个官方文档的截图开始
![ServletConfig官方截图](https://img-blog.csdn.net/20180627102028138?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5Mzg1MTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2.容器初始化原理：
### 2.1 最重要的两个原理：
- WebApplicationInitializer是Spring MVC提供的接口，Spring MVC会自动检测到实现了这个接口的类，并用它来初始化Servlet 3.0+版本的容器。
- Spring MVC同时提供了实现这个接口的抽象类，名为AbstractDispatcherServletInitializer,还有干脆实现了这个接口的类，名为AbatractAnnotationConfigDispatcherServletInitializer,这次主要讲第一个接口实现的方式。


### 2.2 配置过程剖析：
在Servlet3.0环境中，容器会在类路径中查找实现ServletContainerInitializer接口的类，如果能发现的话，就会用它来配置Servlet容器。Spring提供了这个接口的实现（顺便提一下，在eclipse查看源码的方式是control+shift+T），名为SpringServletContainerInitializer，这个类反过来会查找实现WebApplicationInitializer接口的类，并将配置任务交给他来处理。
下面是WebApplicationInitializer源代码里面官方的实现方式：
![WebApplicationInitailizer初始化](https://img-blog.csdn.net/20180627165859170?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5Mzg1MTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```
package config;

import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRegistration;

import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.ContextLoaderListener;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

  @Override
  public void onStartup(ServletContext container) {
    // Create the 'root' Spring application context
    AnnotationConfigWebApplicationContext rootContext =
      new AnnotationConfigWebApplicationContext();
    rootContext.register(RootConfig.class);
  
    // Manage the lifecycle of the root application context
    container.addListener(new ContextLoaderListener(rootContext));
  
    // Create the dispatcher servlet's Spring application context
    AnnotationConfigWebApplicationContext dispatcherContext =
          new AnnotationConfigWebApplicationContext();
    dispatcherContext.register(WebConfig.class);
  
    // Register and map the dispatcher servlet
    ServletRegistration.Dynamic dispatcher =
          container.addServlet("dispatcher", new DispatcherServlet(dispatcherContext));
    dispatcher.setLoadOnStartup(1);
    dispatcher.addMapping("/");
      }
}
```
第一部分用于加载RooConfig配置类，这个用于配置由ContextLoaderListener创建的上下文，另一个WebConfig配置类用于创建Spring应用上下文。
我们希望DispatcherServlet加载包含Web组件的bean，如控制器、视图解析器以及处理器映射，而ContextLoaderListener加载应用中的其他bean。这些bean通常是驱动应用后端的中间件和数据层组件。
## 3.配置视图适配器
```
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
@ComponentScan
//适配器
public class WebConfig extends WebMvcConfigurerAdapter {

  @Bean
  public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
  }
  
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }
  
  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    // TODO Auto-generated method stub
    super.addResourceHandlers(registry);
  }

}

```
在这里配置好viewResolver解析器，负责讲控制器返回的String类型字符串名加上前缀和后缀，变成真正可以解析的网页地址。
## 4.配置控制器和jsp页面
```
package config;

import static org.springframework.web.bind.annotation.RequestMethod.*;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/home")
public class HomeController {

  @RequestMapping(method = GET)
  public String home(Model model) {
    return "home";
  }

}
```
配置控制器负责映射具体的请求，对应的方法返回具体的网页，简单的Spring MVC就搭建出来了