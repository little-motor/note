[toc]
# 1. 引言
注解@interface不是接口是注解类，在jdk1.5之后加入的功能，使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口。
在定义注解时，不能继承其他的注解或接口。@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。
在Java API文档中特意强调了如下内容：
Annotation是所有注释类型的公共扩展接口。注意，手动扩展这个接口并不定义注释类型。还要注意，这个接口本身并不定义注释类型。注释类型的更多信息可以在Java™语言规范的9.6节。
# 2. 语法规范
我们以spring中的@component注解为例，其格式如下所示
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {

	String value() default "";

}
```
其中主要包括四个部分
## 2.1 继承的Annotation父接口
```
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```
## 2.2 @Target中的参数ElementType
```
public enum ElementType {
    TYPE,               /* 类、接口（包括注释类型）或枚举声明  */

    FIELD,              /* 字段声明（包括枚举常量）  */

    METHOD,             /* 方法声明  */

    PARAMETER,          /* 参数声明  */

    CONSTRUCTOR,        /* 构造方法声明  */

    LOCAL_VARIABLE,     /* 局部变量声明  */

    ANNOTATION_TYPE,    /* 注释类型声明  */

    PACKAGE             /* 包声明  */
}
```
## 2.3 @Retention中的参数RetentionPolicy
```
public enum RetentionPolicy {
    SOURCE,            /* Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了  */

    CLASS,             /* 编译器将Annotation存储于类对应的.class文件中。默认行为  */

    RUNTIME            /* 编译器将Annotation存储于class文件中，并且可由JVM读入 */
}
```
## 2.4 成员变量
以无形参的方法形式来声明Annotation的成员变量，方法名和返回值定义了成员变量名称和类型。使用default关键字设置默认值，没设置默认值的变量则使用时必须提供，有默认值的变量使用时可以设置也可以不设置。
```
//定义带成员变量注解MyTag
@Rentention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyTag{
  //定义两个成员变量，以方法的形式定义
  String name();
  int age() default 20;
}

//使用
public class Test{
  @MyTag(name="test")
  public void info(){}
}
```

# 3. 获取注解信息
可以通过反射的方式获取注解在某个元素上的注解和其中的值，在Java反射类库reflect中，Class类实现了AnnotatedElement接口，其中包含多个获取annotation的方法
```
package java.lang.reflect;

    boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)；

    <T extends Annotation> T getAnnotation(Class<T> annotationClass);
    //这个方法检测其参数是否是可重复的注释类型(JLS 9.6)，如果是，则尝试通过“查看”容器注释来查找该类型的一个或多个注释。
    //该方法的调用者可以自由地修改返回的数组;它对返回给其他调用者的数组没有影响。
    <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass);

    Annotation[] getAnnotations();

    //如果直接存在指定类型的注释，则返回该元素的注释，否则为空。此方法忽略继承的注释。
    <T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass)；
    //与上面相同，考虑可重复的注释类型
    <T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass);

    //如果直接存在指定类型的注释，则返回该元素的注释，否则为空。此方法忽略继承的注释。
    Annotation[] getDeclaredAnnotations();

```
## 3.1 使用场景——框架初始化过程中递归获取注解以及注解的注解
```
/**
 * 递归搜索所有的注解，模拟框架搜索注解过程，以当前的类注解为例
 */
@Component
public class SearchMetaAnnations {
  public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    List<String> test = new ArrayList<>();
    Annotation[] annotations = SearchMetaAnnations.class.getAnnotations();
    Set<Annotation> set = new HashSet<>();
    for (Annotation annotation : annotations){
      recursivelyAnnotaion(annotation,set);
    }
    System.out.println(set);
  }

  //递归获取注解的注解
  public static void recursivelyAnnotaion(Annotation annotation, Set<Annotation> set){
    //通过set元素唯一的特点来实现递归搜索所有的注解
    if (set.add(annotation)){
      //获取注解的注解数组
      if (annotation.annotationType().getAnnotations().length > 0){
        //搜索子注解
        for (Annotation childAnnotation : annotation.annotationType().getAnnotations()){
          recursivelyAnnotaion(childAnnotation,set);
        }

      }
      System.out.println(annotation.annotationType().getName());
    }

  }
}
```
## 3.2 使用场景——框架初始化过程中模拟扫描jar包和文件夹中的所有注解
```
/**
 * 一个简单的模拟搜索包下面所有注解的工具类
 */
