[toc]
## 1. 引言
不管你选择哪种持久化方式，Spring都能提供支持。
### 1.1 Spring的数据访问哲学
Spring的目标之一就是允许我们在开发应用程序时，能够遵循面向对象(OO)原则中的“针对接口编程”。为了避免持久化的逻辑分散到应用的各个组件中，最好将数据访问的功能放在一个或多个专注于此项任务的组件中。这样的组件通常称为数据访问对象(data access object,DAO)或Repository。
为了避免应用与特定的数据访问策略耦合在一起，编写良好的Repository应该以接口的方式暴露功能，下图展现了设计数据访问层的合理方式。
![数据层访问的合理方式](https://img-blog.csdn.net/20180721154633490?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5Mzg1MTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
服务对象本身并不会处理数据访问，而是将数据访问委托给Repository。Repository接口确保其与服务对象的松耦合。良好的设计方法是将持久层隐藏在接口之后。
## 2. 了解Spring的数据访问异常体系
在不使用Spring编写JDBC时，我们要捕获SQLException，可能抛出SQLException的常见问题包括：

- 应用程序无法连接数据库
- 要执行的查询存在语法错误
- 查询中所使用的表或列不存在
- 试图插入或更新的数据违反了数据库约束

事实上，能够触发SQLException的问题通常是不能在catch代码中解决的。大多数抛出SQLException的情况表明发生了致命性错误，那既然无法从SQLException中恢复，那为什么我们还要强制捕获它呢。同时SQLException不能明确的表明错误类型和位置。
SpringJDBC提供的数据访问异常体系解决了上述问题，Spring提供了多个数据访问异常，分别描述了它们抛出时所对应的问题，同时他并没有与特定的持久化方式相关联，这有助于我们将所选择持久化机制与数据访问层隔离开来。
### 2.1 不用写catch代码块
Spring JDBC异常都继承自DataAccessException。这是一个非检查型异常，换句话说没必要强制捕获Spring所抛出的数据访问异常。
DataAccessException是Spring处理检查型异常和非检查型异常哲学的一个范例。Spring认为触发异常的很多问题是不能在catch代码块中修复的。Spring使用了非检查型异常，而不是强制开发人员编写catch代码块（里面经常是空的），这把是否捕获异常的权利留给了开发人员。
为了利用Spring的数据访问异常，必须使用Spring所支持的数据访问模板。
## 3. 数据访问模板化
Spring将数据访问过程中固定的和可变的部分明确划分为两个不同的类：模板(template)和回调(callback)。模板管理过程中固定的部分，而回调处理自定义的数据访问代码。

模板类(org.springframework.*) | 用  途 |
----------|-----------------------
 jca.cci.core.CciTemplate | JCA CCI连接
 jdbc.core.JdbcTemplate | JDBC 连接
 jdbc.core.namedparam.NamedParameterJdbcTemplate | 支持命名参数的JDBC连接
 orm.hibernate3.HibernateTemplate|Hibernate 3.x以上的Session
 orm.ibatis.SqlMapClientTemplate|iBATIS SqlMap客户端
 orm.jdo.JdoTemplate|Java数据对象实现
 orm.jpa.JpaTemplate|Java持久化API的实体管理器
 由于Spring所支持的大多数持久化功能都依赖于数据源，所以在声明模板和Repository之前，需要在Spring中配置一个数据源来连接数据库。

## 4. 配置数据源
无论选择Spring的哪种数据访问方式，都需要一个数据源引用。Spring提供了在Spring上下文中配置数据源bean的多种方式

- 通过JDBC驱动程序定义的数据源
- 通过JNDI查找的数据源
- 连接池的数据源
### 4.1 使用JNDI数据源
JNDI(Java Naming and Directory Interface)是J2EE重要的的规范之一，要求所有J2EE容器都要提供JNDI规范的实现。这些服务器允许你配置通过JNDI获取数据源。这种配置的好处在于数据源完全可以在应用程序之外进行管理，这样应用程序只需要在访问数据库的时候查找数据源就可以了。
```
@Bean
public JndiObjectFactoryBean dataSource(){
  JndiObjectFactoryBean jndiObjectFB = new JndiObjectFactoryBean();
  jndiObjectFB.setJndiName("jdbc/H2");
  jndiObjectFB.setResourceRef(true);
  jndiObjectFB.setProxyInterface(javax.sql.DataSource.class);
  return jndiObjectFB;
}
```
### 4.2 使用数据源连接池
### 4.3 基于JDBC驱动的数据源
通过JDBC驱动定义数据源是最简单的配置方式，Spring提供了三个这样的数据源类(均位于org.springframework.jdbc.datasource包中)供选择。

- DriverManagerDataSource:在每个连接请求时都会返回一个新建的连接。
- SimpleDriverDataSource
- SingleConnectionDataSource：在每个连接请求时都会返回同一个的连接。
如下就是配置DriverManagerDataSource的方法：
```
@Bean
public DataSource dataSource(){
  DriverManagerDataSource ds = new DriverManagerDataSource();
  ds.setDriverClassName("org.h2.Driver");
  ds.setUrl("jdbc:h2:tcp://localhost/~/...");
  ds.setUsername("...");
  ds.setPassword("");
  return ds;
}
```
与具备池功能的数据源相比，唯一的区别在于这些数据源bean都没有提供连接池功能，所以没有可配置的池相关的属性。
### 4.4 使用嵌入式的数据源
嵌入式数据库(embedded database)作为应用的一部分运行，而不是应用连接的独立数据库服务器。在开发和测试环境中，嵌入式数据库是个很好的可选方案，如下是使用Java来配置嵌入式数据库。
```
//返回嵌入式数据库的数据源
  @Bean
  public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
               .setType(EmbeddedDatabaseType.H2)
               .addScript("classpath:schema.sql")
               .build();
  }
```
## 5. 在Spring中使用JDBC
在传统的JDBC代码中，约有20%的代码是真正用于查询数据的，而80%代码都是样板代码，但是我们不仅需要这些代码，同时还要保证他是正确的，基于这样的原因，我们才需要框架来保证这些代码只写一次而且是正确的。
Spring将数据访问的样板代码抽象到模板类中，Spring为JDBC提供两种模板类：

- JdbcTemplate：最基本的Spring JDBC模板，这个模板支持简单的JDBC数据库访问以及基于索引参数的查询
- NamedParameterJdbcTemplate：使用该模板类执行查询可以将值以命名参数的形式绑定到SQL中，而不是使用简单的索引参数
### 5.1 使用JdbcTemplate来插入数据
为了让JdbcTemplate正常工作，只需要为其设置DataSource就可以了
```
  //得到JdbcTemplate实例
  //JdbcOperations是JdbcTemplate类实现的接口
  @Bean
  public JdbcOperations jdbcTemplate(DataSource dataSource) {  
    return new JdbcTemplate(dataSource);
  } 
```
将JdbcOperations的bean装配到数据访问对象Rrepository中并使用他来访问数据库。
```
package model;


//Repository组件包含Component注解
@Repository
public class JdbcMicroPostRepository implements MicroPostRepository {
  //此处的接口通过注入后其实是类的实例
  private JdbcOperations jdbcOperations;
  
  //JdbcOperations接口定义了JdbcaTemplate实现的操作
  //通过注入JdbcOperations而不是具体的JdbcTemplate能够保证
  //这个数据访问类与JdbcTemplate保持松耦合
  @Autowired
  public JdbcMicroPostRepository(JdbcOperations jdbcOperations) {
    this.jdbcOperations = jdbcOperations;
  }
  
  ...
  
  //插入内容
  private static final String SQL_INSERT_MICROPOST = 
      "insert into Micropost (id,message,created_at)"
      + "values (?,?,?)";
  //查询内容
  private static final String SQL_SELECT_MICROPOST =
      "select * from Micropost where id = ?";
}

```
jdbcMicroPostRepository类上使用的@Repository注解内部包含@Componnet注解，通过构造器上的@Autowired注解，构造器会获得一个JdbcOperations对象，JdbcOperations接口定义了JdbcTemplate实现的操作，此处动过接口注入而不是具体的类，能够保证jdbcMicroPostRepository通过JdbcOperations接口达到与JdbcTemplate保持松耦合。
基于JdbcTemplate的addMicroPost方法如下
```
  //添加信息
  @Override
  public void addMicroPost(MicroPost microPost) {
    jdbcOperations.update(SQL_INSERT_MICROPOST, 
        microPost.getId(), 
        microPost.getMessage(),
        microPost.getTime());
  }
```
### 5.2 使用JdbcTemplate来查找数据
JdbcTemplate也简化了数据的读取操作，使用JdbcTemplate的回调，实现根据ID查询MicroPost，并将结果映射为MicroPost对象，这里以getMicroPost方法为例。
```

  //查询消息
  //第一个和第三个参数分别为sql和参数
  //第二个参数为RowMapper对象，JdbcTemplate会调用RowMapper
  //的mapRow方法，并传入ResultSet和行号
  @Override
  public MicroPost getMicroPost(long id) {
    return jdbcOperations.queryForObject(SQL_SELECT_MICROPOST,
        new MicroPostRowMapper(), 
        id);     
    }
  
  //由JdbcTemplate的queryForObject方法调用mapRow方法
  //传入ResultSet对象和行号
  private static final class MicroPostRowMapper
  implements RowMapper<MicroPost>{
  @Override
  public MicroPost mapRow(ResultSet rs, int rowNum) throws SQLException {
    return new MicroPost(rs.getLong("id"), 
        rs.getString("message"),
        rs.getDate("created_at"));
    }
  }
  
```
queryForObject()方法有三个参数：

- String对象，包含了要从数据库中查找数据的SQL
- RowMapper对象，用来从ResultSet提取数据并构建域对象
- 可变参数列表，列出要绑定到查询上的索引参数值

JdbcTemplate将会调用RowMapper的mapRow方法，并传入一个ResultSet和包含行号的整数，不同于传统的JDBC，这里没有资源管理或者异常处理代码。
另外也可以通过Lambda表达式和方法引用的方式实现查询方法，此处不再赘述，详见《Spring实战》p308。
## 6. 小结
在Java中，JDBC是与关系型数据库交互的最基本方式，但是按照规范，JDBC有些太笨重了，Spring能够解除JDBC中的大多数痛苦，包括消除样板代码、简化异常处理，而仅仅需要关注执行的SQL语句。

