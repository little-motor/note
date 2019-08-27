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
扫描注解并模拟初始化过程简单的原理我在另一篇[Java基础 注解](https://blog.csdn.net/qq_39385118/article/details/99975996#5_Spring_357)中有简要的分析了一下，主要过程就是根据配置的basePackage属性到该路径下搜索有相应注解标记的类。
需要另外注意的是Spring在扫描的过程中还会有excludeFilters和includeFilters，排除excludeFilters中的注解，属于includeFilters中的注解将会通过筛选条件（默认初始化时includeFilters会加入@Component）。
还有一点是，处于性能方面的Spring获取底层元数据的方式是通过ASM实现的，而不是通过反射，如ClassReader类，相对于ClassLoader体系，Spring ASM更为底层，读取的是类资源，直接操作的是其中的字节码，获取相关元信息，在读取元信息方面Spring抽象出MetadataReader接口。
## 2.3 Spring组合注解（Composed Annotations）
其目的在于将多个注解行为组合成单个自定义注解，比如@SpringBootApplication注解由@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan等几个注解组成，Spring并没有考虑通过Java反射的手段来解析元注解信息，而是抽象出AnnotationMetadata接口，其实现类为AnnotationMetadataReadingVisitor，从Spring 4.0开始，在初始化过程中AnnotationMetadataReadingVisitor所关联的AnnotationAttributesReadingVisitor采用递归查找元注解，并保存在AnnotationMetadataReadingVisitor的metaAnnotationMap字段中。
其中核心代码如下：
```
package org.springframework.core.type.classreading;

/**
 * ASM visitor which looks for annotations defined on a class or method,
 * including meta-annotations.
 *
 * <p>This visitor is fully recursive, taking into account any nested
 * annotations or nested annotation arrays.
 *
 * @author Juergen Hoeller
 * @author Chris Beams
 * @author Phillip Webb
 * @author Sam Brannen
 * @since 3.0
 */
final class AnnotationAttributesReadingVisitor extends RecursiveAnnotationAttributesVisitor {

	private final MultiValueMap<String, AnnotationAttributes> attributesMap;

	private final Map<String, Set<String>> metaAnnotationMap;


	public AnnotationAttributesReadingVisitor(String annotationType,
			MultiValueMap<String, AnnotationAttributes> attributesMap, Map<String, Set<String>> metaAnnotationMap,
			@Nullable ClassLoader classLoader) {

		super(annotationType, new AnnotationAttributes(annotationType, classLoader), classLoader);
		this.attributesMap = attributesMap;
		this.metaAnnotationMap = metaAnnotationMap;
	}


	@Override
	public void visitEnd() {
		super.visitEnd();

		Class<? extends Annotation> annotationClass = this.attributes.annotationType();
		if (annotationClass != null) {
			List<AnnotationAttributes> attributeList = this.attributesMap.get(this.annotationType);
			if (attributeList == null) {
				this.attributesMap.add(this.annotationType, this.attributes);
			}
			else {
				attributeList.add(0, this.attributes);
			}
			if (!AnnotationUtils.isInJavaLangAnnotationPackage(annotationClass.getName())) {
				try {
					Annotation[] metaAnnotations = annotationClass.getAnnotations();
					if (!ObjectUtils.isEmpty(metaAnnotations)) {
						Set<Annotation> visited = new LinkedHashSet<>();
						for (Annotation metaAnnotation : metaAnnotations) {
              //递归查找元注解
							recursivelyCollectMetaAnnotations(visited, metaAnnotation);
						}
						if (!visited.isEmpty()) {
							Set<String> metaAnnotationTypeNames = new LinkedHashSet<>(visited.size());
							for (Annotation ann : visited) {
								metaAnnotationTypeNames.add(ann.annotationType().getName());
							}
							this.metaAnnotationMap.put(annotationClass.getName(), metaAnnotationTypeNames);
						}
					}
				}
				catch (Throwable ex) {
					if (logger.isDebugEnabled()) {
						logger.debug("Failed to introspect meta-annotations on " + annotationClass + ": " + ex);
					}
				}
			}
		}
	}

  //递归查找元注解
	private void recursivelyCollectMetaAnnotations(Set<Annotation> visited, Annotation annotation) {
		Class<? extends Annotation> annotationType = annotation.annotationType();
		String annotationName = annotationType.getName();
		if (!AnnotationUtils.isInJavaLangAnnotationPackage(annotationName) && visited.add(annotation)) {
			try {
				// Only do attribute scanning for public annotations; we'd run into
				// IllegalAccessExceptions otherwise, and we don't want to mess with
				// accessibility in a SecurityManager environment.
				if (Modifier.isPublic(annotationType.getModifiers())) {
					this.attributesMap.add(annotationName,
							AnnotationUtils.getAnnotationAttributes(annotation, false, true));
				}
				for (Annotation metaMetaAnnotation : annotationType.getAnnotations()) {
					recursivelyCollectMetaAnnotations(visited, metaMetaAnnotation);
				}
			}
			catch (Throwable ex) {
				if (logger.isDebugEnabled()) {
					logger.debug("Failed to introspect meta-annotations on " + annotation + ": " + ex);
				}
			}
		}
	}

}

```
### 2.3.1 MetadataReader
在读取元信息方面，Spring抽象出MetadataReader接口：
```
package org.springframework.core.type.classreading;

/**
 * Simple facade for accessing class metadata,
 * as read by an ASM {@link org.springframework.asm.ClassReader}.
 *
 * @author Juergen Hoeller
 * @since 2.5
 */
public interface MetadataReader {

	/**
	 * Return the resource reference for the class file.
	 */
	Resource getResource();

	/**
	 * Read basic class metadata for the underlying class.
	 */
	ClassMetadata getClassMetadata();

	/**
	 * Read full annotation metadata for the underlying class,
	 * including metadata for annotated methods.
	 */
	AnnotationMetadata getAnnotationMetadata();

}

```
 需要注意的是无论是ClassMetadata还是AnnotationMetadata，均没有Java Class和Annotation API那样丰富的关联属性，MetadataReader有明显的资源特性，getResource()方法关联了类资源的Resource信息，他在Spring Framework中有一个final实现类SimpleMetadataReader，其关联的ClassMetadata和AnnotationMetadata信息在构造阶段完成初始化
```
package org.springframework.core.type.classreading;

/**
 * {@link MetadataReader} implementation based on an ASM
 * {@link org.springframework.asm.ClassReader}.
 *
 * <p>Package-visible in order to allow for repackaging the ASM library
 * without effect on users of the {@code core.type} package.
 *
 * @author Juergen Hoeller
 * @author Costin Leau
 * @since 2.5
 */
final class SimpleMetadataReader implements MetadataReader {

	private final Resource resource;

	private final ClassMetadata classMetadata;

	private final AnnotationMetadata annotationMetadata;


	SimpleMetadataReader(Resource resource, @Nullable ClassLoader classLoader) throws IOException {
		InputStream is = new BufferedInputStream(resource.getInputStream());
		ClassReader classReader;
		try {
			classReader = new ClassReader(is);
		}
		catch (IllegalArgumentException ex) {
			throw new NestedIOException("ASM ClassReader failed to parse class file - " +
					"probably due to a new Java class file version that isn't supported yet: " + resource, ex);
		}
		finally {
			is.close();
		}

		AnnotationMetadataReadingVisitor visitor = new AnnotationMetadataReadingVisitor(classLoader);
		classReader.accept(visitor, ClassReader.SKIP_DEBUG);

		this.annotationMetadata = visitor;
		// (since AnnotationMetadataReadingVisitor extends ClassMetadataReadingVisitor)
		this.classMetadata = visitor;
		this.resource = resource;
	}


	@Override
	public Resource getResource() {
		return this.resource;
	}

	@Override
	public ClassMetadata getClassMetadata() {
		return this.classMetadata;
	}

	@Override
	public AnnotationMetadata getAnnotationMetadata() {
		return this.annotationMetadata;
	}

}

```
AnnotationMetadataReadingVisitor同时实现了ClassMetadata和AnnotationMetadata接口，在ClassPathScanningCandidateComponentProvider#findCandidateComponents(String)方法中MetadataReader实例由由metadataReaderFactory.getMetadateReader实例生成。
```
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
		Set<BeanDefinition> candidates = new LinkedHashSet<>();
		try {
			String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
					resolveBasePackage(basePackage) + '/' + this.resourcePattern;
			Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
			boolean traceEnabled = logger.isTraceEnabled();
			boolean debugEnabled = logger.isDebugEnabled();
			for (Resource resource : resources) {
				if (traceEnabled) {
					logger.trace("Scanning " + resource);
				}
				if (resource.isReadable()) {
					try {
            //默认是CachingMetadataReaderFactory
						MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
						if (isCandidateComponent(metadataReader)) {
							ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
							sbd.setResource(resource);
							sbd.setSource(resource);
							if (isCandidateComponent(sbd)) {
								if (debugEnabled) {
									logger.debug("Identified candidate component class: " + resource);
								}
								candidates.add(sbd);
							}
							else {
								if (debugEnabled) {
									logger.debug("Ignored because not a concrete top-level class: " + resource);
								}
							}
						}
						else {
							if (traceEnabled) {
								logger.trace("Ignored because not matching any filter: " + resource);
							}
						}
					}
					catch (Throwable ex) {
						...
					}
				}
				else {
					if (traceEnabled) {
						logger.trace("Ignored because not readable: " + resource);
					}
				}
			}
		}
		catch (IOException ex) {
		  ...
		}
		return candidates;
	}

```
这里的getMetadataReaderFactory()默认是CachingMetadataReaderFactory，他提供getMetadataReader两种重载方法
```
public interface MetadataReaderFactory {

	/**
	 * Obtain a MetadataReader for the given class name.
	 * @param className the class name (to be resolved to a ".class" file)
	 * @return a holder for the ClassReader instance (never {@code null})
	 * @throws IOException in case of I/O failure
	 */
	MetadataReader getMetadataReader(String className) throws IOException;

	/**
	 * Obtain a MetadataReader for the given resource.
	 * @param resource the resource (pointing to a ".class" file)
	 * @return a holder for the ClassReader instance (never {@code null})
	 * @throws IOException in case of I/O failure
	 */
	MetadataReader getMetadataReader(Resource resource) throws IOException;

}
```
下面我们通过一个简单的栗子来通过MetadataReader获取一下元注解
```
/**
 * 通过MetadataReaderFactory获取metadataReader实例并获取元信息
 */
@RestController
public class AnnotationMetadataInfo {

  public static void main(String[] args) throws IOException {
    String className = AnnotationMetadataInfo.class.getName();
    MetadataReaderFactory metadataReaderFactory = new CachingMetadataReaderFactory();
    //通过class全限定名称获取MetadataReader实例
    MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(className);
    //获取当前类的注解元信息
    AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
    //获取当前类的注解元信息的全限定名
    Set<String> set = annotationMetadata.getAnnotationTypes();
    System.out.println(set);
    set.forEach(annotationTypeName -> {
      //通过注解全限定名找到其所有的元注解
      annotationMetadata.getMetaAnnotationTypes(annotationTypeName).forEach(
          metaAnnotationTypeName -> {
            System.out.println(metaAnnotationTypeName);
          }
      );
    });
  }
}
```
## 2.4 Spring注解属性别名和覆盖(Attribute Aliases and Overrides)