public class SearchAnnationsInPackage {

  public static void main(String[] args) {

    // 包下面的类
    Set<Class<?>> clazzs = getClasses("designPattern");
    if (clazzs == null) {
      return;
    }

    System.out.println( "class总量：" + clazzs.size());
    // 某类或者接口的子类
    //Set<Class<?>> inInterface = getByInterface(Object.class, clazzs);
    //System.out.printf(inInterface.size() + "");

    for (Class<?> clazz : clazzs) {

      // 获取类上的注解
      Annotation[] annos = clazz.getAnnotations();
      for (Annotation anno : annos) {
        System.out.println(clazz.getSimpleName().concat(".").concat(anno.annotationType().getSimpleName()));
      }

      // 获取方法上的注解
      Method[] methods = clazz.getDeclaredMethods();
      for (Method method : methods) {
        Annotation[] annotations = method.getDeclaredAnnotations();
        for (Annotation annotation : annotations) {
          System.out.println(clazz.getSimpleName().concat(".").concat(method.getName()).concat(".")
              .concat(annotation.annotationType().getSimpleName()));
        }
      }
    }
  }

  /**
   * 从包package中获取所有的Class
   *
   * @param pack
   * @return
   */
  public static Set<Class<?>> getClasses(String pack) {

    // 第一个class类的集合
    Set<Class<?>> classes = new LinkedHashSet<>();
    // 是否循环迭代
    boolean recursive = true;
    // 获取包的名字 并进行替换
    String packageName = pack;
    String packageDirName = packageName.replace('.', '/');
    // 定义一个枚举的集合 并进行循环来处理这个目录下的things
    Enumeration<URL> dirs;
    try {
      dirs = Thread.currentThread().getContextClassLoader().getResources(packageDirName);
      // 循环迭代下去
      while (dirs.hasMoreElements()) {
        // 获取下一个元素
        URL url = dirs.nextElement();
        // 得到协议的名称
        String protocol = url.getProtocol();
        // 如果是以文件的形式保存在服务器上
        if ("file".equals(protocol)) {
          System.err.println("file类型的扫描");
          // 获取包的物理路径
          String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
          // 以文件的方式扫描整个包下的文件 并添加到集合中
          findAndAddClassesInPackageByFile(packageName, filePath, recursive, classes);
        } else if ("jar".equals(protocol)) {
          // 如果是jar包文件
          // 定义一个JarFile
          // System.err.println("jar类型的扫描");
          JarFile jar;
          try {
            // 获取jar
            jar = ((JarURLConnection) url.openConnection()).getJarFile();
            // 从此jar包 得到一个枚举类
            Enumeration<JarEntry> entries = jar.entries();
            // 同样的进行循环迭代
            while (entries.hasMoreElements()) {
              // 获取jar里的一个实体 可以是目录 和一些jar包里的其他文件 如META-INF等文件
              JarEntry entry = entries.nextElement();
              String name = entry.getName();
              // 如果是以/开头的
              if (name.charAt(0) == '/') {
                // 获取后面的字符串
                name = name.substring(1);
              }
              // 如果前半部分和定义的包名相同
              if (name.startsWith(packageDirName)) {
                int idx = name.lastIndexOf('/');
                // 如果以"/"结尾 是一个包
                if (idx != -1) {
                  // 获取包名 把"/"替换成"."
                  packageName = name.substring(0, idx).replace('/', '.');
                }
                // 如果可以迭代下去 并且是一个包
                if ((idx != -1) || recursive) {
                  // 如果是一个.class文件 而且不是目录
                  if (name.endsWith(".class") && !entry.isDirectory()) {
                    // 去掉后面的".class" 获取真正的类名
                    String className = name.substring(packageName.length() + 1, name.length() - 6);
                    try {
                      // 添加到classes
                      classes.add(Class.forName(packageName + '.' + className));
                    } catch (ClassNotFoundException e) {
                      e.printStackTrace();
                    }
                  }
                }
              }
            }
          } catch (IOException e) {
            // log.error("在扫描用户定义视图时从jar包获取文件出错");
            e.printStackTrace();
          }
        }
      }
    } catch (IOException e) {
      e.printStackTrace();
    }

    return classes;
  }

