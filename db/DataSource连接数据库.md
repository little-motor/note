[toc]
## 1. 引言
一个连接到DataSource对象所代表的物理数据源的工厂。作为DriverManager的另一种选择，DataSource对象是获得连接的首选方法。
## 2. 实现
作为 DriverManager 工具的替代项，DataSource接口由驱动程序供应商实现。共有三种类型的实现：

1. 基本实现 - 生成标准的 Connection 对象

2. 连接池实现 - 生成自动参与连接池的 Connection 对象。此实现与中间层连接池管理器一起使用。

3. 分布式事务实现 - 生成一个 Connection 对象，该对象可用于分布式事务，大多数情况下总是参与连接池。此实现与中间层事务管理器一起使用，大多数情况下总是与连接池管理器一起使用。
## 3. 方法
DataSource接口包含两个方法，分别为无参和有参getConnection()方法。
```
Connection getConnection()
                         throws SQLException

Attempts to establish a connection with the data source that this DataSource object represents.

Returns:
    a connection to the data source
Throws:
    SQLException - if a database access error occurs
```
```
Connection getConnection(String username,
                       String password)
                         throws SQLException

Attempts to establish a connection with the data source that this DataSource object represents.

Parameters:
    username - the database user on whose behalf the connection is being made
    password - the user's password
Returns:
    a connection to the data source
Throws:
    SQLException - if a database access error occurs
Since:
    1.4
```
 