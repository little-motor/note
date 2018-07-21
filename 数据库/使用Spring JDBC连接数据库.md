[toc]
## 1. 引言
不管你选择哪种持久化方式，Spring都能提供支持。
### 1.1 Spring的数据访问哲学
Spring的目标之一就是允许我们在开发应用程序时，能够遵循面向对象(OO)原则中的“针对接口编程”。为了避免持久化的逻辑分散到应用的各个组件中，最好将数据访问的功能放在一个或多个专注于此项任务的组件中。这样的组件通常称为数据访问对象(data access object,DAO)或Repository。
为了避免应用与特定的数据访问策略耦合在一起，便携良好的Repository应该以接口的方式暴露功能，下图展现了设计数据访问层的合理方式。
![数据层访问的合理方式](https://img-blog.csdn.net/20180721154633490?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5Mzg1MTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
服务对象本身并不会处理数据访问，而是将数据访问委托给Repository。Repository接口确保其与服务对象的松耦合。良好的设计方法是将持久层隐藏在接口之后。
## 2. 了解Spring的数据访问异常体系
在不使用Spring编写JDBC时，我们要捕获SQLException，可能抛出SQLException的常见问题包括：
- 应用程序无法连接数据库
- 要执行的查询存在语法错误
- 查询中所使用的表或列不存在
- 试图插入或更新的数据违反了数据库约束

事实上，能够触发SQLException的问题通常是不能在catch代码中解决的。大多数抛出SQLException的情况表明发生了致命性错误，那既然无法从SQLException中恢复，那为什么我们还要强制捕获它呢。同时SQLException不能明确的表明错误类型和位置。
Spring JDBC提供的数据访问异常体系解决了上述问题，Spring提供了多个数据访问异常，分别描述了它们抛出时所对应的问题，同时他并没有与特定的持久化方式相关联，这有助于我们将所选择持久化机制与数据访问层隔离开来。
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