  /**
   * 以文件的形式来获取包下的所有Class
   *
   * @param packageName
   * @param packagePath
   * @param recursive
   * @param classes
   */
  public static void findAndAddClassesInPackageByFile(String packageName, String packagePath, final boolean recursive,
                                                      Set<Class<?>> classes) {
    // 获取此包的目录 建立一个File
    File dir = new File(packagePath);
    // 如果不存在或者 也不是目录就直接返回
    if (!dir.exists() || !dir.isDirectory()) {
      // log.warn("用户定义包名 " + packageName + " 下没有任何文件");
      return;
    }
    // 如果存在 就获取包下的所有文件 包括目录
    File[] dirfiles = dir.listFiles(new FileFilter() {
      // 自定义过滤规则 如果可以循环(包含子目录) 或则是以.class结尾的文件(编译好的java类文件)
      public boolean accept(File file) {
        return (recursive && file.isDirectory()) || (file.getName().endsWith(".class"));
      }
    });
    // 循环所有文件
    for (File file : dirfiles) {
      // 如果是目录 则继续扫描
      if (file.isDirectory()) {
        findAndAddClassesInPackageByFile(packageName + "." + file.getName(), file.getAbsolutePath(), recursive,
            classes);
      } else {
        // 如果是java类文件 去掉后面的.class 只留下类名
        String className = file.getName().substring(0, file.getName().length() - 6);
        try {
          // 添加到集合中去
          // classes.add(Class.forName(packageName + '.' + className));
          // 经过回复同学的提醒，这里用forName有一些不好，会触发static方法，没有使用classLoader的load干净
          classes.add(
              Thread.currentThread().getContextClassLoader().loadClass(packageName + '.' + className));
        } catch (ClassNotFoundException e) {
          // log.error("添加用户自定义视图类错误 找不到此类的.class文件");
          e.printStackTrace();
        }
      }
    }
  }

  @SuppressWarnings({"rawtypes", "unchecked"})
  public static Set<Class<?>> getByInterface(Class clazz, Set<Class<?>> classesAll) {
    Set<Class<?>> classes = new LinkedHashSet<Class<?>>();
    // 获取指定接口的实现类
    if (!clazz.isInterface()) {
      try {
        /**
         * 循环判断路径下的所有类是否继承了指定类 并且排除父类自己
         */
        Iterator<Class<?>> iterator = classesAll.iterator();
        while (iterator.hasNext()) {
          Class<?> cls = iterator.next();
          /**
           * isAssignableFrom该方法的解析，请参考博客：
           * http://blog.csdn.net/u010156024/article/details/44875195
           */
          if (clazz.isAssignableFrom(cls)) {
            if (!clazz.equals(cls)) {// 自身并不加进去
              classes.add(cls);
            } else {

            }
          }
        }
      } catch (Exception e) {
        System.out.println("出现异常");
      }
    }
    return classes;
  }
}
```
# 4. Spring中的注解模式
请至[传送门](www)看我的另一篇设计模式相关的总结

# 5. 参考
https://www.runoob.com/w3cnote/java-annotation.html
https://www.jianshu.com/p/28edf5352b63
https://www.cnblogs.com/rinack/p/7606285.html