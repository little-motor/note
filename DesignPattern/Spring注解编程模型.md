[toc]
# 1. 引言
模式注解使框架的配置变得简洁明了，从Spring Framework 3.1开始Spring开始全面支持面向注解配置，其中一些核心注解如下

Spring模式注解：
 | Spring 注解    | 场景说明          | 起始版本 |
 | -------------- | ----------------- | -------- |
 | @Repository    | 数据仓库模式注解  | 2.0      |
 | @Component     | 通用组件模式注解  | 2.5      |
 | @Service       | 服务模式注解      | 2.5      |
 | @Controller    | Web控制器模式注解 | 2.5      |
 | @Configuration | 配置类模式注解    | 3.0      |

装配注解：
  | Spring 注解     | 场景说明                                | 起始版本 |
  | --------------- | --------------------------------------- | -------- |
  | @ImportResource | 替换XML元素<import>                     | 2.5      |
  | @Import         | 导入bean或者@configuration配置类        | 3.0      |
  | @ComponentScan  | 扫描指定package下标注Spring模式注解的类 | 3.1      |

依赖注入注解如下表所示：
 | Spring 注解 | 场景说明                           | 起始版本 |
 | ----------- | ---------------------------------- | -------- |
 | @Autowired  | Bean依赖注入，支持多种依赖查找方式 | 2.5      |
 | @Qualifier  | 细粒度限定@autowired注入           | 2.5      |

  | Spring 注解 | 场景说明                              | 起始版本 |
  | ----------- | ------------------------------------- | -------- |
  | @Resource   | @Bean依赖注入，仅支持名称依赖查找方式 | 2.5      |

  Bean定义注解如下表所示
   | Spring 注解 | 场景说明    | 起始版本 |
   | ----------- | ----------- | -------- |
   | @Bean       | 替换XML元素 | 3.0      |
   | @DependsOn  | 替换XML元素 | 3.0      |
   | @Lazy       | 替换XML元素 | 3.0      |
   | @Primary    | 替换XML元素 | 3.0      |
   | @Role       | 替换XML元素 | 3.1      |
   | @Lookup     | 替换XML元素 | 4.1      |
   
   Spring条件装配注解：
   | Spring 注解  | 场景说明       | 起始版本 |
   | ------------ | -------------- | -------- |
   | @Profile     | 配置化条件装配 | 3.1      |
   | @Conditional | 编程条件装配   | 4.0      |

   配置属性注解：
   | Spring 注解      | 场景说明                | 起始版本 |
   | ---------------- | ----------------------- | -------- |
   | @PropertySource  | 配置属性抽象            | 3.1      |
   | @PropertySources | @PropertySource集合注解 | 4.0      |

   生命周期回调注解
   | Java 注解      | 场景说明    | 起始版本 |
   | -------------- | ----------- | -------- |
   | @PostConstruct | 替换XML元素 | 2.5      |
   | @PreDestroy    | 替换XML元素 | 2.5      |

   可以标注注解的属性
   | Java 注解 | 场景说明                     | 起始版本 |
   | --------- | ---------------------------- | -------- |
   | @AliasFor | 别名注解属性，实现复用的目的 | 4.2      |

   性能注解：
| Java 注解 | 场景说明                     | 起始版本 |
| --------- | ---------------------------- | -------- |
| @Indexed  | 提升Spring模式注解的扫描效率 | 5.0      |

# 2. Spring注解编程模型
主要有四个方面分别是：
- 元注解(Meta-Annotations)
- Spring模式注解(Stereotype Annotations)
- Spring组合注解(Composed Annotations)
- Spring注解属性别名和覆盖(Attribute Aliases and Overrides)
## 2.1 元注解(Meta-Annotations)
元注解指一个能声明在其他注解上的注解，如果一个注解标注在其他注解上，那么他就是元注解。根据Java语言规范，注解之间不存在继承关系，因此可以通过元注解来注解到另一个注解达到“派生”的作用，这种“派生”特性需要确保注解之间的属性方法签名完全一致.
## 2.2 Spring模式注解（Stereotype Annotations）
引用Spring重的Wiki描述里的一句话
>A sterotype annotation is an annotation that is used to declare the role that a componnet plays within the application.

简而言之Stereotype Annotations就是说明组件扮演的角色，以@Component注解为例，@Service、@Repository、@Controler、@RestController及@Configuration都包含@Component注解，所以包含@Component的功能，同时他们在不同的场景下扮演不同的角色。在Spring Framework 4.x 之前以@Component注解为例，只能识别两层的@Component派生注解，在Spring Framework 4.x 之后可以递归识别多层次的@Component派生注解。
Spring初始化过程中扫描注解的过程简单的原理我在另一篇[Java基础 注解](https://blog.csdn.net/qq_39385118/article/details/99975996#5_Spring_357)中有简单分析了一下，主要过程就是根据配置的basePackage属性到该路径下搜索有相应注解标记的类，然后注入到容器当中。需要另外注意的是Spring在扫描的过程中还会有excludeFilters和includeFilters，排除excludeFilters中的注解，属于includeFilters中的注解将会通过筛选条件（默认初始化时includeFilters会加入@Component）。
## 2.3 Spring组合注解（Composed Annotations）

   



   







