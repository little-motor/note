[toc]
## 1. 引言
MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。
## 2. 快速入门
### 2.1 从 XML 中构建 SqlSessionFactory
每个基于 MyBatis 的应用都是以一个SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过SqlSessionFactoryBuilder 获得。而SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。构建SqlSessionFactory的过程如下
```
String resource = "../mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
XML 配置文件（configuration XML）的简单示例如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```
### 2.2 从 SqlSessionFactory 中获取 SqlSession
SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。
```
SqlSession session = sqlSessionFactory.openSession();
```
执行查询有两种方式，更推荐第二种方式，因为可以执行更清晰和类型安全的代码，而且还不用担心易错的字符串字面值以及强制类型转换。
```
//第一种
try {
  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
  session.close();
}

//第二种
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}
```
### 2.3 mapper文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace用于绑定接口,推荐使用全限定名 -->
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```
映射调用和使用完全限定名调用 Java 对象的方法非常相似，这个命名可以直接映射到在命名空间中同名的 Mapper 类，并将已映射的 select 语句中的名字、参数和返回类型匹配成方法。这样你就可以像上面那样很容易地调用这个对应 Mapper 接口的方法。
### 2.4 作用域(Scope)和生命周期
**SqlSessionFactoryBuilder**这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的最佳作用域是方法作用域（也就是局部方法变量）。
**SqlSessionFactory**一旦被创建就应该在应用的运行期间一直存在，没有任何理由对它进行清除或重建。因此 SqlSessionFactory 的最佳作用域是应用作用域。
**SqlSession**的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。
```
//标准关闭模式
SqlSession session = sqlSessionFactory.openSession();
try {
  // do work
} finally {
  session.close();
}

//关闭映射器示例
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // do work
} finally {
  session.close();
}
```