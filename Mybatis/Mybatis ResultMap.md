[toc]
## 1. 引言
MyBatis 的真正强大在于它的映射语句，SQL 映射文件有很少的几个顶级元素：
```
cache – 给定命名空间的缓存配置。
cache-ref – 其他命名空间缓存配置的引用。
resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
sql – 可被其他语句引用的可重用语句块。
insert – 映射插入语句
update – 映射更新语句
delete – 映射删除语句
select – 映射查询语句
```
## 2. ResultMap
### 2.1 ResultMap元素
```
constructor - 用于在实例化类时，注入结果到构造方法中
  idArg - ID 参数;标记出作为 ID 的结果可以帮助提高整体性能
  arg - 将被注入到构造方法的一个普通结果
id – 一个 ID 结果;标记出作为 ID 的结果可以帮助提高整体性能
result – 注入到字段或 JavaBean 属性的普通结果
association – 一个复杂类型的关联，对应"has-a"关系
collection – 一个复杂类型的集合，对应"has-many"关系
discriminator – 使用结果值来决定使用哪个 resultMap
case – 基于某些值的结果映射，嵌套结果映射 – 一个 case 也是一个映射它本身的结果,因此可以包含很多相同的元素，
或者它可以参照一个外部的 resultMap。
```
### 2.2 has-a 关联
**association元素**用于处理"has-a"关系，比如一个博客有一个用户，关联映射就工作于这种结果之上，Mybatis有两种方式加载关联：
- 嵌套查询:通过执行另外一个 SQL 映射语句来返回预期的复杂类型。
- 嵌套结果:使用嵌套结果映射来处理重复的联合结果的子集。
#### 2.2.1 嵌套查询
```
<resultMap id="blogResult" type="Blog">
  <association property="author" column="author_id" javaType="Author" select="selectAuthor"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectAuthor" resultType="Author">
  SELECT * FROM AUTHOR WHERE ID = #{id}
</select>
```
这里有两个查询语句:一个来加载博客,另外一个来加载作者,而且博客的结果映射描述了“selectAuthor”语句应该被用来加载它的 author 属性。其他所有的属性将会被自动加载,假设它们的列和属性名相匹配，但是这种做法容易引起“N+1 查询问题”。
#### 2.2.2 嵌套结果
```
<select id="selectBlog" resultMap="blogResult">
  select
    B.id            as blog_id,
    B.title         as blog_title,
    B.author_id     as blog_author_id,
    A.id            as author_id,
    A.username      as author_username,
    A.password      as author_password,
    A.email         as author_email,
    A.bio           as author_bio
  from Blog B left outer join Author A on B.author_id = A.id
  where B.id = #{id}
</select>
```
对于这个联合查询，我们可以映射如下结果：
```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <association property="author" column="blog_author_id" javaType="Author" resultMap="authorResult"/>
</resultMap>

<resultMap id="authorResult" type="Author">
  <id property="id" column="author_id"/>
  <result property="username" column="author_username"/>
  <result property="password" column="author_password"/>
  <result property="email" column="author_email"/>
  <result property="bio" column="author_bio"/>
</resultMap>
```
非常重要: id元素在嵌套结果映射中扮演着非常重要的角色。你应该总是指定一个或多个可以唯一标识结果的属性。实际上如果你不指定它的话, MyBatis仍然可以工作,但是会有严重的性能问题。在可以唯一标识结果的情况下, 尽可能少的选择属性，主键是一个显而易见的选择（即使是复合主键）。 
### 2.3 has-many 关系
**collection元素**几乎和关联association是相同的。比如用来表示一个博客只有一个作者，但是博客有很多文章。
```
private List<Post> posts;  //表示文章
```
#### 2.3.1 嵌套查询
使用嵌套查询来为博客加载文章
```
<resultMap id="blogResult" type="Blog">
  <collection property="posts" javaType="ArrayList" column="id" ofType="Post" select="selectPostsForBlog"/>
</resultMap>

<select id="selectBlog" resultMap="blogResult">
  SELECT * FROM BLOG WHERE ID = #{id}
</select>

<select id="selectPostsForBlog" resultType="Post">
  SELECT * FROM POST WHERE BLOG_ID = #{id}
</select>
```
读作: “在 Post 类型的 ArrayList 中的 posts 的集合。” 
#### 2.3.2 嵌套结果
SQL如下：
```
<select id="selectBlog" resultMap="blogResult">
  select
  B.id as blog_id,
  B.title as blog_title,
  B.author_id as blog_author_id,
  P.id as post_id,
  P.subject as post_subject,
  P.body as post_body,
  from Blog B
  left outer join Post P on B.id = P.blog_id
  where B.id = #{id}
</select>
```
映射如下：
```
<resultMap id="blogResult" type="Blog">
  <id property="id" column="blog_id" />
  <result property="title" column="blog_title"/>
  <collection property="posts" ofType="Post" resultMap="blogPostResult" columnPrefix="post_"/>
</resultMap>

<resultMap id="blogPostResult" type="Post">
  <id property="id" column="id"/>
  <result property="subject" column="subject"/>
  <result property="body" column="body"/>
</resultMap>
```