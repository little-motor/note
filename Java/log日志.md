[toc]
# 1. 引言
logback日志框架使用总结，Slf4j是The Simple Logging Facade for Java的简称，是一个简单日志门面抽象框架，它本身只提供了日志Facade API和一个简单的日志类实现，一般常配合Log4j，LogBack，java.util.logging使用。Slf4j作为应用层的Log接入时，程序可以根据实际应用场景动态调整底层的日志实现框架(Log4j/LogBack/JdkLog等)
LogBack官方建议配合Slf4j使用，这样可以灵活地替换底层日志框架。LogBack和Log4j都是开源日记工具库，LogBack是Log4j的改良版本，比Log4j拥有更多的特性，同时也带来很大性能提升。
# 2. LogBack的结构
LogBack被分为3个组件，logback-core, logback-classic 和 logback-access。

其中logback-core提供了LogBack的核心功能，是另外两个组件的基础。

logback-classic则实现了Slf4j的API，所以当想配合Slf4j使用时，需要将logback-classic加入classpath。

logback-access是为了集成Servlet环境而准备的，可提供HTTP-access的日志接口。
# 3. LogBack配置
我们简单分析一下logback加载过程，当我们使用logback-classic.jar时，应用启动，那么logback会按照如下顺序进行扫描：
1. 在系统配置文件System Properties中寻找是否有logback.configurationFile对应的value
2. 在classpath下寻找是否有logback.groovy（即logback支持groovy与xml两种配置方式）
3. 在classpath下寻找是否有logback-test.xml
4. 在classpath下寻找是否有logback.xml

以上任何一项找到了，就不进行后续扫描，按照对应的配置进行logback的初始化，具体代码实现可见ch.qos.logback.classic.util.ContextInitializer类的findURLOfDefaultConfigurationFile方法。
## 3.1 根节点&lt;configuration&gt;
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。

scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。

debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
```
<configuration scan="true" scanPeriod="60 second" debug="false">  
      <!-- 其他配置省略-->  
</configuration>
```
## 3.2 &lt;configuration&gt;的子节点
LogBack的配置大概包括3部分：appender, logger和root。
### 3.2.1 &lt;contextName&gt;
每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用&lt;contextName&gt;设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。
```
<configuration scan="true" scanPeriod="60 second" debug="false">  
      <contextName>myAppName</contextName>  
      <!-- 其他配置省略-->  
</configuration>
```
### 3.2.2 &lt;property&gt;
用来定义变量值的标签，&lt;property&gt;有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值。通过&lt;property&gt;定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。

例如使用&lt;property&gt;定义上下文名称，然后在&lt;contentName&gt;设置logger上下文时使用。
```
<configuration scan="true" scanPeriod="60 second" debug="false">  
      <property name="APP_Name" value="myAppName" />   
      <contextName>${APP_Name}</contextName>  
      <!-- 其他配置省略-->  
</configuration>
```
### 3.2.3 &lt;timestamp&gt;
两个属性 key:标识此&lt;timestamp&gt; 的名字；datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循Java.text.SimpleDateFormat的格式。
例如将解析配置文件的时间作为上下文名称：
```
<configuration scan="true" scanPeriod="60 second" debug="false">  
      <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>   
      <contextName>${bySecond}</contextName>  
      <!-- 其他配置省略-->  
</configuration>
```
### 3.2.4 &lt;logger&gt;
用来设置某一个包或者具体的某一个类的日志打印级别、以及指定&lt;appender&gt;。&lt;logger&gt;仅有一个name属性，一个可选的level和一个可选的additivity属性。
- name：用来指定受此logger约束的某一个包或者具体的某一个类。
- level：用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。
如果未设置此属性，那么当前logger将会继承上级的级别。
- additivity：是否向上级logger传递打印信息。默认是true。

&lt;logger&gt;可以包含零个或多个&lt;appender-ref>元素，标识这个appender将会添加到这个logger。
### 3.2.5 &lt;root>
也是&lt;logger>元素，但是它是根logger。只有一个level属性，因为已经被命名为”root”.
- level：用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。默认是DEBUG。
&lt;root>可以包含零个或多个&lt;appender-ref>元素，标识这个appender将会添加到这个logger。

案例介绍
Java类如下：
```
package logback;  
 
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
 
public class LogbackDemo {  
    private static Logger log = LoggerFactory.getLogger(LogbackDemo.class);  
    public static void main(String[] args) {  
        log.trace("======trace");  
        log.debug("======debug");  
        log.info("======info");  
        log.warn("======warn");  
        log.error("======error");  
    }  
}
```
logback.xml配置文件
```
<configuration>   
 
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
    <!-- encoder 默认配置为PatternLayoutEncoder -->   
    <encoder>   
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>   
    </encoder>   
  </appender>   
 
  <root level="INFO">             
    <appender-ref ref="STDOUT" />   
  </root>     
 
 </configuration>
```
当执行logback.LogbackDemo类的main方法时，root将级别为“INFO”及大于“INFO”的日志信息交给已经配置好的名为“STDOUT”的appender处理，并将大于等于“INFO”级别的信息打印到控制台
```
13:30:38.484 [main] INFO  logback.LogbackDemo - ======info  
13:30:38.500 [main] WARN  logback.LogbackDemo - ======warn  
13:30:38.500 [main] ERROR logback.LogbackDemo - ======error
```
带有logger的配置，不指定级别，不指定appender
```
<configuration>   
 
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">   
    <!-- encoder 默认配置为PatternLayoutEncoder -->   
    <encoder>   
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>   
    </encoder>   
  </appender>   
 
  <!-- logback为java中的包 -->   
  <logger name="logback"/>   
 
  <root level="DEBUG">             
    <appender-ref ref="STDOUT" />   
  </root>     
 
 </configuration>
```
&lt;logger name=”logback” />将控制logback包下的所有类的日志的打印，但是并没有设置打印级别，所以继承他的上级&lt;root>的日志级别“DEBUG”。
没有设置additivity，默认为true，将此logger的打印信息向上级传递。
没有设置appender，此logger本身不打印任何信息。
&lt;root level=”DEBUG”>将root的打印级别设置为“DEBUG”，指定了名字为“STDOUT”的appender。
当执行logback.LogbackDemo类的main方法时，因为LogbackDemo 在包logback中，所以首先执行&lt;logger name=”logback” />，将级别为“DEBUG”及大于“DEBUG”的日志信息传递给root，本身并不打印。
root接到下级传递的信息，交给已经配置好的名为“STDOUT”的appender处理，“STDOUT”appender将信息打印到控制台。打印效果如下：
```
13:19:15.406 [main] DEBUG logback.LogbackDemo - ======debug  
13:19:15.406 [main] INFO  logback.LogbackDemo - ======info  
13:19:15.406 [main] WARN  logback.LogbackDemo - ======warn  
13:19:15.406 [main] ERROR logback.LogbackDemo - ======error
```
参考：
http://www.importnew.com/22290.html