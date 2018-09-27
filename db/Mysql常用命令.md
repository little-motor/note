[toc]
## 1. 数据库相关
### 1.1 创建数据库
```
create database database_name;
```
### 1.2 选择数据库
```
show databases;           #显示所有的数据库
use database_name;        #使用指定的数据库
```
### 1.3 删除数据库
```
drop database database_name;
```
## 2. 表的操作
### 2.1 创建表
```
create table table_name(
  属性名  数据类型，
  属性名  数据类型
)
```
### 2.2 查看表结构
```
describe table_name;               #查看表定义
show create table table_name;      #查看表详细定义
```
### 2.3 删除表
```
drop table table_name;
```
### 2.4 修改表名
```
alter table old_table_name rename new_table_name 
```
### 2.5 增加字段
```
alter table table_name
  add 属性名 属性类型;         #在表的最后一个位置增加字段

alter table table_name
  add 属性名 属性类型 first;   #在表的第一个位置增加字段

alter table table_name
  add 属性名 属性类型
    after 属性名              #在表的指定字段之后增加增加字段
```
### 2.6 删除字段
```
alter table table_name
  drop 属性名;
```
### 2.7 修改字段
```
alter table table_name
  modify 属性名 数据类型；                     #修改字段数据类型

alter table table_name
  change 旧属性名 新属性名 旧数据类型           #修改字段名称

alter table table_name
  change 旧属性名 新属性名 新数据类型；         #修改字段名称和数据类型
```

