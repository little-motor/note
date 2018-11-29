[toc]
# 1. 数据库相关
## 1.1 创建数据库
```
create database database_name;
```
## 1.2 选择数据库
```
show databases;           #显示所有的数据库
use database_name;        #使用指定的数据库
```
## 1.3 删除数据库
```
drop database database_name;
```
# 2. 表的操作
## 2.1 创建表
```
create table table_name(
  属性名  数据类型，
  属性名  数据类型
)
```
## 2.2 查看表结构
```
describe table_name;               #查看表定义
show create table table_name;      #查看表详细定义
```
## 2.3 删除表
```
drop table table_name;
```
## 2.4 修改表名
```
alter table old_table_name rename new_table_name 
```
## 2.5 增加字段
```
alter table table_name
  add 属性名 属性类型;         #在表的最后一个位置增加字段

alter table table_name
  add 属性名 属性类型 first;   #在表的第一个位置增加字段

alter table table_name
  add 属性名 属性类型
    after 属性名              #在表的指定字段之后增加增加字段
```
## 2.6 删除字段
```
alter table table_name
  drop 属性名;
```
## 2.7 修改字段
```
alter table table_name
  modify 属性名 数据类型；                     #修改字段数据类型

alter table table_name
  change 旧属性名 新属性名 旧数据类型           #修改字段名称

alter table table_name
  change 旧属性名 新属性名 新数据类型；         #修改字段名称和数据类型
```
# 3. 增删改查
## 3.1 insert
增加新的数据
```
insert into table_name(field1, field2, field3,...)
values(value1,value2,value3...)
```
## 3.2 delete
delete可以删除特定的数据记录或删除所有数据记录
```
delete from table_name
    where condition
```
## 3.3 update
update可以更新特定数据记录或更新所有数据记录
```
update table_name          #更新特定数据
    set field1=value1,
        field2=value2,
        field3=value3,
    where condition;
```
## 3.4 select 
查询操作,支持四则运算(+,-,*,/,%)
```
select field1, field2... from table_name;
```
### 2.4.1避免重复查询
```
select distinct field1, field2... from table_name;
```
### 2.4.2 as修改字段名
```
select field1 as name1, field2 as name2... from table_name;
```
### 2.4.3 条件查询
条件查询语句包含如下功能:
- 带关系运算符和逻辑运算符的条件查询

比较运算符 | 描述
---------|----------|---------
> | 
 < | 
= | 
!=(<>) | 
>=|
<=|

逻辑运算符 | 描述
---------|----------
 and(&&)| 
 or(ll)|
 xor | 逻辑异或
 not(!) | 逻辑非

- 带between and关键字条件查询

判定字段数值在指定范围的条件查询
```
select field1,field2...
    from table_name
        where fidle between value1 and value2;
```
- 带is null关键字条件查询
- 带in关键字条件查询

判断数值在指定集合的条件查询
```
select field1,field2...
    from table_name
        where field in(value1,value2...);
```
- 带like关键字条件查询

like可以实现模糊查询,通过字符串和通配符组合的方式查询字段,字符串必须加上单引号或双引号,"_"匹配单个字符,"%"匹配任意长度字符
```
select field1, fidld2...
    frome table_name
        where field [not] like value; 
```
### 2.4.4 排序
可以按照单字段或者多字段排序,多字段排序的过程为首先按照第一个字段进行排序,如果遇到值相同的字段则会按照第二个字段进行排序
```
select field1, fidld2...
    from table_name
        where condition
            order by field1 [asc|desc][,field2[asc|desc]];
```
### 2.4.5 限制查询数量
```
select field1, field2...
    from table_name
        where condition
            limit offset_start,row_count;    #offset_start代表初始值,row_count代表行数
```
limit经常与order by配合使用,即先对查询结果进行排序,然后显示其中部分数据
```
select field1, field2...
    from table_name
        where condition
            order by field limit offset_start,row_count;
```
### 2.4.6 统计函数
- count():统计表中记录的条数
- avg():统计字段的平均值
- sum():统计字段的总和
- max():查询字段值的最大值
- min():查询字段值的最小值
```
select function(field)
    from table_name
        where condition;
```
### 2.4.7 分组查询
分组所依据的字段值一定要具有重复性,否则没有任何意义,建议与统计函数一起使用
```
select function()
    from table_name
        where condition
            group by field;  #根据field中的字段分组,并显示每组中的一条数据
```
function可以使用
```
group_concat(field)  #合并同组的所有字段并用,分隔后显示
count(field)  #显示字段数量
```
多个字段分组查询的过程是首先按照字段field1进行分组,然后针对每组按照字段field2进行分组
```
select group_concat(field),function(field)
    from table_name
        where condition
            group by field1,field2...;
```
HAVING字句限定分组查询
因为where主要是用来实现条件限制数据记录,在条件限制分组记录中要使用having
```
select function(field)
    from table_name
        where condition
        group by field1,field2...
        having condition;
```

# 4. 触发器
数据库对象触发器(trigger)是用来实现由一些表事件触发的某个操作,他所响应的事件有：
- delete
- update
- insert
## 4.1 创建有一条执行语句的触发器
建议触发器命名为trigger_xxx,before和after指定了触发器执行时间，trigger_event表示触发事件有delete、insert、update语句，for each row表示任何一条记录上的操作满足触发条件都会触发该触发器。
```
create trigger trigger_name
    before|after trigger_event
        on table_name for each row trigger_stmt
```
## 4.2 创建有多条执行语句的触发器
需要注意的是多条语句需要用;隔开，所以在触发器前后用delimiter将结束符暂时设置为$$，在结束后再修改为;
```
delimiter $$
create trigger trigger_name
    before|after trigger_event
        on table_name for each row 
            begin
                stmt1;
                stmt2;
                ...;
            end
        $$
delimiter ;
```
## 4.3 NEW和OLD
MySQL 中定义了 NEW 和 OLD，用来表示触发器的所在表中，触发了触发器的那一行数据，来引用触发器中发生变化的记录内容，具体地：
- 在INSERT型触发器中，NEW用来表示将要（BEFORE）或已经（AFTER）插入的新数据；
- 在UPDATE型触发器中，OLD用来表示将要或已经被修改的原数据，NEW用来表示将要或已经修改为的新数据；
- 在DELETE型触发器中，OLD用来表示将要或已经被删除的原数据；

使用方法：NEW.columnName（columnName为相应数据表某一列名）另外，OLD是只读的，而NEW则可以在触发器中使用 SET 赋值，这样不会再次触发触发器，造成循环调用。


