

# work diary







# Mybatis

### 什么是 MyBatis ？

```doc
MyBatis 是一款优秀的持久层框架。
它支持定制化 SQL、存储过程以及高级映射。
MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。
MyBatis 本是apache的一个开源项目iBatis。
2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。
2013年11月迁移到Github。
iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。
```

### 导入Jar

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

### 从 XML 中构建 SqlSessionFactory

```doc
每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。
SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。
而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。
从 XML 文件中构建 SqlSessionFactory 的实例非常简单，建议使用类路径下的资源文件进行配置。
包括字符串形式的文件路径或者 file:// 的 URL 形式的文件路径来配置。

```

MyBatis 包含一个名叫 Resources 的工具类，它包含一些实用方法，可使从 classpath 或其他位置加载资源文件更加容易。

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

XML 配置文件（configuration XML）中包含了对 MyBatis 系统的核心设置，

包含获取数据库连接实例的数据源（DataSource）和决定事务作用域和控制方式的事务管理器（TransactionManager）。



XML 配置文件的详细内容后面再探讨，这里先给出一个简单的示例：

```xml
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

要注意 XML 头部的声明，用来验证 XML 文档正确性。

environment 元素体中包含了事务管理和连接池的配置。

mappers 元素则是包含一组 mapper 映射器（这些 mapper 的 XML 文件包含了 SQL 代码和映射定义信息）。



### 不使用 XML 构建 SqlSessionFactory

```java
// 如果你更愿意直接从 Java 程序而不是 XML 文件中创建 configuration，或者创建你自己的 configuration 构建器，MyBatis 也提供了完整的配置类，提供所有和 XML 文件相同功能的配置项。
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
// 注意该例中，configuration 添加了一个映射器类（mapper class）。映射器类是 Java 类，它们包含 SQL 映射语句的注解从而避免了 XML 文件的依赖。不过，由于 Java 注解的一些限制加之某些 MyBatis 映射的复杂性，XML 映射对于大多数高级映射（比如：嵌套 Join 映射）来说仍然是必须的。有鉴于此，如果存在一个对等的 XML 配置文件的话，MyBatis 会自动查找并加载它（这种情况下， BlogMapper.xml 将会基于类路径和 BlogMapper.class 的类名被加载进来）。
```



### 从 SqlSessionFactory 中获取 SqlSession

```java
//既然有了 SqlSessionFactory ，顾名思义，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。例如：

SqlSession session = sqlSessionFactory.openSession();
try {
  Blog blog = (Blog) session.selectOne("org.mybatis.BlogMapper.selectBlog", 101);
} finally {
  session.close();
}

//诚然这种方式能够正常工作，并且对于使用旧版本 MyBatis 的用户来说也比较熟悉，不过现在有了一种更直白的方式。使用对于给定语句能够合理描述参数和返回值的接口（比如说BlogMapper.class），你现在不但可以执行更清晰和类型安全的代码，而且还不用担心易错的字符串字面值以及强制类型转换。
  
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}


```

### XML 映射配置文件

```doc
MyBatis 的配置文件
 	configuration 配置
 	properties 属性
    settings 设置
    typeAliases 类型别名
    typeHandlers 类型处理器
    objectFactory 对象工厂
    plugins 插件
    environments 环境
    	environment 环境变量
    		transactionManager 事务管理器
    		dataSource 数据源
    databaseIdProvider 数据库厂商标识
    mappers 映射器
     
```

#### 1.properties

这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。

```
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

```
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

这个例子中的 username 和 password 将会由 properties 元素中设置的相应值来替换。 driver 和 url 属性将会由 config.properties 文件中对应的值来替换。这样就为配置提供了诸多灵活选择。

属性也可以被传递到 SqlSessionFactoryBuilder.build()方法中。

```
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, props);

// ... or ...

SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, props);
```

如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：

- 在 properties 元素体内指定的属性首先被读取。
- 然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
- 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。

因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。

#### 2.settings

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为.

一个配置完整的 settings 元素的示例如下：

```
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

#### 3.typeAliases

类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。例如:

```
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

当这样配置时，Blog可以用在任何使用domain.blog.Blog的地方。

也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如:

```
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

每一个在包 domain.blog 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 domain.blog.Author 的别名为 author；若有注解，则别名为其注解值。看下面的例子：

```
@Alias("author")
public class Author {
    ...
}
```

#### 4.typeHandlers

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

#### 5.映射器（mappers）

既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要定义 SQL 映射语句了。但是首先我们需要告诉 MyBatis 到哪里去找到这些语句。 Java 在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉 MyBatis 到哪里去找映射文件。你可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 file:/// 的 URL），或类名和包名等。例如：

```
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

```
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```





### Mapper XML 文件

MyBatis 的真正强大在于它的映射语句，也是它的魔力所在。

SQL 映射文件有很少的几个顶级元素（按照它们应该被定义的顺序）：

- cache – 给定命名空间的缓存配置。
- cache-ref – 其他命名空间缓存配置的引用。
- resultMap – 是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。
- sql – 可被其他语句引用的可重用语句块。
- insert – 映射插入语句
- update – 映射更新语句
- delete – 映射删除语句
- select – 映射查询语句



#### select

查询语句是 MyBatis 中最常用的元素之一，光能把数据存到数据库中价值并不大，如果还能重新取出来才有用，多数应用也都是查询比修改要频繁。对每个插入、更新或删除操作，通常对应多个查询操作。这是 MyBatis 的基本原则之一，也是将焦点和努力放到查询和结果映射的原因。简单查询的 select 元素是非常简单的。比如：

```
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句被称作 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。

注意参数符号：

```
#{id}
```

这就告诉 MyBatis 创建一个预处理语句参数，通过 JDBC，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：

```
// Similar JDBC code, NOT MyBatis…
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

当然，这需要很多单独的 JDBC 的代码来提取结果并将它们映射到对象实例中，这就是 MyBatis 节省你时间的地方。我们需要深入了解参数和结果映射，细节部分我们下面来了解。



select 元素有很多属性允许你配置，来决定每条语句的作用细节。

```xml
<select
  id="selectPerson"
  parameterType="int"
  parameterMap="deprecated"
  resultType="hashmap"
  resultMap="personResultMap"
  flushCache="false"
  useCache="true"
  timeout="10000"
  fetchSize="256"
  statementType="PREPARED"
  resultSetType="FORWARD_ONLY">
```

| 属性            | 描述                                       |
| ------------- | ---------------------------------------- |
| id            | 在命名空间中唯一的标识符，可以被用来引用这条语句。                |
| parameterType | 将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。 |
| parameterMap  | 这是引用外部 parameterMap 的已经被废弃的方法。使用内联参数映射和 parameterType 属性。 |
| resultType    | 从这条语句中返回的期望类型的类的完全限定名或别名。注意如果是集合情形，那应该是集合可以包含的类型，而不能是集合本身。使用 resultType 或 resultMap，但不能同时使用。 |
| resultMap     | 外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，对其有一个很好的理解的话，许多复杂映射的情形都能迎刃而解。使用 resultMap 或 resultType，但不能同时使用。 |
| flushCache    | 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false。 |
| useCache      | 将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true。 |
| timeout       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 |
| fetchSize     | 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动）。 |
| statementType | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| resultSetType | FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）。 |
| databaseId    | 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |
| resultOrdered | 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：false。 |
| resultSets    | 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。 |



#### insert, update 和 delete

数据变更语句 insert，update 和 delete 的实现非常接近：

```xml
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

```

| 属性               | 描述                                       |
| ---------------- | ---------------------------------------- |
| id               | 命名空间中的唯一标识符，可被用来代表这条语句。                  |
| parameterType    | 将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。 |
| parameterMap     | 这是引用外部 parameterMap 的已经被废弃的方法。使用内联参数映射和 parameterType 属性。 |
| flushCache       | 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句）。 |
| timeout          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 |
| statementType    | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| useGeneratedKeys | （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。 |
| keyProperty      | （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn        | （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| databaseId       | 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |

下面就是 insert，update 和 delete 语句的示例：

```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>
```

如前所述，插入语句的配置规则更加丰富，在插入语句里面有一些额外的属性和子元素用来**处理主键的生成**，而且有多种生成方式。

首先，如果你的**数据库支持自动生成主键的字段**（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上就OK了。例如，如果上面的 Author 表已经对 id 使用了自动生成的列类型，那么语句可以修改为:

```
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```



如果你的数据库还支持多行插入, 你也可以传入一个Authors数组或集合，并返回自动生成的主键。

```
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```

对于不支持自动生成类型的数据库或可能不支持自动生成主键的 JDBC 驱动，MyBatis 有另外一种方法来生成主键。

这里有一个简单（甚至很傻）的示例，它可以生成一个随机 ID（你最好不要这么做，但这里展示了 MyBatis 处理问题的灵活性及其所关心的广度）：

```
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```



在上面的示例中，selectKey 元素将会首先运行，Author 的 id 会被设置，然后插入语句会被调用。这给你了一个和数据库中来处理自动生成的主键类似的行为，避免了使 Java 代码变得复杂。

selectKey 元素描述如下：

```
<selectKey
  keyProperty="id"
  resultType="int"
  order="BEFORE"
  statementType="PREPARED">
```

| 属性            | 描述                                       |
| ------------- | ---------------------------------------- |
| keyProperty   | selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn     | 匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| resultType    | 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 |
| order         | 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 |
| statementType | 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 |



#### sql

这个元素可以被用来定义可重用的 SQL 代码段，可以包含在其他语句中。它可以被静态地(在加载参数) 参数化. 不同的属性值通过包含的实例变化. 比如：

```
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```



这个 SQL 片段可以被包含在其他语句中，例如：

```
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```



### 参数（Parameters）

前面的所有语句中你所见到的都是简单参数的例子，实际上参数是 MyBatis 非常强大的元素，对于简单的做法，大概 90% 的情况参数都很少，比如：

```
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>
```



















## Spring 朱哥哥带你们看看

Spring是一个开源框架

一个轻量级的Java开发框架

**Spring的核心是控制反转(IoC)和面向切面(AOP)。简单来说，Spring是一个分层的JavaSE/EEfull-stack(一站式)轻量级开源框架**。

为什么说Spring是一个一站式的轻量级开源框架呢？EE开发可分成三层架构，针对JavaEE的三层结构，每一层Spring都提供了不同的解决技术。

WEB层：SpringMVC
业务层：Spring的IoC
持久层：Spring的JDBCTemplate(Spring的JDBC模板，ORM模板用于整合其他的持久层框架)

Spring的核心有两部分：

- IoC：控制反转。 
  举例来说，在之前的操作中，比方说有一个类，我们想要调用类里面的方法(不是静态方法)，就要创建类的对象，使用对象调用方法实现。对于Spring来说，Spring创建对象的过程，不是在代码里面实现的，而是交给Spring来进行配置实现的。
- AOP：面向切面编程。 

Spring的优点：

方便解耦，简化开发。 
Spring就是一个大工厂，可以将所有对象的创建和依赖关系的维护，交给Spring管理。
AOP编程的支持 
Spring提供面向切面编程，可以方便的实现对程序进行权限拦截、运行监控等功能。
声明式事务的支持 
只需要通过配置就可以完成对事务的管理，而无须手动编程。
方便程序的测试 
Spring对Junit4支持，可以通过注解方便的测试Spring程序。
方便集成各种优秀的框架 
Spring不排斥各种优秀的开源框架，其内部提供了对各种优秀框架(如：Struts2、Hibernate、MyBatis、Quartz等)的直接支持。
降低JavaEE API的使用难度 
Spring对JavaEE开发中非常难用的一些API(JDBC、JavaMail、远程调用等)，都提供了封装，使这些API应用难度大大降低。

### 1.IOC的底层实现原理

IOC：Inversion of Control，控制反转。指的是对象的创建权反转(交给)给Spring，其作用是实现了程序的解耦合。也可这样解释：获取对象的方式变了。对象创建的控制权不是“使用者”，而是“框架”或者“容器”。 

IOC就是指对象的创建，并不是在代码中用new操作new出来的，而是通过Spring进行配置创建的。其底层实现原理是XML配置文件+SAX解析+工厂设计模式。 
就拿持久层(也即dao(data access object，数据访问对象)层)的开发来说，官方推荐做法是先创建一个接口，然后再创建接口对应的实现类。 

例：

```java
// 先创建一个Userdao接口
public interface UserDao {
    public void add();
}
// 再创建Userdao接口的UserDaoImpl实现类
public class UserDaoImpl implements UserDao {
    public void add() {
        balabala......
    }
}
```



接着我们在service层调用dao层，核心代码如下：

```java
// 接口 实例变量 = new 实现类
UserDao dao = new UserDaoImpl();
dao.add();
```

可发现缺点：service层和dao层耦合度太高了。解决方法是使用工厂模式进行解耦合操作。 
创建一个工厂类，在工厂类中提供一个方法，返回实现类的对象。

```java
public class Factory {
    // 提供返回实现类对象的方法
    public static UserDao getUserDaoImpl() {
        return new UserDaoImpl();
    }
}
```

然后在service层调用dao层的核心代码就变为：

```java
UserDao dao = Factory.getUserDaoImpl();
dao.add();
```

如若这样做，会发现又产生了一个缺点：service层和工厂类又耦合了。所以使用工厂模式进行解耦合也只是一种权宜之计。下面我就来简单讲讲Spring IOC的底层实现原理：

配置文件中可能会有如下配置信息：

<bean id="userDaoImpl" class="cn.itcast.dao.impl.UserDaoImpl" />

也是要创建一个工厂类，在工厂类中提供一个返回实现类对象的方法，但并不是直接new实现类，而是使用SAX解析配置文件，根据标签bean中的id属性值得到对应的class属性值，使用反射创建实现类对象。

```java
public class Factory {
    public static UserDao getUserDaoImpl() {
        // 1.使用SAX解析得到配置文件内容
        // 直接根据id值userDaoImpl得到class属性值
        String classvalue = "class属性值";
        // 2.使用反射得到对象
        Class clazz = Class.forName(classvalue);
        UserDaoImpl userDaoImpl = (UserDaoImpl)lazz.newInstance();
        return userDaoImpl;
    }
}
```

### 2.面向对象设计的七大原则

1. 单一职责原则（Single Responsibility Principle）：每一个类应该专注于做一件事情。
2. 里氏替换原则（Liskov Substitution Principle）：超类存在的地方，子类是可以替换的。
3. 依赖倒置原则（Dependence Inversion Principle）：实现尽量依赖抽象，不依赖具体实现。
4. 接口隔离原则（Interface Segregation Principle）：应当为客户端提供尽可能小的单独的接口，而不是提供大的总的接口。
5. 迪米特法则（Law Of Demeter）：又叫最少知识原则，一个软件实体应当尽可能少的与其他实体发生相互作用。
6. 开闭原则（Open Close Principle）：面向扩展开放，面向修改关闭。
7. 组合/聚合复用原则（Composite/Aggregate Reuse Principle CARP）：尽量使用组合/聚合达到复用，尽量少用继承。原则： 一个类中有另一个类的对象。

### 3.Spring的IOC入门

#### 步骤一：下载Spring的开发包

Spring的官网是[http://spring.io](http://spring.io/)。Spring的开发包的下载地址是[http://repo.springsource.org/libs-release-local/org/springframework/spring](http://repo.springsource.org/libs-release-local/org/springframework/spring)，上面说过，我下载的是spring-framework-4.2.4.RELEASE。解压缩之后，可发现Spring开发包的目录结构如下： 

#### 步骤二：创建Web项目，引入Spring的开发包

由于我们只是初次入门Spring，所以也只是使用Spring的基本功能，所以需要使用到下面的这4个Jar包：

spring-beans-4.2.4.RELEASE.jar

spring-context-4.2.4.RELEASE.jar

spring-core-4.2.4.RELEASE.jar

spring-expression-4.2.4.RELEASE.jar

除此之外，还要导入Spring支持的日志jar包

commons-logging-1.2.jar

log4j.jar

#### 步骤三：编写相关的类，在类中创建方法

在src目录下创建一个cn.itcast.ioc包，并在该包下创建一个User类。

```java
public class User {
    public void add() {
        System.out.println("add.................");
    }
}
```

#### 步骤三：创建Spring配置文件

注意：Spring配置文件的名称和位置没有固定要求，一般建议把该文件放到src下面，名称可随便写，官方建议写成applicationContext.xml。但我觉得这个文件名称太长了，所以决定写为bean1.xml。然后我们还需要在配置文件中引入约束，Spring学习阶段的约束是schema约束。那么问题来了，这个约束又该怎么写呢？可参考docs\spring-framework-reference\html目录下的xsd-configuration.html文件，在其内容最后找到如下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

#### 步骤四：在配置文件中配置对象的创建

<!-- 1.配置user对象的创建 --> 
<bean id="user" class="cn.itcast.ioc.User"></bean>

#### 步骤五：编写测试程序

我们要在Spring中写代码来实现获取bean1.xml文件中配置的对象（该段代码不要求重点掌握，只是用在测试中而已）。这段代码主要用来解析Spring配置文件得到对象，但这个过程不需要我们写代码实现，Spring封装了一个对象帮我们进行了这些操作，这个对象叫ApplicationContext，它就能实现这个功能。 
在cn.itcast.ioc包下创建一个TestIOC单元测试类，如下：

```java
public class TestIOC {

    // 得到配置的user对象
    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml"); // classpath：类路径，src目录下的文件最终要编译到类路径下
        // 2.根据配置文件的id得到user对象
        User user = (User) context.getBean("user");
        System.out.println(user);
        user.add();
    }
}
```

**注意：classpath为类路径，src目录下的文件最终要编译到类路径下。**









### 4.Spring的bean管理

通俗一点说，Spring的bean管理即指创建对象时不需要new操作代码实现，而是交给Spring进行配置完成。 
Spring进行bean管理有两种方式：

***使用配置文件方式实现***
***使用注解方式实现***

Spring实例化bean的三种方式

#### **使用无参构造（重点）**

创建对象时候，调用类里面的无参数的构造方法实现。那么Spring配置文件中又该怎样写呢？基本类似于如下写法：

```xml
<!-- 1.配置user对象的创建 --> 
<bean id="user" class="cn.itcast.ioc.User"></bean>
```

#### 使用静态工厂（了解）

创建一个工厂类，在工厂类中提供一个静态的方法，这个方法返回类的对象；调用工厂类的方法时候，直接使用类名.方法名称即可以调用。下面举例来说明。 
在src目录下创建一个cn.itcast.bean包，并在该包下创建一个Bean1类。

```java
public class Bean1 {

    public void bean1() {
        System.out.println("bean1..........");
    }

}

```

然后在该包下创建一个Bean1Factory工厂类。

```java
public class Bean1Factory {

    // 静态方法
    public static Bean1 getBean1() {
        return new Bean1();
    }

}
```

接着Spring配置文件中应向下面这样配置：

```xml
<!-- 2.使用静态工厂创建对象 -->
<bean id="bean1" class="cn.itcast.bean.Bean1Factory" factory-method="getBean1"></bean>
```

最后在该包下创建一个TestIOC单元测试类。

```java
public class TestIOC {

    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");
        // 2.根据配置文件的id得到user对象
        Bean1 bean1 = (Bean1) context.getBean("bean1");
        System.out.println(bean1);
    }
}
```

#### 使用实例工厂（了解）

创建一个工厂类，在工厂类里面提供一个普通的方法，这个方法返回类对象；调用工厂类的方法时候，创建工厂类对象，使用对象调用方法即可。下面也举例来说明。 
在src目录下的cn.itcast.bean包下创建一个Bean2类。

```java
public class Bean2 {

    public void bean2() {
        System.out.println("bean2..........");
    }

}
```

然后在该包下创建一个Bean2Factory工厂类。

```java
public class Bean2Factory {

    public Bean2 getBean2() {
        return new Bean2();
    }

}
```



接着Spring配置文件中应向下面这样配置：

```xml
<!-- 3.使用实例工厂创建对象 -->
<!-- 3.1先创建工厂对象 -->
<bean id="bean2Factory" class="cn.itcast.bean.Bean2Factory"></bean>
<!-- 3.2再使用工厂对象创建bean2对象 -->
<bean id="bean2" factory-bean="bean2Factory" factory-method="getBean2"></bean>
```

最后将TestIOC单元测试类的代码修改为：

```java
public class TestIOC {

    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");
        // 2.根据配置文件的id得到user对象
        Bean2 bean2 = (Bean2) context.getBean("bean2");
        System.out.println(bean2);
    }
}
```

### 5.Spring配置文件中bean标签常用的属性

Spring配置文件中bean标签常用的属性有以下四种

#### id 

属性：根据id属性值得到配置对象。

在Spring配置文件中会有多个bean标签，但它们的id属性值是不能相同的。Bean起名字时，在约束中采用的是ID约束——唯一，而且名字必须以字母开始，可以使用字母、数字、连字符、下划线、句号、冒号等，但id属性值不能有特殊符号。

#### class

属性：要创建对象的类的全路径。

#### scope

属性：bean的作用范围。 
scope属性共有以下5个属性：

- singleton：创建的对象是单例的，也是scope属性的默认值。 



举例

```java
public class TestIOC {

    // 得到配置的user对象
    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");
        // 2.根据配置文件的id得到user对象
        User user1 = (User) context.getBean("user");
        User user2 = (User) context.getBean("user");
        System.out.println(user1);
        System.out.println(user2);
    }
}
```

单元测试以上方法，一切就尽在不言中。其实，此时Spring配置文件中有关如下bean的配置：

<bean id="user" class="cn.itcast.ioc.User"></bean>

就相当于：

<bean id="user" class="cn.itcast.ioc.User" scope="singleton"></bean>

prototype：创建的对象是多实例的。 
也可举例来说明它。将Spring配置文件中有关如下bean的配置：

<bean id="user" class="cn.itcast.ioc.User"></bean>

修改为：

<bean id="user" class="cn.itcast.ioc.User" scope="prototype"></bean>

测试单元测试类的方法就能明白了。

globalSession：用在单点登录(即SSO，single sign on)上。

#### name

属性：name属性的功能和id属性是一样的。name属性和id属性区别是：在id属性值里面不能有特殊符号，在name属性值里面可以添加特殊符号。



### 6.bean的生命周期的配置

通过配置<bean>标签上的init-method作为bean的初始化的时候执行的方法，配置destroy-method作为bean的销毁的时候执行的方法。销毁方法想要执行，需要是单例创建的Bean而且在工厂关闭的时候，Bean才会被销毁。

### 7.Spring中Bean的属性注入



Bean的属性注入共有三种方式

#### set方法注入

```java
public class Book {

    private String bookname;

    public void setBookname(String bookname) {
        this.bookname = bookname;
    }

}

Book book = new Book();
book.setBookName("Java编程思想");
```

#### 有参数构造注入 

用代码可表示如下：

```java
public class Book {

    private String bookname;

    public Book(String bookname) {
        this.bookname = bookname;
    }

}

Book book = new Book("代码大全");
```

#### 接口注入

先编写一个接口：

```java
public interface Dao {
    public void add(String name);
}
```



再编写这个接口的实现类：

```java
public class DaoImpl implements Dao {
    private String name;

    public void add(String name) {
        this.name = name;
    }
}
```

但在Spring框架里面，只支持前两种方式，即set方法注入和有参数构造注入。

**构造方法的方式注入属性**
在src目录下创建一个cn.itcast.property包，并在该包下编写一个Book实体类。

```java
public class Book {

    private String bookname;

    public Book(String bookname) {
        this.bookname = bookname;
    }

    public void testBook() {
        System.out.println("book.............." + bookname);
    }

}
```

接着在Spring配置文件中对以上JavaBean添加如下配置：

```xml
<!-- 4.使用有参数的构造注入属性 -->
<bean id="book" class="cn.itcast.property.Book">
    <!-- 使用标签，name：为属性的名字；value：为属性的值 -->
    <constructor-arg name="bookname" value="beautifulMan_美美侠"></constructor-arg>
</bean>
```

最后在该包下编写一个TestIOC单元测试类：

```java
public class TestIOC {

    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");

        Book book = (Book) context.getBean("book");
        book.testBook();
    }
}
```

**set方法的方式注入属性**

我们同样在cn.itcast.property包下编写一个Person实体类，在类中定义属性，并生成set方法。

```java
public class Person {

    // 1.定义一个属性
    private String username;

    // 2.生成这个属性的set方法
    public void setUsername(String username) {
        this.username = username;
    }

    public void testperson() {
        System.out.println("person.............." + username);
    }

}
```



然后在Spring配置文件中，使用bean标签创建对象，在bean标签里面使用property标签注入属性。即在Spring配置文件中对以上JavaBean添加如下配置，

```xml
<!-- 5.使用set方法进行注入属性 -->
<bean id="person" class="cn.itcast.property.Person">
    <!--
        使用property标签注入属性值
        name：类属性名称
        value属性：往属性中注入的值
    -->
    <property name="username" value="李阿昀"></property>
</bean>
```

最后将TestIOC单元测试类的代码修改为：

```java
public class TestIOC {

    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");

        Person person = (Person)context.getBean("person");
        person.testperson();
    }
}
```

**Spring的属性注入：对象类型的注入**
在实际开发中，我们要提交表单到action里面去，然后在action里面调用service层的方法，接着在service层里面调用dao层的方法。在这里，我假设在service层里面调用dao层的方法，所以需要在servcie层里面创建dao层的对象实现调用。 

先在src目录下创建一个cn.itcast.dao包，并在该包下编写一个UserDao类。

```JAVA
public class UserDao {

    public void add() {
        System.out.println("dao................");
    }
}
```

然后在src目录下再创建一个cn.itcast.service包，并在该包下编写一个UserService类。

```JAVA
public class UserService {

    public void add() {
        System.out.println("service.........");
        // 调用dao
        balabala......
    }

}
```

如果我们使用最原始的方式在service层里面调用dao层的方法，那么UserService类中add()方法应这样写：

```java
public void add() {
    System.out.println("service.........");
    // 原始方式，调用dao
    UserDao dao = new UserDao();
    dao.add();
}
```



在Spring里面，我们就应该这么玩了。我们的最终目的是在service层里面得到dao层对象。所以步骤为：





第一步，让dao作为service的一个属性。

```java
// 1.让dao作为service的一个属性
private UserDao userDao;
```



第二步，生成dao属性的set方法。

```java
// 2.生成dao属性的set方法
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}
```

这时，UserService类的代码就变成：

```java
public class UserService {

    // 1.让dao作为service的一个属性
    private UserDao userDao;

    // 2.生成dao属性的set方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add() {
        System.out.println("service.........");
        userDao.add();
    }

}
```





第三步，在Spring配置文件中进行配置和注入。

```xml
<!-- 6.注入对象的属性 -->
<!-- 6.1先创建dao对象 -->
<bean id="userDao" class="cn.itcast.dao.UserDao"></bean>
<!-- 6.2再创建service对象    -->
<bean id="userService" class="cn.itcast.service.UserService">
    <!-- 在servcie里面注入userDao属性
        name属性：service对象里面的userDao属性的名称
        注入dao对象，不能写value属性，要写ref属性：dao配置的bean的id值
    -->
    <property name="userDao" ref="userDao"></property>
</bean>
```

#### *名称空间p的属性注入方式——Spring2.x版本后提供的方式*

在src目录下创建一个cn.itcast.propertydemo包，并在该包下编写一个Orders实体类。

```java
public class Orders {

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
    }

    public void testorders() {
        System.out.println("orders................" + oname);
    }
}
```





接下来我们需要在Spring核心配置文件中的schema约束位置定义p名称空间。



xmlns:p="http://www.springframework.org/schema/p"



紧接着，我们需要在Spring核心配置文件中添加如下配置：

```xml
<!-- 7.p(property，属性)名称空间的注入 -->
<bean id="orders" class="cn.itcast.propertydemo.Orders" p:oname="gogogo"></bean>
```



最后将cn.itcast.property包下的TestIOC单元测试类的代码修改为：

```java
public class TestIOC {
    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");

        Orders orders = (Orders)context.getBean("orders");
        orders.testorders();
    }
}
```



结论——使用p名称空间：

普通属性：p:属性名称=”…”
对象类型的属性：p:属性名称-ref=”…”

### 8.注入复杂属性

#### 注入数组类型的属性

将cn.itcast.propertydemo包下的Orders类的代码修改为：



```java
public class Orders {

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
    }

    // 1.数组类型的属性
    private String[] arrays;
    public void setArrays(String[] arrays) {
        this.arrays = arrays;
    }

    public void testorders() {
        // System.out.println("orders................" + oname);
        System.out.println("数组：" + Arrays.toString(arrays));
    }
}
```

然后需要在Spring核心配置文件中添加如下配置：

```xml
<bean id="orders" class="cn.itcast.propertydemo.Orders"> <!-- 创建对象 -->
    <!-- 数组类型 -->
    <property name="arrays"> <!-- 注入属性 -->
        <list>
            <value>叶子</value>
            <value>liayun</value>
            <value>杰哥</value>
        </list>
    </property>
</bean>
```

最后将cn.itcast.property包下的TestIOC单元测试类的代码修改为：

```java
public class TestIOC {
    @Test
    public void demo1() {
        // 1.加载Spring配置文件，把配置文件中的对象进行创建
        ApplicationContext context = 
                new ClassPathXmlApplicationContext("bean1.xml");
        Orders orders = (Orders)context.getBean("orders");
        orders.testorders();
    }
}
```

#### 注入List集合类型的属性

将cn.itcast.propertydemo包下的Orders类的代码修改为：

public class Orders {

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
    }
    
    // 2.list类型的属性
    private List<String> list;
    public void setList(List<String> list) {
        this.list = list;
    }
    
    public void testorders() {
        // System.out.println("orders................" + oname);
        System.out.println("list：" + list);
    }
}



然后需要在Spring核心配置文件中添加如下配置：

```xml
<bean id="orders" class="cn.itcast.propertydemo.Orders"> <!-- 创建对象 -->
    <!-- list类型 -->
    <property name="list">
        <list>
            <value>叶子</value>
            <value>李昀玲</value>
            <value>杰哥</value>
        </list>
    </property>
</bean>
```

#### 注入Set集合类型的属性

```java
public class Orders {

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
    }

    // 3.set类型的属性
    private Set<String> keyset;
    public void setKeyset(Set<String> keyset) {
        this.keyset = keyset;
    }

    public void testorders() {
        // System.out.println("orders................" + oname);
        System.out.println("set：" + keyset);
    }
}
```

```xml
<bean id="orders" class="cn.itcast.propertydemo.Orders"> <!-- 创建对象 -->
    <!-- set类型 -->
    <property name="keyset">
        <set>
            <value>蝙蝠侠</value>
            <value>钢铁侠</value>
            <value>美美侠</value>
        </set>
    </property>
</bean>
```



其实，以上配置也可以写为：

```xml
<bean id="orders" class="cn.itcast.propertydemo.Orders"> <!-- 创建对象 -->
    <!-- set类型 -->
    <property name="keyset">
        <list>
            <value>蝙蝠侠</value>
            <value>钢铁侠</value>
            <value>美美侠</value>
        </list>
    </property>
</bean>
```

#### 注入Map集合类型的属性

```java
public class Orders {

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
    }

    // 4.map类型
    private Map<String, String> map;
    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public void testorders() {
        // System.out.println("orders................" + oname);
        System.out.println("map：" + map);
    }
}
```

```xml
<bean id="orders" class="cn.itcast.propertydemo.Orders"> <!-- 创建对象 -->
    <!-- map类型 -->
    <property name="map">
        <map>
            <entry key="username" value="潘金莲"></entry>
            <entry key="password" value="1314"></entry>
            <entry key="address" value="明初"></entry>
        </map>
    </property>
</bean>
```

#### 注入Properties类型的属性

````java
public class Orders {

    private String oname;

    public void setOname(String oname) {
        this.oname = oname;
    }

    // 5.properties
    private Properties properties;
    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    public void testorders() {
        // System.out.println("orders................" + oname);
        System.out.println("properties：" + properties);
    }
}
````



然后需要在Spring核心配置文件中添加如下配置：

````xml
<bean id="orders" class="cn.itcast.propertydemo.Orders"> <!-- 创建对象 -->
    <!-- properties类型 -->
    <property name="properties">
        <props>
            <prop key="name">宫本一郎</prop>
            <prop key="address">日本</prop>
        </props>
    </property>
</bean>
````

## IoC和DI的区别

```dockerfile
IoC：控制反转，即把对象的创建交给Spring进行管理。所以Spring IoC容器是用来创建对象，管理依赖关系的。
DI(Dependency Injection)：依赖注入，即在创建对象的过程中，向类里面的属性中设置值。
IoC和DI的关系：依赖注入不能单独存在，须在控制反转基础之上完成，用更通俗点的话来说，就是注入类里面的属性值，不能直接注入，须创建类的对象再完成注入。
```



## Spring中的工厂

### ApplicationContext

ApplicationContext接口有两个实现类，

- ClassPathXmlApplicationContext：加载的是类路径下的Spring配置文件


- FileSystemXmlApplicationContext：加载的是本地磁盘下的Spring配置文件

### ApplicationContext和BeanFactory的区别

虽然使用这两个对象都可以加载Spring的配置文件，并创建配置文件中的对象。但他俩还是有区别的，最主要的区别是：

- 使用applicationContext操作时，可把Spring里面的配置文件中的对象都进行创建。
- 使用BeanFactory对象操作时，在调用getBean方法的时候进行对象的创建。







## Spring的bean管理（注解方式）



注解：代码中的特殊标记，注解可以使用在类、方法、属性上面，使用注解可实现一些基本的功能。注解的写法是`@注解名称(属性=属性值)`。

使用注解创建对象
第一步，创建Web项目，引入Spring的开发包 



除了导入Spring基本的Jar包外(可参考《Spring的概述》一文)，还须导入Spring注解的Jar包

Spring-aop-4.2.4.RELEASE



**第二步，编写相关的类** 
在src目录下创建一个cn.itcast.anno包，并在该包下编写一个User类。

```java
public class User {

    public void add() {
        System.out.println("add....................");
    }

}
```

**第三步，创建Spring配置文件**

- 在Spring配置文件中引入约束，如下：

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context" 
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context 
          http://www.springframework.org/schema/context/spring-context.xsd">
  ```

  ​

  在Spring配置文件中做些事情，即开启注解扫描。

  ```xml
  <!-- 开启注解的扫描。到配置的包里面扫描类、方法、属性上面是否有注解 -->
  <context:component-scan base-package="cn.itcast"></context:component-scan>
  ```

  注意：也可以这样开启注解扫描，如下：

```xml
<context:annotation-config></context:annotation-config>
```

但是这种开启注解扫描的方式，只会扫描属性上面的注解。实际开发中用到的并不多！故不推荐使用。



**第四步，在创建对象所在的类上面使用注解实现**

```java
@Component(value="user") // 类似于<bean id="user" class="..." />
public class User {

    public void add() {
        System.out.println("add....................");
    }

}
```



如若注解里面属性名称是value，则可以省略，所以上面的User类亦可这样写为：

```java
@Component("user") // 类似于<bean id="user" class="..." />
public class User {

    public void add() {
        System.out.println("add....................");
    }

}
```

第五步，编写测试类 

````java
public class TestDemo {

    @Test
    public void testUser() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        User user = (User) context.getBean("user");
        System.out.println(user);
        user.add();
    }

}
````



## Spring的bean管理中常用的注解

### @Component(作用在类上)

@Repository：用于对DAO实现类进行标注(持久层)。

@Service：用于对Service实现类进行标注(业务层)。

@Controller：用于对Controller实现类进行标注(WEB层)。

### bean的作用范围的注解

`@Scope`

- singleton：单例，默认值
- prototype：多例



所以我们可通过注解设置创建对象是单例或者还是多实例的。这样User类的代码亦可写为：

```java
@Service("user")
@Scope("singleton")
public class User {

    public void add() {
        System.out.println("add....................");
    }

}
```

## 使用注解注入属性（对象）



这儿，我举个例子来说明如何使用注解注入(对象类型的)属性。 
先创建业务层中的UserService类：



public class UserService {
​    public void add() {
​        System.out.println("service...........");
​    }
}



再创建持久层中的UserDao类：



public class UserDao {
​    public void add() {
​        System.out.println("dao................");
​    }
}



以上两个类都创建在cn.itcast.anno包中。我们要实现的目的是在UserService类里面调用UserDao类的方法，这样我们就要在UserService类里面得到UserDao类的对象。之前是采用xml配置文件的方式来注入属性的，本文将使用注解的方式完成注入属性的操作。



step1:在UserService类里面定义UserDao类型属性

private UserDao userDao;

在UserService类里面定义UserDao类型的属性，由于是使用注解的方式，故不需要手动生成set方法。



step2:

进行注解方式实现属性注入

创建UserDao类的对象和UserService类的对象

```java
@Service("userService")
public class UserService {

    private UserDao userDao;

    public void add() {
        System.out.println("service...........");
        userDao.add();
    }

}


@Repository("userDao")
public class UserDao {

    public void add() {
        System.out.println("dao................");
    }
}
```



在UserService类里面注入UserDao类的对象，使用注解来实现。首先我使用`@Autowired`注解来实现。

```java
@Service("userService")
public class UserService {

    @Autowired
    private UserDao userDao;

    public void add() {
        System.out.println("service...........");
        userDao.add();
    }

}
```



注意：使用注解`@Autowired`，它不是根据名字去找Dao，而是默认按类型进行装配。 
当然了，也可使用`@Resource`注解来实现，如下：

```java
@Service("userService")
public class UserService {

    @Resource(name="userDao")
    private UserDao userDao;

    public void add() {
        System.out.println("service...........");
        userDao.add();
    }

}
```



注意，使用`@Resource`注解，它默认是按名称进行注入的。在实际开发中，我们也是使用`@Resource`注解来注入属性的，注解`@Autowired`用到的并不多。







# AOP的概述

## 什么是AOP



Spring是用来解决实际开发中的一些问题的，AOP解决了OOP中遇到的一些问题，是OOP的延续和扩展。我们可从以下三个方面来理解AOP：

1.扩展功能不是通过修改源代码而实现的。 可通过Struts2框架中的拦截器来理解。

2.AOP采用横向抽取机制实现。 
要理解横向抽取机制，就必须认识纵向抽取机制。例如有如下的一个类：

```java
public class User {
    public void add() {
        添加的逻辑...
    }
}
```

现在我们在add()方法中要扩展一个功能，即日志添加功能，添加完该功能之后，可记录在什么时候添加了哪个用户。想到的最原始的方法就是直接修改源代码。

```java
public class User {
    public void add() {
        添加的逻辑...
        直接写添加日志记录的代码以实现...
    }
}
```

很显然这是一种愚蠢的做法，并且这儿还有一个原则——**修改功能一般不是直接修改源代码来实现的**。顺其自然地就要来讲**纵向抽取机制**了，这时我们可编写一个BaseUser类。

```java
public class BaseUser {
    public void wirtelog() {
        记录日志的逻辑...
    }
}
```

接下来让User类继承BaseUser类，如下：

```java
public class User extends BaseUser  {
    public void add() {
        添加的逻辑...
        // 记录日志
        super.wirtelog();
    }
}
```



这样是不是就万事大吉了呢？你懂的！因为当父类的方法名称变化时，子类调用的方法也必然要进行修改。最后终于要讲横向抽取机制了，横向抽取机制分为两种情况，下面分别加以简单阐述。 
**有接口情况的横向抽取机制：** 



例如有一个如下接口：



public interface UserDao {
​    public void add();
}

接口的一个实现类如下：

```
public class UserDaoImpl implements UserDao {
    public void add() {
        ...
    }
}
```

我们现在即可使用动态代理技术增强类里面的方法，即创建接口的实现类代理对象，并增强UserDaoImpl类里面的add()方法。 
**无接口情况的横向抽取机制：** 
例如有一个如下User类：

```
public class User {
    public void add() {
        ...
    }
}12345
```

我们也可使用动态代理技术增强类里面的方法，即创建被增强方法所在类的子类代理对象，并增强User类里面的add()方法。

3.AOP底层使用动态代理实现 

- 有接口情况：创建接口实现类代理对象
- 没有接口情况：创建增强方法所在类的子类代理对象

## 为什么学习AOP

在不修改源代码的情况下，即可对程序进行增强。AOP可以进行***权限校验***、**日志记录**、**性能监控**和**事务控制**。

## Spring的AOP的由来

AOP最早是由AOP联盟的组织提出的，他们制定了一套规范。Spring将AOP思想引入到框架中，且必须遵守AOP联盟的规范。



## Spring的AOP的底层实现

Spring的AOP的底层用到了两种代理机制：

- JDK的动态代理：针对实现了接口的类产生代理。
- Cglib的动态代理：针对没有实现接口的类产生代理，应用的是底层的字节码增强的技术，生成当前类的子类对象。



## AOP开发中的相关术语

- Joinpoint(连接点)：所谓连接点是指那些被拦截到的点。在Spring中，这些点指的是**方法**，因为**Spring只支持方法类型的连接点**。
- Pointcut(切入点)：所谓切入点是指我们要对哪些Joinpoint进行拦截的定义。
- Advice(通知/增强)：所谓通知是指拦截到Joinpoint之后所要做的事情就是通知。通知分为前置通知、后置通知、异常通知、最终通知和环绕通知(切面要完成的功能)。
- Aspect(切面)：是切入点和通知的结合。
- Target(目标对象)：代理的目标对象(要增强的类)
- Weaving(织入)：是指把增强应用到目标对象来创建新的代理对象的过程。Spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入。
- Proxy(代理)：一个类被AOP织入增强后，就产生一个结果代理类。
- Introduction(引介)：引介是一种特殊的通知在不修改类代码的前提下，Introduction可以在运行期为类动态地添加一些方法或Field。



例如有如下一个类：

```java
public class User {
    public void add() {

    }

    public void update() {

    }

    public void delete() {

    }
}
```



下面是用比较通俗易懂的话来阐述AOP开发中的常见的相关术语：

- 连接点：在User类里面有3个方法，这3个方法都可以被增强，类里面的哪些方法可以被增强，这些**方法就可被成为连接点**。


- 切入点：在一个类中可以有很多的方法被增强，在实际操作中，如若只增强了类里面的add方法，则**实际增强的方法被称为切入点**。


- 增强/通知：比如增强User类里面的add方法，在add方法中添加了日志功能，这个日志功能就称为增强。 
  通知类型：
  - 前置通知：在增强的方法执行之前进行操作。
  - 后置通知：在增强的方法执行之后进行操作。
  - 环绕通知：在增强的方法执行之前和执行之后进行操作。
  - 最终通知：增强了两个方法，执行第一个方法，执行第二个方法，在第二个方法执行之后进行操作。 
    也可理解为后置通知后面执行的通知或者**无论目标方法是否出现异常，最终通知都会执行**。
  - 异常通知：程序出现异常之后执行的通知。
- 切面：把增强应用到切入点的过程。即把具体增强的逻辑用到具体的方法上面的过程。
- 目标对象：增强的方法所在的类，即要增强的类。
- Weaving(织入)：是指把增强应用到目标对象的过程。即把把advice应用到target的过程。

## Spring的基于AspectJ的AOP开发

## @AspectJ的简介



AspectJ是一个面向切面的框架，它扩展了Java语言。

AspectJ是一个基于Java语言的AOP框架。

使用AspectJ方式来开发AOP共有两种方式：

**基于AspectJ的xml配置文件的方式**
**基于AspectJ的注解的方式**



## Spring使用AspectJ进行AOP的开发：XML的方式

**第一步，引入相应的Jar包** 
上面我说过，除了导入最基本的Jar包外，使用AspectJ还需要导入Spring AOP和AspectJ相关的Jar包。

Spring的传统AOP的开发的包： 
spring-aop-4.2.4.RELEASE.jar
aopalliance-1.0.jar

AspectJ的开发包 
aspectjweaver-1.8.7.jar
spring-aspects-4.2.4.RELEASE.jar



**第二步，编写目标类** 
在src目录下创建一个cn.itcast.aop包，并在该包下编写一个目标类。

```java
public class Book {

    public void add() {
        System.out.println("book add.................");
    }

}
```



**第三步，创建增强的类以及增强的方法**

```java
public class MyBook {

    // 前置通知
    public void before1() {
        System.out.println("before........");
    }

}
```



我们现在要求在Book类里面的add方法之前执行MyBook类里面的before1的方法。 

**第四步，在Spring配置文件中进行配置**

1. 在Spring配置文件中引入aop约束

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd">
```

2.配置两个类对象

```xml
<!-- book对象 -->
<bean id="book" class="cn.itcast.aop.Book"></bean>
<bean id="myBook" class="cn.itcast.aop.MyBook"></bean>
```

3.配置AOP操作，即需要配置切入点和切面。

```xml
<!-- 配置AOP的操作  -->
<aop:config>
    <!-- 配置切入点，对Book类里面的所有方法都增强 -->
    <aop:pointcut expression="execution(* cn.itcast.aop.Book.*(..))" id="pointcut1"></aop:pointcut>

    <!-- 配置切面    aop:aspect标签里面使用属性ref，ref属性值写增强类的bean的id值 -->
    <aop:aspect ref="myBook">
        <!-- 
            增强类型
            method属性：增强类的方法名称
            pointcut-ref属性：切入点的id值
        -->
        <!-- 前置通知 -->
        <aop:before method="before1" pointcut-ref="pointcut1"></aop:before>
    </aop:aspect>
</aop:config>
```



**第五步，编写一个单元测试类并进行测试** 
在cn.itcast.aop包下编写一个TestDemo单元测试类。

```java
public class TestDemo {

    @Test
    public void testUser() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
        Book book = (Book) context.getBean("book");
        book.add();
    }
}
```





其实我们也可以整合Junit单元测试，Spring对Junit4进行了支持，可以通过注解方便的测试Spring程序，所以就不必写那么麻烦的单元测试类了。首先导入如下Jar包： 

spring-test-4.2.4.RELEASE.jar



然后编写如下的单元测试类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/bean2.xml") // 或者也可写为：@ContextConfiguration("classpath:bean2.xml")
public class TestDemo {

    @Autowired
    private Book book;

    @Test
    public void demo1() {
        book.add();
    }

}
```

### 演示其他通知类型

先将MyBook增强类的代码修改为：

```java
public class MyBook {

    // 前置通知
    public void before1() {
        System.out.println("before........");
    }

    // 后置通知
    public void after11() {
        System.out.println("after...........");
    }

    // 环绕通知
    public void around1(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("方法之前执行...........");
        // 让被增强的方法执行
        joinPoint.proceed();
        System.out.println("方法之后执行...........");
    }

}
```

然后再在Spring配置文件中进行配置。

```xml
<!-- 配置AOP的操作  -->
<aop:config>
    <!-- 配置切入点，对Book类里面的所有方法都增强 -->
    <aop:pointcut expression="execution(* cn.itcast.aop.Book.*(..))" id="pointcut1"></aop:pointcut>

    <!-- 配置切面    aop:aspect标签里面使用属性ref，ref属性值写增强类的bean的id值 -->
    <aop:aspect ref="myBook">
        <!-- 
            增强类型
            method属性：增强类的方法名称
            pointcut-ref属性：切入点的id值
        -->
        <!-- 前置通知 -->
        <aop:before method="before1" pointcut-ref="pointcut1"></aop:before>
        <!-- 后置通知 -->
        <aop:after-returning method="after11" pointcut-ref="pointcut1"></aop:after-returning>
        <!-- 环绕通知 -->
        <aop:around method="around1" pointcut-ref="pointcut1"></aop:around>
    </aop:aspect>
</aop:config>
```

## 切入点表达式

通过execution函数，可以定义切点的方法切入。 



语法为：`execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)`。 

例如：

- 匹配所有类的public方法：`execution(public *.*(..))`
- 匹配指定包下所有类的方法：`execution(* cn.itcast.dao.*(..))`，但不包含子包
- `execution(* cn.itcast.dao..*(..))`，`..*`表示包、子孙包下所有类。
- 匹配指定类所有方法：`execution(* cn.itcast.service.UserService.*(..))`
- 匹配实现特定接口的所有类的方法：`execution(* cn.itcast.dao.GenericDAO+.*(..))`
- 匹配所有save开头的方法：`execution(* save*(..))`
- 匹配所有类里面的所有的方法：`execution(* *.*(..))`

# IoC配置文件和注解混合使用

实际开发中，我们都会混合使用IoC配置文件和注解，一般是使用配置文件方式创建对象，使用注解方式注入属性。当然了，你亦可另辟蹊径。下面我举个例子来演示。 
在src目录下创建一个cn.itcast.xmlanno包，并在该包下编写一个BookDao类、PersonDao类、BookService类。



BookDao类

```java
public class BookDao {

    public void update() {
        System.out.println("book dao.............");
    }
}
```



PersonDao类

````java
public class PersonDao {

    public void update() {
        System.out.println("person dao.............");
    }
}
````



BookService类

```java
public class BookService {

    public void update() {
        System.out.println("service............");
    }

}
```

接着创建对象，使用配置文件实现。

````xml
<bean id="bookService" class="cn.itcast.xmlanno.BookService"></bean>
<bean id="bookDao" class="cn.itcast.xmlanno.BookDao"></bean>
<bean id="personDao" class="cn.itcast.xmlanno.PersonDao"></bean>
````

也有人说创建对象，能用注解就用注解，而不要写上面这种乱七八糟的东西。我觉得也蛮有道理的。 
然后在BookService类里面注入BookDao类以及PersonDao类的对象，使用注解方式。

```java
public class BookService {

    @Resource(name="bookDao")
    private BookDao bookDao;

    @Resource(name="personDao")
    private PersonDao personDao;

    public void update() {
        System.out.println("service............");
        bookDao.update();
        personDao.update();
    }

}
```

**注意，不要忘了开启注解的扫描：**

```xml
<context:component-scan base-package="cn.itcast"></context:component-scan>
```

最后再在cn.itcast.xmlanno包下编写一个TestDemo单元测试类。

```java
public class TestDemo {

    @Test
    public void testBook() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
        BookService bookService = (BookService) context.getBean("bookService");
        bookService.update();
    }
}
```





## Spring使用AspectJ进行AOP的开发：注解方式

创建一个Web项目， 引入相关的jar包。

引入Spring的配置文件。主要引入AOP的约束：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop" xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd">

</beans>
```

编写目标类。 
在Web项目的src目录下创建一个cn.itcast.aop包，并在该包下编写一个Book类：

```
public class Book {

    public void add() {
        System.out.println("book add.................");
    }

}
```

编写切面类。 
再在cn.itcast.aop包下编写一个MyBook增强类：

```
public class MyBook {

    public void before1() {
        System.out.println("before........");
    }

}
```

- 在增强的类上面使用`@Aspect`注解

  ```
  @Aspect
  public class MyBook {

      public void before1() {
          System.out.println("before........");
      }

  }
  ```

- 在增强类的方法里面，使用注解配置通知类型：

  ```
  @Aspect
  public class MyBook {

      // 前置通知
      // *：方法的访问修饰符，也可写为execution(public void cn.itcast.aop.Book.*(..))，但一般都不会用
      @Before("execution(* cn.itcast.aop.Book.*(..))")
      public void before1() {
          System.out.println("before........");
      }

  }
  ```

AspectJ的AOP的注解：

- `@Aspect`：定义切面类的注解
- `@Before`：前置通知，相当于BeforeAdvice
- `@AfterReturning`：后置通知，相当于AfterReturningAdvice
- `@Around`：环绕通知，相当于MethodInterceptor
- `@AfterThrowing`：抛出通知，相当于ThrowAdvice
- `@After`：最终final通知，不管是否异常，该通知都会执行
- `@Pointcut`：定义切入点的注解

开启AspectJ的注解

```
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

创建Book类和MyBook类的两个对象(使用配置文件)

```
<bean id="book" class="cn.itcast.aop.Book"></bean>
<bean id="myBook" class="cn.itcast.aop.MyBook"></bean>
```

最后在cn.itcast.aop包下编写一个TestDemo单元测试类

```
public class TestDemo {

    @Test
    public void testBook() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        Book book = (Book) context.getBean("book");
        book.add();
    }
}
```

## Spring的JDBC模板

Spring是一个分层的JavaSE/EEfull-stack(一站式)轻量级开源框架。在dao层，Spring提供了JDBC模板的技术，可对数据库进行CRUD操作。

Spring框架对不同的持久层技术做了封装，如对传统的JDBC使用JdbcTemplate进行了封装，对Hibernate框架使用HibernateTemplate进行了封装。JdbcTemplate对JDBC进行了简单封装，使用类似于dbutils，但是使用并没有dbutils方便，只是提供了一种实现的方式而已。下面来演示使用JdbcTemplate模板类实现CRUD操作。

### 使用JdbcTemplate模板类实现CRUD操作

使用JdbcTemplate模板类还须导入jar包，先引入JdbcTemplate的jar包： 

spring-jdbc-4.2.4.RELEASE.jar

spring-tx-4.2.4.RELEASE.jar

但我们要知道spring-jdbc-4.2.4.RELEASE.jar包才是最主要的。除此之外还须导入MySQL数据库驱动的jar包。

### 添加操作

在src目录下创建一个cn.itcast.jdbcTemplate包，并在该包下编写一个JdbcTemplateDemo1单元测试类。现在要编写一个add方法来测试添加操作。

public class JdbcTemplateDemo1 {

    // 1.添加操作
    @Test
    public void add() {
        // 1.设置数据库相关信息(JDBC模板依赖连接池获得数据库连接，所以必须先构造连接池)
        DriverManagerDataSource dataSource = new DriverManagerDataSource(); // 数据源
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///spring_lee");
        dataSource.setUsername("root");
        dataSource.setPassword("yezi");
    
        // 2.做添加的操作
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        String sql = "insert into user values(?,?)";
        int rows = jdbcTemplate.update(sql, "liayun", "lee");
        System.out.println(rows);
    }

}
注意：JDBC模板依赖连接池获得数据库连接，所以必须先构造连接池，然后再创建JdbcTemplate模板类对象。而且还须用到JdbcTemplate模板类的update方法： 

### 修改操作

现在要在单元测试类中编写一个update方法来测试修改操作。

public class JdbcTemplateDemo1 {

    // 2.修改操作
    @Test
    public void update() {
        // 1.设置数据库相关信息
        DriverManagerDataSource dataSource = new DriverManagerDataSource(); // 数据源
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///spring_lee");
        dataSource.setUsername("root");
        dataSource.setPassword("yezi");
    
        // 实现修改操作
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        String sql = "update user set password=? where username=?";
        int rows = jdbcTemplate.update(sql, "9999", "liayun");
        System.out.println(rows);
    }

}

### 删除操作

现在要在单元测试类中编写一个delete方法来测试删除操作。

public class JdbcTemplateDemo1 {

    // 3.删除操作
    @Test
    public void delete() {
        // 1.设置数据库相关信息
        DriverManagerDataSource dataSource = new DriverManagerDataSource(); // 数据源
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///spring_lee");
        dataSource.setUsername("root");
        dataSource.setPassword("yezi");
    
        // 实现删除操作
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        String sql = "delete from user where username=?";
        int rows = jdbcTemplate.update(sql, "liayun");
        System.out.println(rows);
    }

}
查询操作
查询表中的记录数
现在要在单元测试类中编写一个testCount方法来测试查询表中记录数的操作。

public class JdbcTemplateDemo1 {

    // 查询表记录数
    @Test
    public void testCount() {
        // 1.设置数据库相关信息
        DriverManagerDataSource dataSource = new DriverManagerDataSource(); // 数据源
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql:///spring_lee");
        dataSource.setUsername("root");
        dataSource.setPassword("yezi");
    
        // 2.创建JdbcTemplate模板类的对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        // 3.sql语句
        String sql = "select count(*) from user";
        // 4.调用JdbcTemplate模板类里面的方法
        // 返回int类型
        int count = jdbcTemplate.queryForObject(sql, Integer.class);
        System.out.println(count);
    }

}



### **最原始jdbc**

好了，接下来复习一下编写JDBC最原始的代码做查询操作，基本功不要忘记了啊！ 
首先在cn.itcast.jdbcTemplate包下编写一个User类。

```java
public class User {

    private String username;
    private String password;
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    @Override
    public String toString() {
        return "User [username=" + username + ", password=" + password + "]";
    }

}
```

然后再在该包下编写一个JdbcTemplateDemo2单元测试类，编写JDBC最原始的代码做查询操作。

```java
public class JdbcTemplateDemo2 {

    // jdbc最原始的代码做查询操作
    @Test
    public void testJDBC() {
        Connection conn = null;
        PreparedStatement psmt = null;
        ResultSet rs = null;

        try {
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql:///spring_lee", "root", "yezi");
            String sql = "select * from user where username=?";
            // 对sql进行预编译操作
            psmt = conn.prepareStatement(sql);
            psmt.setString(1, "mary");
            // 执行sql
            rs = psmt.executeQuery();
            // 遍历结果
            while (rs.next()) {
                String username = rs.getString("username");
                String password = rs.getString("password");

                User user = new User();
                user.setUsername(username);
                user.setPassword(password);
                System.out.println(user);
            }
        } catch (Exception e) {

        } finally {
            try {
                rs.close();
                psmt.close();
                conn.close();
            } catch (Exception e2) {

            }
        }
    }       

}
```

## Spring配置连接池

在实际开发中，一般都会用Spring配置C3P0连接池，所以下面我就来重点介绍在Spring中如何配置C3P0连接池。 
首先引入Spring的配置文件，主要是引入约束：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">

</beans>
```

接着导入Spring的基本jar包，除此之外，还要导入C3P0的jar包：

在Spring配置文件(bean2.xml)中配置C3P0连接池了，即在Spring配置文件中添加如下配置：

```xml
<!-- 配置C3P0连接池 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///spring_lee"></property>
    <property name="user" value="root"></property>
    <property name="password" value="yezi"></property>
</bean>
```





现在举例来演示如何在Spring中配置C3P0连接池了。我的思路是这样的：**创建一个UserService类和一个UserDao类，然后在UserService类里面调用UserDao类的方法，在UserDao类中使用JdbcTemplate模板类进行数据库CRUD操作，并且用上C3P0连接池**。

先在src目录下创建一个cn.itcast.c3p0包，并在该包下编写一个UserDao类。

public class UserDao {

    // 在Dao里面要得到JdbcTemplate对象
    private JdbcTemplate jdbcTemplate;
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    // 添加操作，使用JdbcTemplate模板来实现添加
    public void add() {
        String sql = "insert into user values(?,?)";
        jdbcTemplate.update(sql, "李阿昀", "lee");
    }
}
再在该包下编写一个UserService类，并在UserService类里面调用UserDao类的add方法。

public class UserService {

    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public void add() {
        userDao.add();
    }
}

那么Spring核心配置文件就应该像下面这样配置：

```xml
<!-- 配置service和dao以及它们的注入 -->
<bean id="userService" class="cn.itcast.c3p0.UserService">
    <property name="userDao" ref="userDao"></property>
</bean>
<bean id="userDao" class="cn.itcast.c3p0.UserDao">
    <property name="jdbcTemplate" ref="jdbcTemplate"></property>
</bean>

<!-- 配置JdbcTemplate模板类的对象 -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!-- 注入dataSource，因为在其源代码中dataSource属性有其对应的set方法，故可直接注入 -->
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

从上面的配置可看出：UserDao中注入了JdbcTemplate对象，JdbcTemplate对象里面又注入了dataSource。 
最后再在该包下编写一个TestDemo单元测试类。

```
public class TestDemo {

    @Test
    public void testBook() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean2.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.add();
    }
}
```





## Spring中的事务操作

### 什么是事务

事务是逻辑上的一组操作，组成这组操作的各个逻辑单元，**要么一起成功，要么一起失败**。

### 事务的特性

原子性：强调事务的不可分割。
一致性：事务的执行的前后数据的完整性保持一致。
隔离性：一个事务执行的过程中，不应该受到其他事务的干扰。
持久性：事务一旦结束，数据就持久化到数据库。

```doc
如果不考虑隔离性会引发的安全性问题
脏读：一个事务读到了另一个事务的未提交的数据。
不可重复读：一个事务读到了另一个事务已经提交的update的数据，导致多次查询的结果不一致。
虚读：一个事务读到了另一个事务已经提交的insert的数据，导致多次查询的结果不一致。
解决读问题：设置事务的隔离级别
未提交读：脏读、不可重复读和虚读都有可能发生。
已提交读：避免脏读，但是不可重复读和虚读有可能发生。
可重复读：避免脏读和不可重复读，但是虚读有可能发生。
串行化的：避免以上所有读问题。 
mysql数据库的默认隔离级别就是可重复读。
```

# Spring进行事务操作常用的API

## PlatformTransactionManager：平台事务管理器

Spring进行事务操作时候，主要使用一个PlatformTransactionManager接口，它表示事务管理器，即真正管理事务的对象。 
Spring针对不同的持久化框架，提供了不同PlatformTransactionManager接口的实现类

```doc
TrancactionDefinition：事务定义信息
事务定义信息有：

隔离级别
传播行为
超时信息
是否只读
TrancactionStatus：事务的状态
记录事务的状态。

Spring的这组接口是如何进行事务管理的
平台事务管理器根据事务定义的信息进行事务的管理，事务管理的过程中产生一些状态，将这些状态记录到TrancactionStatus里面。

事务的传播行为
PROPAGION_XXX：事务的传播行为。

保证在同一个事务中 
PROPAGION_REQUIRED：支持当前事务，如果不存在，就新建一个(默认) 
PROPAGION_SUPPORTS：支持当前事务，如果不存在，就不使用事务 
PROPAGION_MANDATORY：支持当前事务，如果不存在，就抛出异常

保证没有在同一个事务中 
PROPAGION_REQUIRES_NEW：如果有事务存在，挂起当前事务，创建一个新的事务 
PROPAGION_NOT_SUPPORTED：以非事务方式运行，如果有事务存在，挂起当前事务 
PROPAGION_NEVER：以非事务方式运行，如果有事务存在，抛出异常 
PROPAGION_NESTED：如果当前事务存在，则嵌套事务执行

关于事务的传播行为，我真的是一点都不了解啊！希望随着时间的推移，能够真心明白。
```

## Spring的声明式事务管理方式

Spring进行声明式事务配置的方式有两种：

1. 基于xml配置文件方式
2. 基于注解方式

这两种方式我都会讲解，但无论使用什么方式进行Spring的事务操作，首先要配置一个事务管理器。

# 搭建转账的环境

```doc
现在我举例来演示Spring如何进行声明式事务的配置。例子就是模拟银行转账，首先要搭建好转账的环境。 
第一步，创建数据库表。

DROP TABLE IF EXISTS `account`;
CREATE TABLE `account` (
  `id` int(11) DEFAULT NULL,
  `username` varchar(100) DEFAULT NULL,
  `salary` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `account` VALUES ('1', '小郑', '10000');
INSERT INTO `account` VALUES ('2', '小谭', '10000');
第二步，创建一个Web项目，并引入Spring的相关jar包。 

第三步，创建业务层和DAO层的类。 
在Web项目的src目录下创建一个cn.itcast.tx包，并在该包下编写业务层和DAO层的类。

业务层——BookService.java

public class BookService {

    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

}
DAO层——BookDao.java

public class BookDao {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

}
第四步，配置业务层和DAO层的类。

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 配置C3P0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///spring_lee"></property>
        <property name="user" value="root"></property>
        <property name="password" value="yezi"></property>
    </bean>

    <!-- 创建service和dao的对象 -->
    <bean id="bookService" class="cn.itcast.tx.BookService">
        <!-- 注入dao -->
        <property name="bookDao" ref="bookDao"></property>
    </bean>
    <bean id="bookDao" class="cn.itcast.tx.BookDao">
        <!-- 注入JdbcTemplate模板类的对象 -->
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>

    <!-- 创建JdbcTemplate模板类的对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入dataSource -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
</beans>
第五步，转账的具体实现，实现小郑转账1000元给小谭。 
JavaEE中DAO层做的事情主要是对数据库进行操作，在DAO层里面一般不写业务操作，一般写单独操作数据库的方法。所以BookDao类的代码要修改为：

public class BookDao {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // 小郑少1000
    public void lessMoney() {
        String sql = "update account set salary=salary-? where username=?";
        jdbcTemplate.update(sql, 1000, "小郑");
    }

    // 小谭多1000
    public void moreMoney() {
        String sql = "update account set salary=salary+? where username=?";
        jdbcTemplate.update(sql, 1000, "小谭");
    }
}
JavaEE中Service层写具体的业务操作，所以BookService类的代码要修改为：

public class BookService {

    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    // 转账的业务
    public void accountMoney() {
        // 1.小郑少1000
        bookDao.lessMoney();

        // 2.小谭多1000
        bookDao.moreMoney();
    }
}
第六步，编写一个测试类。 
在cn.itcast.tx包下编写一个TestDemo单元测试类。

public class TestDemo {

    @Test
    public void testAccount() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = (BookService) context.getBean("bookService");
        bookService.accountMoney();
    }
}
测试以上方法即可实现小郑转账1000元给小谭。现在我来演示一个问题，在BookService类中调用BookDao类的两个方法构成了转账业务，但是如果小郑少了1000元之后，这时突然出现异常，比如银行断电，就会出现小郑的钱少了，而小谭的钱没有多，钱丢失了的情况。

public class BookService {

    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    // 转账的业务
    public void accountMoney() {
        // 1.小郑少1000
        bookDao.lessMoney();

        int x = 10 / 0; // 模拟银行断电的情况(出现的异常)

        // 2.小谭多1000
        bookDao.moreMoney();
    }
}
```

这时应该怎么解决这个问题呢？就可使用事务来解决。Spring中进行事务的操作主要有两种方式：

1. 第一种：编程式事务管理(这种了解就行，不用掌握)
2. **第二种：声明式事务管理** 
   **基于xml配置文件方式****基于注解方式**

## Spring的声明式事务管理——XML方式：思想就是AOP

基于xml配置文件的方式来进行声明式事务的操作，不需要进行手动编写代码，通过一段配置完成事务管理。下面我在搭建好的转账环境下演示它。 
第一步，配置事务管理器。 
之前，我就讲过Spring针对不同的持久化框架，提供了不同PlatformTransactionManager接口的实现类，如下：

```doc
所以我们需要在Spring的配置文件中添加如下配置：

<!-- 1.配置事务的管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 指定要对哪个数据库进行事务操作 -->
    <property name="dataSource" ref="dataSource"></property>
</bean>
第二步，配置事务的增强，即指定对哪个事务管理器进行增强。故需要向Spring的配置文件中添加如下配置：

<!-- 2.配置事务的增强，指定对哪个事务管理器进行增强 -->
<tx:advice id="txadvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--
            表示来配置你要增强的方法的匹配的一个规则，
            注意：只须改方法的命名规则，其他都是固定的！
            propagation：事务的传播行为。
        -->
        <tx:method name="account*" propagation="REQUIRED"></tx:method>
        <!-- <tx:method name="insert*" propagation="REQUIRED"></tx:method> -->
    </tx:attributes>
</tx:advice>
第三步，配置切入点和切面。这步须向Spring的配置文件中添加如下配置：

<!-- 3.配置切入点和切面(最重要的一步) -->
<aop:config>
    <!-- 切入点 -->
    <aop:pointcut expression="execution(* cn.itcast.tx.BookService.*(..))" id="pointcut1"/>
    <!-- 切面，即表示把哪个增强用在哪个切入点上 -->
    <aop:advisor advice-ref="txadvice" pointcut-ref="pointcut1"/>
</aop:config>
这时测试TestDemo单元测试类的testAccount方法即可。
```



## Spring的声明式事务的注解方式

```doc
基于注解方式来进行声明式事务的操作会更加简单，在实际开发中我们也会用的比较多。下面我在搭建好的转账环境下演示它。 
第一步，配置事务管理器。

<!-- 1.配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
第二步，开启事务注解。

<!-- 2.开启事务的注解 -->
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
以上配置添加完毕之后，Spring核心配置文件的内容就为：

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 配置C3P0连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///spring_lee"></property>
        <property name="user" value="root"></property>
        <property name="password" value="yezi"></property>
    </bean>

    <!-- 1.配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 2.开启事务的注解 -->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>

    <!-- 创建service和dao的对象 -->
    <bean id="bookService" class="cn.itcast.tx.BookService">
        <!-- 注入dao -->
        <property name="bookDao" ref="bookDao"></property>
    </bean>
    <bean id="bookDao" class="cn.itcast.tx.BookDao">
        <!-- 注入JdbcTemplate模板类的对象 -->
        <property name="jdbcTemplate" ref="jdbcTemplate"></property>
    </bean>

    <!-- 创建JdbcTemplate模板类的对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入dataSource -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
</beans>
第三步，在具体使用事务的方法所在的类上面添加注解：@Transactional。即BookService类应修改为：

@Transactional
public class BookService {

    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    // 转账的业务
    public void accountMoney() {
        // 1.小郑少1000
        bookDao.lessMoney();

        int x = 10 / 0; // 模拟银行断电的情况(出现的异常)

        // 2.小谭多1000
        bookDao.moreMoney();
    }
}
```





## Oracle

### 1.oracle dmp文件导入

注意：impdp命令在cmd下直接用，不必登录oracle。只能导入expdp导出的dmp文件。

expdp导出的时候，需要创建 DIRECTORY

导出什么表空间，导入也要什么表空间。

导出什么用户，导入也要什么用户。

如果没有要新建。

例：

```doc
从杭州服务器expdp导出了TOOLBOX用户的数据库dmp文件，要导入宁波本地开发环境中。

宁波本地oracle环境是全新的（windows环境）。



创建表空间

create tablespace TOOLBOX
logging
datafile 'C:\oraclexe\app\oracle\oradata\XE\TOOLBOX.dbf'
size 50m
autoextend on
next 32m maxsize unlimited
extent management local;

创建用户，赋予权限
create user TOOLBOX identified by 123456;
alter user TOOLBOX default tablespace TOOLBOX;
grant CREATE ANY DIRECTORY,create session,create table,create view,unlimited tablespace to TOOLBOX;

登录ToolBox用户

创建DIRECTORY

CREATE OR REPLACE DIRECTORY 
DMPDIR AS  'c:\'; 



编写导入impdp语句

impdp toolbox/123456 DIRECTORY=DMPDIRDUMPFILE=hz_toolbox_20160613.dmp full=y
```



实际运用

```doc
oracle dmp文件导入
使用 SQL Plus 工具

创建表空间（要和导出来的表空间一样的名字）
create tablespace APPA datafile 'D:\Arvin\oraspace\appa.dbf' size 4000M;

创建用户
create user zwj identified by zwj;

授权用户
grant connect,resource,dba to zwj;

给用户分配表空间
alter user zwj default tablespace APPA;

alter user zwj quota unlimited on APPD;

创建目录 (用来做为dmp文件存储的地方)
create or replace directory dp_dir as 'D:\dmparea';


进入DOS窗口

impdp zwj/zwj@orcl directory=dp_dir dumpfile=eastday.dmp logfile=emp.log full=y;
impdp eastdaybar/eastdaybar@eastday directory=eastdaydata dumpfile=eastday.dmp





--删除用户
drop user 用户名称 cascade;
--删除表空间
drop tablespace 表空间名称 including contents and datafiles cascade constraint;

select * from all_users;  查看你能管理的所有用户！

1.查看sql数据库中的所有表空间的信息：

       select * from dba_data_files
	   
	   
方法一：增加数据文件alter tablespace h2_busi add datafile \'E:\\H2DATA\\H2_BUSI_1.DBF\' size 500M autoextend on next 10M maxsize unlimited;
	   
方法二：修改数据文件大小为自增且无限大
ALTER DATABASE datafile 'D:\Arvin\oraspace\H2_BUSI.DBF' autoextend ON NEXT 10m maxsize unlimited;
```

### 2.创建100G表空间

```doc
CREATE BIGFILE TABLESPACE APPD
datafile 'e:\appd.dbf'  size 100000M Autoextend on;
```







## JAVA

#### 1.类型转换

```java
// Java 如何将String转化为Int
  String str = "123";
try {
    int a = Integer.parseInt(str);
} catch (NumberFormatException e) {
  // 防止字符串不是数字
    e.printStackTrace();
}

String str = "123";
try {
    int b = Integer.valueOf(str).intValue()
} catch (NumberFormatException e) {
    e.printStackTrace();
}
```

#### 2.BeanUtils.copyProperties(A,B)字段复制

```doc
BeanUtils提供对Java反射和自省API的包装。其主要目的是利用反射机制对JavaBean的属性进行处理。我们知道，一个JavaBean通常包含了大量的属性，很多情况下，对JavaBean的处理导致大量get/set代码堆积，增加了代码长度和阅读代码的难度。

 注意：属性复制，不同jar中的方法，用法不一样。
1、package org.springframework.beans;中的

     BeanUtils.copyProperties(A,B);

     是A中的值付给B

2、package org.apache.commons.beanutils;（常用）

      BeanUtils.copyProperties(A,B);

      是B中的值付给A


```

### 加密 MD5   SHA  加盐



## Use Tool

### 1.postmain

```doc
请求url中 localhost管用，ip不能访问？
解决：ip冲突了，禁用无线网络即可。
```

### 2.PL/SQL register

```doc
版本：12.0.7.1837  (64 bit)

product code： 4vkjwhfeh3ufnqnmpr9brvcuyujrx3n3le 
serial Number：226959 
password: xs374ca
```



### 获取request

```java
		System.err.println("all--"+request.getQueryString());
		System.err.println("请求方式--"+request.getMethod());
		System.err.println("请求资源--"+request.getRequestURI());
		System.err.println("请求http协议版本--"+request.getProtocol());
		System.err.println("根据name,获取对应请求头数据--"+request.getHeader("Cookie"));
		System.err.println("获取所有的请求头名称--"+request.getHeaderNames());
		System.err.println("获取所有参数名称列表--"+request.getParameterNames());
```

## JSON

### 一、数据转换成Json

```java
Map 转成 Json
1. json-lib
   Map map = new HashMap();  
   map.put("msg", "yes");//map里面装有yes  
   JSONObject jsonObject = JSONObject.fromObject(map);  

2. com.alibaba.fastjson.JSON
  JSONUtils.toJSONString(requestMap);

3.com.google.gson.2.2.2.jar 
  new Gson().toJson(param);

String转换成JSON
  String str = "{\"result\":\"success\",\"message\":\"成功！\"}";  
        JSONObject json = JSONObject.fromObject(str);  
Object转换成JSON
  ResultCode rc = new ResultCode(new ResultData("4fgrrt5343dfdf",  "56576767"), 0, 0, "uploadImg", "", "");
        JSONObject json2 = JSONObject.fromObject(rc);
```







## Linux

### 1.Permission denied的解决办法

```doc
解决的办法：

$ sudo chmod -R 777 myResources
其中
-R 是指级联应用到目录里的所有子目录和文件
777 是所有用户都拥有最高权限

查看当前开启的tomcat 进程
ps -ef|grep tomcat

杀死pid的进程
kill -9   pid     #pid 为相应的进程号
 kill -9 11209
```

### 2.查看linux是多少位操作系统

```doc
# getconf LONG_BIT
```

### 3.JDK版本不兼容

```doc
Unsupported major.minor version 52.0 (unable to load class com.servlet.MailSenderServlet)
```

### 4.实时查看tomcat日志

```doc
定位进入tomcat 的文件夹logs
tail -f catalina.out
```

### 5.linux安装与卸载jdk

```doc
step1
首先去官网下载一个.tar.gz后缀的jdk

step2
卸载当前低版本的,查看当前系统的jdk版本
rpm -qa | grep gcj或者rpm -qa | grep jdk查看jdk的具体信息
rpm -e --nodeps java-1.5.0-gcj-1.5.0.0-29.1.el6.x86_64命令卸载相应的jdk；

step3
定位到 jdk 文件 解压
tar -zxvf jdk-8u131-linux-x64.tar.gz

step4
设置环境变量
export JAVA_HOME=/root/server/jdk1.8/jdk1.8.0_172
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
echo $PATH
java -version

```



### https ssl 

cmd进入dos窗口输入

生成 keystore

 keytool  -genkey    -keystore   "d:\mykey.keystore"     -alias   localhost    -keyalg   RSA 



### SocketServer 和 SocketClient

```java
Java Socket实现信息共享 - 聊天室
  客户端设计思路：客户端循环可以不停的输入消息，将消息发送给服务端，并开启一个线程监听服务端发过来的消息并进行处理，接收到的消息打印在控制台上。还可以输入特殊的命令，如果输入viewuser，则获取聊天室在线成员列表，输入quit，则申请退出聊天室。
  服务端设计思路：服务端启动时，启动一个队列监听器去监听聊天室内任何一个成员推送一条消息时，将消息发送给所有成员。每接收到一个客户端请求连接时，就新开一个线程进行处理，并将线程存放到线程集合中，将成员的端口后当做成员名称存放到公共的成员集合中。客户端所发送的所有消息都会存放到消息队列（ArrayBlockingQueue）中，队列监听器在队列中获取到消息后循环发送给聊天室内所有在线的成员，通知所有成员该消息是谁发送的，消息内容是什么。
  
```



### String 类中的字符串包含方法

```java
1. java 判断一个字符串是否包含某个字符 contains
   String str = "abc";  
   boolean status = str.contains("a"); 
2. 判断一个字符串中是否包含某个字符串。indexOf
  String str1 = "abcdefg"; 
   int result1 = str1.indexOf("abc");  
    if(result1 != -1){  
        System.out.println("字符串str中包含子串“abc”"+result1);  
    }else{  
        System.out.println("字符串str中不包含子串“abc”"+result1);  
    }  
  
```



2018/3/29    星期四  晴

### Git与Github的连接与使用

```doc
一.使用git 将项目上传到GitHub上

因为本地Git仓库和GitHub仓库之间的传输是通过SSH加密传输的，GitHub需要识别是否是你推送，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送，所以需要配置ssh key。

1.创建SSH Key。

在用户主目录（C:\Users\Administrator）下，看看有没有.ssh文件，如果有，再看文件下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接到下一步。如果没有，打开Git Bash，输入命令，创建SSH Key

$ ssh-keygen -t rsa -C "123@126.com" //123 是你自己注册GitHub的邮箱

创建成功啦，再去用户主目录里找到.ssh文件夹，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露，id_rsa.pub是公钥，可以公开。

2.接下来到GitHub上，打开“Account settings”--“SSH Keys”页面，然后点击“Add SSH Key”，填上Title（随意写），在Key文本框里粘贴 id_rsa.pub文件里的全部内容。

3.验证是否成功，在git bash里输入下面的命令
ssh -T git@github.com
如果初次设置的话，会出现如下界面，输入yes 同意即可

4.下面开始设置username和email，因为github每次commit都会记录他们
 git config --global user.name  "name"//你的GitHub登陆名
 git config --global user.email "123@126.com"//你的GitHub注册邮箱

5.接下来就是把本地仓库传到github上去，之前在GitHub上建好一个新的仓库是，跳转的页面，完全按照上面的只是操作就可以了。
git remote add origin git@github.com:flora0103/example.git    //关联一个远程库命令， git@github.com:flora0103/example.git   这个是自己远程库
git push -u origin master    //关联后,第一次推送master分支的所有内容命令，此后，每次本地提交后，就可以使用命令git push origin master推送最新修改




```

## PowerDesigner(数据库建模工具)

https://www.cnblogs.com/biehongli/p/6025954.html



概念模型(CDM Conceptual Data Model)

物理模型（PDM,Physical Data Model）

面向对象的模型（OOM Objcet Oriented Model）

业务模型（BPM Business Process Model）



Name: 实体名字一般为中文如论坛用户

Code: 实体代号,一般用英文如XXXUser

Comment:注释对此实体详细说明。

Code属性代号一般用英文UID DataType

Domain域表示属性取值范围如可以创建10个字符的地址域 

M:Mandatory强制属性，表示该属性必填。不能为空

P:Primary Identifer是否是主标识符，表示实体店唯一标识符

D:Displayed显示出来，默认全部勾选



## Exception

```doc
1.java.io.IOException: 文件名、目录名或卷标语法不正确。
解决：文件命名有问题。

Windows 中文件夹命名规则是： 
① 文件名或文件夹名可以由1～256个西文字符或128个汉字（包括空格）组成，不能多于256个字符。 
② 文件名可以有扩展名，也可以没有。有些情况下系统会为文件自动添加扩展名。一般情况下，文件名与扩展名中间用符号“.”分隔。 
③ 文件名和文件夹名可以由字母、数字、汉字或~、!、@、#、$、%、^、&、( )、_、-、{}、’等组合而成。 
④ 可以有空格，可以有多于一个的圆点。 
⑤ 文件名或文件夹名中不能出现以下字符：\、/、:、*、?、"、<、>、| 。 
⑥ 不区分英文字母大小写。 



```



2018/3/30   星期五  阴

### java  获取tomcat下目录

```doc

String path = request.getSession().getServletContext().getRealPath("/"); 
			// path:D:\Arvin\AreaCode\.metadata\.plugins\org.eclipse.wst.server.core\tmp1\wtpwebapps\IHAD-master\
			String headSculpturePath = request.getSession().getServletContext().getRealPath("/upload")+"/";
			// D:\Arvin\AreaCode\.metadata\.plugins\org.eclipse.wst.server.core\tmp1\wtpwebapps\IHAD-master\upload/
			String nodepath = this.getClass().getClassLoader().getResource("/").getPath();  
			// nodepath:/D:/Arvin/AreaCode/.metadata/.plugins/org.eclipse.wst.server.core/tmp1/wtpwebapps/IHAD-master/WEB-INF/classes/
			//String temporaryDirectory = request.getSession().getServletContext().getRealPath("/WEB-INF/upload/temporary_directory/" +"pengyue"+uuid+sdfn.format(new Date()));
```







### socket长连接与短连接

```java
短连接

连接->传输数据->关闭连接

HTTP是无状态的，浏览器和服务器每进行一次HTTP操作，就建立一次连接，但任务结束后就中断连接。短连接是指SOCKET连接后发送后接收完数据后马上断开连接

长连接

连接->传输数据->保持连接->传输数据->....->关闭连接

长连接指建立SOCKET连接后不管是否使用都保持连接，但安全性较差。

http的长连接

HTTP也可以建立长连接的，使用Connection:keep-alive,HTTP1.1默认进行持久连接。HTTP1.1和HTTP1.0相比较而言，最大的区别就是增加了持久连接支持，但还是无状态的，或者说是不可以信任的。

那什么场景下使用长连接或者短连接？

长连接多用于操作频繁，点对点的通讯(及时通讯)，而且连接数不能太多。每个TCP连接都需要三次握手，建立连接，会比较耗时。如果每个操作都要先连接，再操作的话，处理速度会降低很多。长连接的特点是每个操作完后都不断开连接，直接发送数据包就可以了，不需要每次都建立连接。
  
一、长连接和短连接的概念

　　1、长连接与短连接的概念：前者是整个通讯过程，客户端和服务端只用一个Socket对象，长期保持Socket的连接；后者是每次请求，都新建一个Socket,处理完一个请求就直接关闭掉Socket。所以，其实区分长短连接就是：整个客户和服务端的通讯过程是利用一个Socket还是多个Socket进行的。

　　可能你会想：这还不简单，长连接不就是不关Socket嘛，短连接不就是每次都关Socket每次都new Socket嘛。然而事实其实并没有那么简单的，请继续看下面的整理
  
  
  在关闭了对应的流（无论是输出或者输入）之后，下一次调用getInputStream或者getOutputStream会抛出异常说：Socket is closed；这里讲明一个事实：Socket和流联系着，流关闭了，Socket其实也就相当于关闭状态！

　　其实这个也很好理解，Socket本来就是依靠流进行关闭的，流，就只有一个，你关闭了流，Socket赖以通讯的渠道也就关闭了，与客户的连接也断开了，所以抛出异常是很合理的。

　　所以，流关闭而要求Socket正常通讯是不可能的！
  
总结  
1、对应流关闭，Socket的对应输入（出）数据的通道也就关闭，此时无法达到长连接效果；
2、关闭Socket，记得显式关闭流与socket,顺序是线管流再关socket.

3、要实先长连接，一般需要发送结束标记符号来告诉客户端服务端的某段消息已经发送完毕，否则客户端会一直阻塞在read方法。

```

## Java 流(Stream)、文件(File)和IO 



```java
Java.io 包几乎包含了所有操作输入、输出需要的类。所有这些流类代表了输入源和输出目标。
Java.io 包中的流支持很多种格式，比如：基本类型、对象、本地化字符集等等。
一个流可以理解为一个数据的序列。
输入流表示从一个源读取数据，输出流表示向一个目标写数据。
Java 为 I/O 提供了强大的而灵活的支持，使其更广泛地应用到文件传输和网络编程中。
Reader 字符流    Stream 字节流

```



#### 读取控制台输入

```doc
Java 的控制台输入由 System.in 完成。
为了获得一个绑定到控制台的字符流，
你可以把 System.in 包装在一个 BufferedReader 对象中来创建一个字符流。

创建 BufferedReader 的基本语法：
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
BufferedReader 对象创建后，我们便可以使用 read() 方法从控制台读取一个字符，或者用 readLine() 方法读取一个字符串。
从控制台读取多字符输入
从 BufferedReader 对象读取一个字符要使用 read() 方法
int read( ) throws IOException
每次调用 read() 方法，它从输入流读取一个字符并把该字符作为整数值返回。 当流结束的时候返回 -1。该方法抛出 IOException。
程序示范了用 read() 方法从控制台不断读取字符直到用户输入 "q"。

从控制台读取字符串
从标准输入读取一个字符串需要使用 BufferedReader 的 readLine() 方法。

它的一般格式是：

String readLine( ) throws IOException
下面的程序读取和显示字符行直到你输入了单词"end"。

控制台输出
在此前已经介绍过，控制台的输出由 print( ) 和 println() 完成。这些方法都由类 PrintStream 定义，System.out 是该类对象的一个引用。

PrintStream 继承了 OutputStream类，并且实现了方法 write()。这样，write() 也可以用来往控制台写操作。

PrintStream 定义 write() 的最简单格式如下所示：

void write(int byteval)
该方法将 byteval 的低八位字节写到流中。



```

```java
public class BufferedReaderRead {
	public static void main(String[] args) throws IOException {
		char c;
		// 使用System.in 创建 BufferReader 字符流
		BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
		System.err.println("输入q退出");
		do {
			c = (char)bufferedReader.read();
			System.out.println(c);
		}while(c!='q');
		
	}
}


public class BufferedReaderReadLine {
	public static void main(String[] args) throws IOException {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		String test;
		System.out.println("Enter 'end' to quit.");
		do {
			test = br.readLine();
			System.err.println(test);
		} while (!test.equals("end"));

	}
}

```

#### FileInputStream 

字节流，输入流 ，读取文件

```doc

该流用于从文件读取数据，它的对象可以用关键字 new 来创建。
可以使用字符串类型的文件名来创建一个输入流对象来读取文件：
InputStream f = new FileInputStream("C:/java/hello");

也可以使用一个文件对象来创建一个输入流对象来读取文件。我们首先得使用 File() 方法来创建一个文件对象：

File f = new File("C:/java/hello");
InputStream out = new FileInputStream(f);

创建了InputStream对象，就可以使用下面的方法来读取流或者进行其他的流操作。

public void close() throws IOException{}
关闭此文件输入流并释放与此流有关的所有系统资源。抛出IOException异常。

protected void finalize()throws IOException {}
这个方法清除与该文件的连接。确保在不再引用文件输入流时调用其 close 方法。抛出IOException异常。

public int read(int r)throws IOException{}
这个方法从 InputStream 对象读取指定字节的数据。返回为整数值。返回下一字节数据，如果已经到结尾则返回-1。

public int read(byte[] r) throws IOException{}
这个方法从输入流读取r.length长度的字节。返回读取的字节数。如果是文件结尾则返回-1。

public int available() throws IOException{}
返回下一次对此输入流调用的方法可以不受阻塞地从此输入流读取的字节数。返回一个整数值。


```

#### FileOutputStream

字节流，输出流，向文件写数据。

```doc
该类用来创建一个文件并向文件中写数据。

如果该流在打开文件进行输出前，目标文件不存在，那么该流会创建该文件。

有两个构造方法可以用来创建 FileOutputStream 对象。

使用字符串类型的文件名来创建一个输出流对象：
OutputStream f = new FileOutputStream("C:/java/hello")

也可以使用一个文件对象来创建一个输出流来写文件。我们首先得使用File()方法来创建一个文件对象：

File f = new File("C:/java/hello");
OutputStream f = new FileOutputStream(f);

创建OutputStream 对象完成后，就可以使用下面的方法来写入流或者进行其他的流操作。

public void close() throws IOException{}
关闭此文件输入流并释放与此流有关的所有系统资源。抛出IOException异常。

protected void finalize()throws IOException {}
这个方法清除与该文件的连接。确保在不再引用文件输入流时调用其 close 方法。抛出IOException异常。

public void write(int w)throws IOException{}
这个方法把指定的字节写到输出流中。

public void write(byte[] w)
把指定数组中w.length长度的字节写到OutputStream中。


```

```java
public class FileStream666 {
	public static void main(String[] args) throws IOException {

		File f = new File("D:\\output.txt");
		FileOutputStream fop = new FileOutputStream(f);
		// 构建FileOutputStream对象,文件不存在会自动新建

		OutputStreamWriter writer = new OutputStreamWriter(fop, "UTF-8");
		// 构建OutputStreamWriter对象,参数可以指定编码,默认为操作系统默认编码,windows上是gbk

		writer.append("中文输入");
		// 写入到缓冲区

		writer.append("\r\n");
		// 换行

		writer.append("English");
		// 刷新缓存冲,写入到文件,如果下面已经没有写入的内容了,直接close也会写入

		writer.close();
		// 关闭写入流,同时会把缓冲区内容写入文件,所以上面的注释掉

		fop.close();
		// 关闭输出流,释放系统资源

		FileInputStream fip = new FileInputStream(f);
		// 构建FileInputStream对象

		InputStreamReader reader = new InputStreamReader(fip, "UTF-8");
		// 构建InputStreamReader对象,编码与写入相同

		StringBuffer sb = new StringBuffer();
		while (reader.ready()) {
			sb.append((char) reader.read());
			// 转成char加到StringBuffer对象中
		}
		System.out.println(sb.toString());
		reader.close();
		// 关闭读取流

		fip.close();
		// 关闭输入流,释放系统资源

	}
}
```



#### Java中的目录

```doc
File类中有两个方法可以用来创建文件夹：

mkdir( )方法创建一个文件夹，成功则返回true，失败则返回false。
失败表明File对象指定的路径已经存在，或者由于整个路径还不存在，该文件夹不能被创建。
mkdirs()方法创建一个文件夹和它的所有父文件夹。

import java.io.File;
 
public class CreateDir {
  public static void main(String args[]) {
    String dirname = "/tmp/user/java/bin";
    File d = new File(dirname);
    // 现在创建目录
    d.mkdirs();
  }
}

读取目录
一个目录其实就是一个 File 对象，它包含其他文件和文件夹。
如果创建一个 File 对象并且它是一个目录，那么调用 isDirectory() 方法会返回 true。
可以通过调用该对象上的 list() 方法，来提取它包含的文件和文件夹的列表。

下面展示的例子说明如何使用 list() 方法来检查一个文件夹中包含的内容：
import java.io.File;
 
public class DirList {
  public static void main(String args[]) {
    String dirname = "/tmp";
    File f1 = new File(dirname);
    if (f1.isDirectory()) {
      System.out.println( "目录 " + dirname);
      String s[] = f1.list();
      for (int i=0; i < s.length; i++) {
        File f = new File(dirname + "/" + s[i]);
        if (f.isDirectory()) {
          System.out.println(s[i] + " 是一个目录");
        } else {
          System.out.println(s[i] + " 是一个文件");
        }
      }
    } else {
      System.out.println(dirname + " 不是一个目录");
    }
  }
}


删除目录或文件

删除文件可以使用 java.io.File.delete() 方法。
以下代码会删除目录/tmp/java/，即便目录不为空。

import java.io.File;
 
public class DeleteFileDemo {
  public static void main(String args[]) {
      // 这里修改为自己的测试目录
    File folder = new File("/tmp/java/");
    deleteFolder(folder);
  }
 
  //删除文件及目录
  public static void deleteFolder(File folder) {
    File[] files = folder.listFiles();
        if(files!=null) { 
            for(File f: files) {
                if(f.isDirectory()) {
                    deleteFolder(f);
                } else {
                    f.delete();
                }
            }
        }
        folder.delete();
    }
}
```



## Java IO

http://www.cnblogs.com/skywang12345/p/io_01.html

### Java IO简介

```doc
java io系统的设计初衷，就是为了实现“文件、控制台、网络设备”这些io设置的通信。例如，对于一个文件，我们可以打开文件，然后进行读取和写入。
在java 1.0中，java提供的类都是以字节(byte)为单位，例如，FileInputStream和FileOutputStream。而到了java 1.1，为了与国际化进行接轨，在java io中添加了许多以字符(Unicode)为单位进行操作的类。

在java io的称呼中，我们经常会提到“输入流”、“输出流”等等概念。首先，什么是流呢？
所谓“流”，就是一种抽象的数据的总称，它的本质是能够进行传输。
      a) 按照“流”的数据流向，可以将其化分为：输入流和输出流。
      b) 按照“流”中处理数据的单位，可以将其区分为：字节流和字符流。在java中，字节是占1个Byte，即8位；而字符是占2个Byte，即16位。而且，需要注意的是，java的字节是有符号类型，而字符是无符号类型！
```



### Java IO框架

```doc
1. 以字节为单位的输入流的框架


                  |--ByteArrayInputStream				
                  |--PipedInputStream
                  |							|DataInputStream 
inputStream-------|--FilterInputStream  ---
                  |							|BufferedInputStream 
                  |						|File
                  |--FileInputStream ---
                  |						|FileDescriptor 
                  |--ObjectInputStream


从中，我们可以看出。
(01) InputStream 是以字节为单位的输入流的超类。InputStream提供了read()接口从输入流中读取字节数据。
(02) ByteArrayInputStream 是字节数组输入流。它包含一个内部缓冲区，该缓冲区包含从流中读取的字节；通俗点说，它的内部缓冲区就是一个字节数组，而ByteArrayInputStream本质就是通过字节数组来实现的。
(03) PipedInputStream 是管道输入流，它和PipedOutputStream一起使用，能实现多线程间的管道通信。
(04) FilterInputStream 是过滤输入流。它是DataInputStream和BufferedInputStream的超类。
(05) DataInputStream 是数据输入流。它是用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”。
(06) BufferedInputStream 是缓冲输入流。它的作用是为另一个输入流添加缓冲功能。
(07) File 是“文件”和“目录路径名”的抽象表示形式。关于File，注意两点：
a), File不仅仅只是表示文件，它也可以表示目录！
b), File虽然在io保重定义，但是它的超类是Object，而不是InputStream。
(08) FileDescriptor 是“文件描述符”。它可以被用来表示开放文件、开放套接字等。
(09) FileInputStream 是文件输入流。它通常用于对文件进行读取操作。
(10) ObjectInputStream 是对象输入流。它和ObjectOutputStream一起，用来提供对“基本数据或对象”的持久存储。



2.以字节为单位的输出流的框架图



                  |--ByteArrayOutputStream				
                  |--PipedOutputStream
                  |							|DataOutputStream 
OutputStream-------|--FilterOutputStream  ---
                  |							|BufferedOutputStream 
                  |						|File
                  |--FileOutputStream ---
                  |						|FileDescriptor 
                  |--ObjectOutputStream


从中，我们可以看出。以字节为单位的输出流的公共父类是OutputStream。
(01) OutputStream 是以字节为单位的输出流的超类。OutputStream提供了write()接口从输出流中读取字节数据。
(02) ByteArrayOutputStream 是字节数组输出流。写入ByteArrayOutputStream的数据被写入一个 byte 数组。缓冲区会随着数据的不断写入而自动增长。可使用 toByteArray() 和 toString() 获取数据。
(03) PipedOutputStream 是管道输出流，它和PipedInputStream一起使用，能实现多线程间的管道通信。
(04) FilterOutputStream 是过滤输出流。它是DataOutputStream，BufferedOutputStream和PrintStream的超类。
(05) DataOutputStream 是数据输出流。它是用来装饰其它输出流，它“允许应用程序以与机器无关方式向底层写入基本 Java 数据类型”。
(06) BufferedOutputStream 是缓冲输出流。它的作用是为另一个输出流添加缓冲功能。
(07) PrintStream 是打印输出流。它是用来装饰其它输出流，能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。
(08) FileOutputStream 是文件输出流。它通常用于向文件进行写入操作。
(09) ObjectOutputStream 是对象输出流。它和ObjectInputStream一起，用来提供对“基本数据或对象”的持久存储。




3.以字符为单位的输入流的框架图
下面，是以字符为单位的输入流的框架图。

          |CharArrayReader 
          |PipedReader 
          |FilterReader 
Reader----
          |BufferedReader 
          |InputStreamReader <--FileReader-->File
          
从中，我们可以看出。以字符为单位的输入流的公共父类是Reader。
(01) Reader 是以字符为单位的输入流的超类。它提供了read()接口来取字符数据。
(02) CharArrayReader 是字符数组输入流。它用于读取字符数组，它继承于Reader。操作的数据是以字符为单位！
(03) PipedReader 是字符类型的管道输入流。它和PipedWriter一起是可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedWriter和PipedReader配套使用。
(04) FilterReader 是字符类型的过滤输入流。
(05) BufferedReader 是字符缓冲输入流。它的作用是为另一个输入流添加缓冲功能。
(06) InputStreamReader 是字节转字符的输入流。它是字节流通向字符流的桥梁：它使用指定的 charset 读取字节并将其解码为字符。
(07) FileReader 是字符类型的文件输入流。它通常用于对文件进行读取操作。


4.以字符为单位的输出流的框架图

下面，是以字符为单位的输出流的框架图。





          |CharArrayWriter 
          |PipedWriter 
          |FilterWriter 
Writer----
          |BufferedWriter
          |InputStreamWriter <--FileWriter-->File
          |PrintWriter

(01) Writer 是以字符为单位的输出流的超类。它提供了write()接口往其中写入数据。
(02) CharArrayWriter 是字符数组输出流。它用于读取字符数组，它继承于Writer。操作的数据是以字符为单位！
(03) PipedWriter 是字符类型的管道输出流。它和PipedReader一起是可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedWriter和PipedWriter配套使用。
(04) FilterWriter 是字符类型的过滤输出流。
(05) BufferedWriter 是字符缓冲输出流。它的作用是为另一个输出流添加缓冲功能。
(06) OutputStreamWriter 是字节转字符的输出流。它是字节流通向字符流的桥梁：它使用指定的 charset 将字节转换为字符并写入。
(07) FileWriter 是字符类型的文件输出流。它通常用于对文件进行读取操作。
(08) PrintWriter 是字符类型的打印输出流。它是用来装饰其它输出流，能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。



5.字节转换为字符流的框架图

在java中，字节流能转换为字符流，下面是它们的转换关系图。
File<-----依赖----FileInputStream<---依赖---InputStreamReader<----继承---FileReader

从中，我们可以看出。
(01) FileReader继承于InputStreamReader，而InputStreamReader依赖于InputStream。具体表现在InputStreamReader的构造函数是以InputStream为参数。我们传入InputStream，在InputStreamReader内部通过转码，将字节转换成字符。
(02) FileWriter继承于OutputStreamWriter，而OutputStreamWriter依赖于OutputStream。具体表现在OutputStreamWriter的构造函数是以OutputStream为参数。我们传入OutputStream，在OutputStreamWriter内部通过转码，将字节转换成字符。


```



## 事务

```doc
事务是为了保证对同一数据表操作的一致性。
即多条语句放在事务中执行的时候，要么一起成功，要么全不成功。如果想看严格定义，就去随便找一本数据库教材看看吧。
我只说我的理解：数据库中的事务就是需要捆绑在一起执行的操作集合，他们应不能被部分的完成。
虽然事务并不都是用在数据库中的，但他们都有以下共同性质
事务应当具有ACID性质，
A是原子性(atomic)：事务中包含的各项操作必须全部成功执行或者全部不执行。任何一项操作失败，将导致整个事务失败，其他已经执行的任务所作的数据操作都将被撤销，只有所有的操作全部成功，整个事务才算是成功完成。
C是一致性(consistent)：保证了当事务结束后，系统状态是一致的。那么什么是一致的系统状态？例如，如果银行始终遵循着"银行账号必须保持正态平衡"的原则，那么银行系统的状态就是一致的。上面的转账例子中，在取钱的过程中，账户会出现负态平衡，在事务结束之后，系统又回到一致的状态。这样，系统的状态对于客户来说，始终是一致的。
I是隔离性(isolated)：使得并发执行的事务，彼此无法看到对方的中间状态。保证了并发执行的事务顺序执行，而不会导致系统状态不一致。
D是持久性(durable)：保证了事务完成后所作的改动都会被持久化，即使是发生灾难性的失败。可恢复性资源保存了一份事务日志，如果资源发生故障，可以通过日志来将数据重建起来。
```





### JAVA 消息队列

消息队列解决的是将突发大量请求转换为后端能承受的队列请求

```doc
消息队列的使用场景是怎样的？
经常听到诸如rebbitmq,activemq，请教一下各位前辈消息队列的使用场景是怎样的，什么时候会用到它
 
校验用户名等信息，如果没问题会在数据库中添加一个用户记录
如果是用邮箱注册会给你发送一封注册成功的邮件，手机注册则会发送一条短信
分析用户的个人信息，以便将来向他推荐一些志同道合的人，或向那些人推荐他
发送给用户一个包含操作指南的系统通知
等等……

除了这些之外，比较常用的就是起到消峰时需要用到：
比如你的服务器一秒能处理100个订单，但秒杀活动1秒进来1000个订单，持续10秒，在后端能力无法增加的情况下，你可以用消息队列将总共10000个请求压在队列里，后台consumer按原有能力处理，100秒后处理完所有请求（而不是直接宕机丢失订单数据）。

消息队列的主要特点是异步处理，主要目的是减少请求响应时间和解耦。所以主要的使用场景就是将比较耗时而且不需要即时（同步）返回结果的操作作为消息放入消息队列。同时由于使用了消息队列，只要保证消息格式不变，消息的发送方和接收方并不需要彼此联系，也不需要受对方的影响，即解耦和。

使用场景的话，举个例子：
假设用户在你的软件中注册，服务端收到用户的注册请求后，它会做这些操作：
校验用户名等信息，如果没问题会在数据库中添加一个用户记录
如果是用邮箱注册会给你发送一封注册成功的邮件，手机注册则会发送一条短信
分析用户的个人信息，以便将来向他推荐一些志同道合的人，或向那些人推荐他
发送给用户一个包含操作指南的系统通知
等等……
但是对于用户来说，注册功能实际只需要第一步，只要服务端将他的账户信息存到数据库中他便可以登录上去做他想做的事情了。至于其他的事情，非要在这一次请求中全部完成么？值得用户浪费时间等你处理这些对他来说无关紧要的事情么？所以实际当第一步做完后，服务端就可以把其他的操作放入对应的消息队列中然后马上返回用户结果，由消息队列异步的进行这些操作。

或者还有一种情况，同时有大量用户注册你的软件，再高并发情况下注册请求开始出现一些问题，例如邮件接口承受不住，或是分析信息时的大量计算使cpu满载，这将会出现虽然用户数据记录很快的添加到数据库中了，但是却卡在发邮件或分析信息时的情况，导致请求的响应时间大幅增长，甚至出现超时，这就有点不划算了。面对这种情况一般也是将这些操作放入消息队列（生产者消费者模型），消息队列慢慢的进行处理，同时可以很快的完成注册请求，不会影响用户使用其他功能。

所以在软件的正常功能开发中，并不需要去刻意的寻找消息队列的使用场景，而是当出现性能瓶颈时，去查看业务逻辑是否存在可以异步处理的耗时操作，如果存在的话便可以引入消息队列来解决。否则盲目的使用消息队列可能会增加维护和开发的成本却无法得到可观的性能提升，那就得不偿失了。
 
 
 
我可以举个小例子先说明应用场景

假设你的服务器每分钟的处理量为200个，但客户端再峰值的时候可能一分钟会发1000个消息给你，这时候你就可以把他做成队列，然后按正常有序的处理，先进后出(LIFO)，先进先出(FIFO）可根据自己的情况进行定夺

stack  先进后出(LIFO)--------java 对应的类 Stack

队列 先进先出(FIFO）--------java对应的类Queue



/** 
 * 
 * --------->>>>>>队列的实现-------------- 
 */  
public class MyStack<T> {  
    private LinkedList<T> storage = new LinkedList<T>();  
  
    public synchronized void push(T e) {//需要加上同步  
        storage.addFirst(e);  
    }  
  
    public T peek() {  
        return storage.getFirst();  
    }  
  
    public void pop() {  
        storage.removeFirst();  
    }  
  
    public boolean empty() {  
        return storage.isEmpty();  
    }  
  
    @Override  
    public String toString() {  
        return storage.toString();  
    }  
  
}  

```





## Java消息队列

https://www.cnblogs.com/doit8791/p/7759806.html

### 一、消息队列概述

```doc
消息队列中间件是分布式系统中重要的组件，主要解决应用解耦，异步消息，流量削锋等问题，实现高性能，高可用，可伸缩和最终一致性架构。目前使用较多的消息队列有ActiveMQ，RabbitMQ，ZeroMQ，Kafka，MetaMQ，RocketMQ
```



### 二、消息队列应用场景

以下介绍消息队列在实际应用中常用的使用场景。异步处理，应用解耦，流量削锋和消息通讯四个场景。



#### 2.1异步处理

场景说明：用户注册后，需要发注册邮件和注册短信。传统的做法有两种 1.串行的方式；2.并行方式
a、串行方式：将注册信息写入数据库成功后，发送注册邮件，再发送注册短信。以上三个任务全部完成后，返回给客户端。

b、并行方式：将注册信息写入数据库成功后，发送注册邮件的同时，发送注册短信。以上三个任务完成后，返回给客户端。与串行的差别是，并行的方式可以提高处理的时间。



假设三个业务节点每个使用50毫秒钟，不考虑网络等其他开销，则串行方式的时间是150毫秒，并行的时间可能是100毫秒。
因为CPU在单位时间内处理的请求数是一定的，假设CPU1秒内吞吐量是100次。则串行方式1秒内CPU可处理的请求量是7次（1000/150）。并行方式处理的请求量是10次（1000/100）
小结：如以上案例描述，传统的方式系统的性能（并发量，吞吐量，响应时间）会有瓶颈。如何解决这个问题呢？



引入消息队列，将不是必须的业务逻辑，异步处理。改造后的架构如下

按照以上约定，用户的响应时间相当于是注册信息写入数据库的时间，也就是50毫秒。注册邮件，发送短信写入消息队列后，直接返回，因此写入消息队列的速度很快，基本可以忽略，因此用户的响应时间可能是50毫秒。因此架构改变后，系统的吞吐量提高到每秒20 QPS。比串行提高了3倍，比并行提高了两倍。





#### 2.2应用解耦

场景说明：用户下单后，订单系统需要通知库存系统。传统的做法是，订单系统调用库存系统的接口。

传统模式的缺点：假如库存系统无法访问，则订单减库存将失败，从而导致订单失败，订单系统与库存系统耦合。

如何解决以上问题呢？引入应用消息队列后的方案，

订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功
库存系统：订阅下单的消息，采用拉/推的方式，获取下单信息，库存系统根据下单信息，进行库存操作
假如：在下单时库存系统不能正常使用。也不影响正常下单，因为下单后，订单系统写入消息队列就不再关心其他的后续操作了。实现订单系统与库存系统的应用解耦。

#### 2.3流量削锋

流量削锋也是消息队列中的常用场景，一般在秒杀或团抢活动中使用广泛。
应用场景：秒杀活动，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列。
a、可以控制活动的人数
b、可以缓解短时间内高流量压垮应用

用户的请求，服务器接收后，首先写入消息队列。假如消息队列长度超过最大数量，则直接抛弃用户请求或跳转到错误页面。
秒杀业务根据消息队列中的请求信息，再做后续处理

#### 2.4日志处理

日志处理是指将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题。

日志采集客户端，负责日志数据采集，定时写受写入Kafka队列
Kafka消息队列，负责日志数据的接收，存储和转发
日志处理应用：订阅并消费kafka队列中的日志数据 

#### 2.5消息通讯

消息通讯是指，消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等
点对点通讯：

客户端A（发消息）和客户端B（收消息）使用同一消息队列，进行消息通讯。



聊天室通讯：

客户端A，客户端B，客户端N订阅同一主题，进行消息发布和接收。实现类似聊天室效果。



以上实际是消息队列的两种消息模式，点对点或发布订阅模式。

### 三、消息中间件示例

3.1电商系统



消息队列采用高可用，可持久化的消息中间件。比如Active MQ，Rabbit MQ，Rocket Mq。

消息队列采用高可用，可持久化的消息中间件。比如Active MQ，Rabbit MQ，Rocket Mq。
（1）应用将主干逻辑处理完成后，写入消息队列。消息发送是否成功可以开启消息的确认模式。（消息队列返回消息接收成功状态后，应用再返回，这样保障消息的完整性）
（2）扩展流程（发短信，配送处理）订阅队列消息。采用推或拉的方式获取消息并处理。
（3）消息将应用解耦的同时，带来了数据一致性问题，可以采用最终一致性方式解决。比如主数据写入数据库，扩展应用根据消息队列，并结合数据库方式实现基于消息队列的后续处理。



分为Zookeeper注册中心，日志收集客户端，Kafka集群和Storm集群（OtherApp）四部分组成。
Zookeeper注册中心，提出负载均衡和地址查找服务
日志收集客户端，用于采集应用系统的日志，并将数据推送到kafka队列
Kafka集群：接收，路由，存储，转发等消息处理
Storm集群：与OtherApp处于同一级别，采用拉的方式消费队列中的数据

### 四、JMS消息服务



讲消息队列就不得不提JMS 。JMS（JAVA Message Service，java消息服务）API是一个消息服务的标准/规范，允许应用程序组件基于JavaEE平台创建、发送、接收和读取消息。它使分布式通信耦合度更低，消息服务更加可靠以及异步性。
在EJB架构中，有消息bean可以无缝的与JM消息服务集成。在J2EE架构模式中，有消息服务者模式，用于实现消息与应用直接的解耦。

#### 4.1消息模型

在JMS标准中，有两种消息模型P2P（Point to Point）,Publish/Subscribe(Pub/Sub)。P2P模式包含三个角色：消息队列（Queue），发送者(Sender)，接收者(Receiver)。每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，直到他们被消费或超时。

P2P的特点
每个消息只有一个消费者（Consumer）(即一旦被消费，消息就不再在消息队列中)
发送者和接收者之间在时间上没有依赖性，也就是说当发送者发送了消息之后，不管接收者有没有正在运行，它不会影响到消息被发送到队列
接收者在成功接收消息之后需向队列应答成功 
如果希望发送的每个消息都会被成功处理的话，那么需要P2P模式。

##### 4.1.2 Pub/Sub模式

包含三个角色主题（Topic），发布者（Publisher），订阅者（Subscriber） 多个发布者将消息发送到Topic，系统将这些消息传递给多个订阅者。

Pub/Sub的特点
每个消息可以有多个消费者
发布者和订阅者之间有时间上的依赖性。针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息
为了消费消息，订阅者必须保持运行的状态
为了缓和这样严格的时间相关性，JMS允许订阅者创建一个可持久化的订阅。这样，即使订阅者没有被激活（运行），它也能接收到发布者的消息。
如果希望发送的消息可以不被做任何处理、或者只被一个消息者处理、或者可以被多个消费者处理的话，那么可以采用Pub/Sub模型。

#### 4.2消息消费

在JMS中，消息的产生和消费都是异步的。对于消费来说，JMS的消息者可以通过两种方式来消费消息。
（1）同步
订阅者或接收者通过receive方法来接收消息，receive方法在接收到消息之前（或超时之前）将一直阻塞；

（2）异步
订阅者或接收者可以注册为一个消息监听器。当消息到达之后，系统自动调用监听器的onMessage方法。

JNDI：Java命名和目录接口,是一种标准的Java命名系统接口。可以在网络上查找和访问服务。通过指定一个资源名称，该名称对应于数据库或命名服务中的一个记录，同时返回资源连接建立所必须的信息。
JNDI在JMS中起到查找和访问发送目标或消息来源的作用。

### 五、常用消息队列

一般商用的容器，比如WebLogic，JBoss，都支持JMS标准，开发上很方便。但免费的比如Tomcat，Jetty等则需要使用第三方的消息中间件。本部分内容介绍常用的消息中间件（Active MQ,Rabbit MQ，Zero MQ,Kafka）以及他们的特点。

#### 5.1 ActiveMQ



ActiveMQ默认启动时，启动了内置的jetty服务器，提供一个用于监控ActiveMQ的admin应用。 
http://127.0.0.1:8161/admin/

默认账号：admin

　　密码：admin

ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现，尽管JMS规范出台已经是很久的事情了，但是JMS在当今的J2EE应用中间仍然扮演着特殊的地位。

ActiveMQ特性如下：
⒈ 多种语言和协议编写客户端。语言: Java,C,C++,C#,Ruby,Perl,Python,PHP。应用协议： OpenWire,Stomp REST,WS Notification,XMPP,AMQP
⒉ 完全支持JMS1.1和J2EE 1.4规范 （持久化，XA消息，事务)
⒊ 对Spring的支持，ActiveMQ可以很容易内嵌到使用Spring的系统里面去，而且也支持Spring2.0的特性
⒋ 通过了常见J2EE服务器（如 Geronimo,JBoss 4,GlassFish,WebLogic)的测试，其中通过JCA 1.5 resource adaptors的配置，可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上
⒌ 支持多种传送协议：in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA
⒍ 支持通过JDBC和journal提供高速的消息持久化
⒎ 从设计上保证了高性能的集群，客户端-服务器，点对点
⒏ 支持Ajax
⒐ 支持与Axis的整合
⒑ 可以很容易得调用内嵌JMS provider，进行测试。





#### 5.2 Kafka

Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群机来提供实时的消费。

Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：
通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。（文件追加的方式写入数据，过期的数据定期删除）
高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息
支持通过Kafka服务器和消费机集群来分区消息
支持Hadoop并行数据加载
Kafka相关概念
Broker
Kafka集群包含一个或多个服务器，这种服务器被称为broker[5]
Topic
每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
Partition
Parition是物理上的概念，每个Topic包含一个或多个Partition.
Producer
负责发布消息到Kafka broker
Consumer
消息消费者，向Kafka broker读取消息的客户端。
Consumer Group
每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。
一般应用在大数据日志处理或对实时性（少量延迟），可靠性（少量丢数据）要求稍低的场景使用。





2018/4/2   星期一  多云

# 详解Java消息队列-Spring整合ActiveMq

### 一、Java消息队列----JMS概述

#### 1.什么是JMS?

JMS即Java消息服务（Java Message Service）应用程序接口。

```doc
是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。

我们可以简单的理解：两个应用程序之间需要进行通信，我们使用一个JMS服务，进行中间的转发，通过JMS 的使用，我们可以解除两个程序之间的耦合。

```

#### 2、JMS的优势

Asynchronous（异步）

JMS 原本就是一个异步的消息服务，客户端获取消息的时候，不需要主动发送请求，消息会自动发送给可用的客户端



Reliable（可靠）

JMS保证消息只会递送一次。大家都遇到过重复创建消息问题，而JMS能帮你避免该问题。



#### 3、JMS的消息模型

　　　JMS具有两种通信模式：

　　　　　　1、Point-to-Point Messaging Domain （点对点）

　　　　　　2、Publish/Subscribe Messaging Domain （发布/订阅模式）



##### (1)、Point-to-Point Messaging Domain（点对点通信模型）

在点对点通信模式中，应用程序由消息队列，发送方，接收方组成。

每个消息都被发送到一个特定的队列，接收者从队列中获取消息。队列保留着消息，直到他们被消费或超时。

每个消息只要一个消费者
发送者和接收者在时间上是没有时间的约束，也就是说发送者在发送完消息之后，不管接收者有没有接受消息，都不会影响发送方发送消息到消息队列中。

发送方不管是否在发送消息，接收方都可以从消息队列中去到消息。

接收方在接收完消息之后，需要向消息队列应答成功。

##### (2)、Publish/Subscribe Messaging Domain（发布/订阅通信模型）

在发布/订阅消息模型中，发布者发布一个消息，该消息通过topic传递给所有的客户端。该模式下，发布者与订阅者都是匿名的，即发布者与订阅者都不知道对方是谁。并且可以动态的发布与订阅Topic。Topic主要用于保存和传递消息，且会一直保存消息直到消息被传递给客户端。

一个消息可以传递个多个订阅者（即：一个消息可以有多个接受方）
发布者与订阅者具有时间约束，针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息，而且为了消费消息，订阅者必须保持运行的状态。
为了缓和这样严格的时间相关性，JMS允许订阅者创建一个可持久化的订阅。这样，即使订阅者没有被激活（运行），它也能接收到发布者的消息。

#### 4、JMS接收消息

JMS中，消息的产生和消息是异步的。对于消费来说，JMS的消息者可以通过两种方式来消费消息。

##### （1）、同步（Synchronous）

　　　　　　　　在同步消费信息模式模式中，订阅者/接收方通过调用 receive（）方法来接收消息。在receive（）方法中，线程会阻塞直到消息到达或者到指定时间后消息仍未到达。

##### （2）、异步（Asynchronous）

　　　　　　　　使用异步方式接收消息的话，消息订阅者需注册一个消息监听者，类似于事件监听器，只要消息到达，JMS服务提供者会通过调用监听器的onMessage()递送消息。



#### 5、JMS编程模型　

1管理对象（Administered objects）-连接工厂（Connection Factories）和目的地（Destination）
2连接对象（Connections）
3会话（Sessions）
4消息生产者（Message Producers）
5消息消费者（Message Consumers）
6消息监听者（Message Listeners）



```doc
（1）、Connection Factories

　　　　　　　　创建Connection对象的工厂，针对两种不同的jms消息模型，分别有QueueConnectionFactory和TopicConnectionFactory两种。可以通过JNDI来查找ConnectionFactory对象。客户端使用一个连接工厂对象连接到JMS服务提供者，它创建了JMS服务提供者和客户端之间的连接。JMS客户端（如发送者或接受者）会在JNDI名字空间中搜索并获取该连接。使用该连接，客户端能够与目的地通讯，往队列或话题发送/接收消息。
QueueConnectionFactory queueConnFactory = (QueueConnectionFactory) initialCtx.lookup ("primaryQCF");
Queue purchaseQueue = (Queue) initialCtx.lookup ("Purchase_Queue");
Queue returnQueue = (Queue) initialCtx.lookup ("Return_Queue");

（2）、Destination

 　　　　　　　　目的地指明消息被发送的目的地以及客户端接收消息的来源。JMS使用两种目的地，队列和话题。如下代码指定了一个队列和话题：
 　　　　　　　　创建一个队列Session：
 　　　QueueSession ses = con.createQueueSession (false, Session.AUTO_ACKNOWLEDGE);  //get the Queue object  
Queue t = (Queue) ctx.lookup ("myQueue");  //create QueueReceiver  
QueueReceiver receiver = ses.createReceiver(t); 
　创建一个Topic Session：

QueueSession ses = con.createQueueSession (false, Session.AUTO_ACKNOWLEDGE);  //get the Queue object  
Queue t = (Queue) ctx.lookup ("myQueue");  //create QueueReceiver  
QueueReceiver receiver = ses.createReceiver(t); 

（3）、Connection

　　　　　　Connection表示在客户端和JMS系统之间建立的链接（对TCP/IP socket的包装）。Connection可以产生一个或多个Session。跟ConnectionFactory一样，Connection也有两种类型：QueueConnection和TopicConnection。

　　　　　　连接对象封装了与JMS提供者之间的虚拟连接，如果我们有一个ConnectionFactory对象，可以使用它来创建一个连接。
Connection connection = connectionFactory.createConnection();


Session 是我们对消息进行操作的接口，可以通过session创建生产者、消费者、消息等。Session 提供了事务的功能，如果需要使用session发送/接收多个消息时，可以将这些发送/接收动作放到一个事务中。

　　　　　　我们可以在连接创建完成之后创建session：

Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
　　　　　　这里面提供了参数两个参数，第一个参数是是否支持事务，第二个是事务的类型

（5）、Producter

　　　　　　消息生产者由Session创建，用于往目的地发送消息。生产者实现MessageProducer接口，我们可以为目的地、队列或话题创建生产者；

　　　　　　

MessageProducer producer = session.createProducer(dest);
MessageProducer producer = session.createProducer(queue);
MessageProducer producer = session.createProducer(topic);


　（6）、Consumer

　　　　　　消息消费者由Session创建，用于接收被发送到Destination的消息。

　

MessageConsumer consumer = session.createConsumer(dest);
MessageConsumer consumer = session.createConsumer(queue);
MessageConsumer consumer = session.createConsumer(topic);



（7）、MessageListener

　　　　　　消息监听器。如果注册了消息监听器，一旦消息到达，将自动调用监听器的onMessage方法。EJB中的MDB（Message-Driven Bean）就是一种MessageListener。

 
```



2018/04/03   星期二     晴

## Java 多线程

### 一、概述

一个线程不能独立的存在，它必须是进程的一部分。一个进程一直运行，直到所有的非守护线程都结束运行后才能结束。

多线程能满足程序员编写高效率的程序来达到充分利用 CPU 的目的。

```doc
新建状态:
使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。

就绪状态:
当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。

运行状态:
如果就绪状态的线程获取 CPU 资源，就可以执行 run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。

阻塞状态:
如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：

等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。

同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。

其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

死亡状态:
一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

线程的优先级
每一个 Java 线程都有一个优先级，这样有助于操作系统确定线程的调度顺序。

Java 线程的优先级是一个整数，其取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）。

默认情况下，每一个线程都会分配一个优先级 NORM_PRIORITY（5）。

具有较高优先级的线程对程序更重要，并且应该在低优先级的线程之前分配处理器资源。但是，线程优先级不能保证线程执行的顺序，而且非常依赖于平台。







```

### 二、创建一个线程

Java 提供了三种创建线程的方法：

通过实现 Runnable 接口；
通过继承 Thread 类本身；
通过 Callable 和 Future 创建线程。

通过Thread类

```java
package com.ahhf.ljxbw.arvinmq.test;

public class ArvinThread extends Thread {

	private String message;

	public ArvinThread(String message) {
		this.message = message;
	}

	public void run() {
		for (int i = 0; i < 5; i++) {
			System.out.println(message + "运行  :  " + i);
			try {
				
				Thread.sleep((int) Math.random() * 10);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

}

public static void main(String[] args) {
		ArvinThread mTh1 = new ArvinThread("A");
		ArvinThread mTh2 = new ArvinThread("B");
		mTh1.start();
		mTh2.start();

	}

/**
 * 程序启动运行main时候，java虚拟机启动一个进程，主线程main在main()调用时候被创建。随着调用MitiSay的两个对象的start方法，另外两个线程也启动了，这样，整个应用就在多线程下运行。
 
注意：start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的。
从程序运行的结果可以发现，多线程程序是乱序执行。因此，只有乱序执行的代码才有必要设计为多线程。
Thread.sleep()方法调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留出一定时间给其他线程执行的机会。
实际上所有的多线程代码执行顺序都是不确定的，每次执行的结果都是随机的。



但是start方法重复调用的话，会出现java.lang.IllegalThreadStateException异常。
Thread1 mTh1=new Thread1("A");  
Thread1 mTh2=mTh1;  
mTh1.start();  
mTh2.start();  
输出：

Exception in thread "main" java.lang.IllegalThreadStateException
    at java.lang.Thread.start(Unknown Source)
    at com.multithread.learning.Main.main(Main.java:31)
A运行  :  0
A运行  :  1
A运行  :  2
A运行  :  3
A运行  :  4








 *
 */
```



通过实现 Runnable 接口来创建线程
创建一个线程，最简单的方法是创建一个实现 Runnable 接口的类。

为了实现 Runnable，一个类只需要执行一个方法调用 run()

```java
package com.ahhf.ljxbw.arvinmq.test;

public class ArvinThread2 implements Runnable {
	private String threadName;

	public ArvinThread2(String threadName) {
		this.threadName = threadName;
	}

	public void run() {
		// TODO Auto-generated method stub

		for (int i = 0; i < 10; i++) {
			System.out.println(threadName + (i + 1) + "跑起来了————————");
			try {
				Thread.sleep((int) Math.random() * 10);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}  
		}
	}

}
public static void main(String[] args) {
 		new Thread(new ArvinThread2("张三")).start();
		new Thread(new ArvinThread2("李四")).start();

}


/**
Thread2类通过实现Runnable接口，使得该类有了多线程类的特征。run（）方法是多线程程序的一个约定。所有的多线程代码都在run方法里面。Thread类实际上也是实现了Runnable接口的类。
在启动的多线程的时候，需要先通过Thread类的构造方法Thread(Runnable target) 构造出对象，然后调用Thread对象的start()方法来运行多线程代码。
实际上所有的多线程代码都是通过运行Thread的start()方法来运行的。因此，不管是扩展Thread类还是实现Runnable接口来实现多线程，最终还是通过Thread的对象的API来控制线程的，熟悉Thread类的API是进行多线程编程的基础。

*/
```



### 三、Thread和Runnable的区别

如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

总结：

实现Runnable接口比继承Thread类所具有的优势：

1）：适合多个相同的程序代码的线程去处理同一个资源

2）：可以避免java中的单继承的限制

3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

4）：线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

 

提醒一下大家：main方法其实也是一个线程。在java中所以的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。

在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个ＪＶＭ，每一个ｊＶＭ实习在就是在操作系统中启动了一个进程。

### 四、线程状态转换

```doc
1、新建状态（New）：新创建了一个线程对象。
2、就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3、运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4、阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
（一）、等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。(wait会释放持有的锁)
（二）、同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
（三）、其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。（注意,sleep是不会释放持有的锁）
5、死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。
```

### 五、线程调度

#### 1、调整线程优先级

```doc


1、调整线程优先级：Java线程有优先级，优先级高的线程会获得较多的运行机会。
 
Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：
static int MAX_PRIORITY
          线程可以具有的最高优先级，取值为10。
static int MIN_PRIORITY
          线程可以具有的最低优先级，取值为1。
static int NORM_PRIORITY
          分配给线程的默认优先级，取值为5。
 
Thread类的setPriority()和getPriority()方法分别用来设置和获取线程的优先级。
每个线程都有默认的优先级。主线程的默认优先级为Thread.NORM_PRIORITY。
线程的优先级有继承关系，比如A线程中创建了B线程，那么B将和A具有相同的优先级。
JVM提供了10个线程优先级，但与常见的操作系统都不能很好的映射。如果希望程序能移植到各个操作系统中，应该仅仅使用Thread类有以下三个静态常量作为优先级，这样能保证同样的优先级采用了同样的调度方式。
```



#### 2、线程睡眠

Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

#### 3、线程等待

Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

#### 4、线程让步

Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

#### 5、线程加入

join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

#### 6、线程唤醒

Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。
 注意：Thread中suspend()和resume()两个方法在JDK1.5中已经废除，不再介绍。因为有死锁倾向。



### 六、常用函数说明

①sleep(long millis): 在指定的毫秒数内让当前正在执行的线程休眠（暂停执行）

②join():指等待t线程终止。
使用方式。
join是Thread类的一个方法，启动线程后直接调用，即join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

Thread t = new AThread(); t.start(); t.join();  

#### 为什么要用join()方法

在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

```java
// 不加join。
package com.multithread.join;  
class Thread1 extends Thread{  
    private String name;  
    public Thread1(String name) {  
        super(name);  
       this.name=name;  
    }  
    public void run() {  
        System.out.println(Thread.currentThread().getName() + " 线程运行开始!");  
        for (int i = 0; i < 5; i++) {  
            System.out.println("子线程"+name + "运行 : " + i);  
            try {  
                sleep((int) Math.random() * 10);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
        System.out.println(Thread.currentThread().getName() + " 线程运行结束!");  
    }  
}  
  
public class Main {  
  
    public static void main(String[] args) {  
        System.out.println(Thread.currentThread().getName()+"主线程运行开始!");  
        Thread1 mTh1=new Thread1("A");  
        Thread1 mTh2=new Thread1("B");  
        mTh1.start();  
        mTh2.start();  
        System.out.println(Thread.currentThread().getName()+ "主线程运行结束!");  
  
    }  
  
}  

输出结果：
main主线程运行开始!
main主线程运行结束!
B 线程运行开始!
子线程B运行 : 0
A 线程运行开始!
子线程A运行 : 0
子线程B运行 : 1
子线程A运行 : 1
子线程A运行 : 2
子线程A运行 : 3
子线程A运行 : 4
A 线程运行结束!
子线程B运行 : 2
子线程B运行 : 3
子线程B运行 : 4
B 线程运行结束!
发现主线程比子线程早结束
  
  
 
  
  
  
 //加join 
  public class Main {  
  
    public static void main(String[] args) {  
        System.out.println(Thread.currentThread().getName()+"主线程运行开始!");  
        Thread1 mTh1=new Thread1("A");  
        Thread1 mTh2=new Thread1("B");  
        mTh1.start();  
        mTh2.start();  
        try {  
            mTh1.join();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        try {  
            mTh2.join();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(Thread.currentThread().getName()+ "主线程运行结束!");  
  
    }  
  
}  
运行结果：
main主线程运行开始!
A 线程运行开始!
子线程A运行 : 0
B 线程运行开始!
子线程B运行 : 0
子线程A运行 : 1
子线程B运行 : 1
子线程A运行 : 2
子线程B运行 : 2
子线程A运行 : 3
子线程B运行 : 3
子线程A运行 : 4
子线程B运行 : 4
A 线程运行结束!
主线程一定会等子线程都结束了才结束
```



③yield():暂停当前正在执行的线程对象，并执行其他线程。

        Thread.yield()方法作用是：暂停当前正在执行的线程对象，并执行其他线程。
         yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。可看上面的图。

```java
package com.multithread.yield;  
class ThreadYield extends Thread{  
    public ThreadYield(String name) {  
        super(name);  
    }  
   
    @Override  
    public void run() {  
        for (int i = 1; i <= 50; i++) {  
            System.out.println("" + this.getName() + "-----" + i);  
            // 当i为30时，该线程就会把CPU时间让掉，让其他或者自己的线程执行（也就是谁先抢到谁执行）  
            if (i ==30) {  
                this.yield();  
            }  
        }  
      
}  
}  
  
public class Main {  
  
    public static void main(String[] args) {  
          
        ThreadYield yt1 = new ThreadYield("张三");  
        ThreadYield yt2 = new ThreadYield("李四");  
        yt1.start();  
        yt2.start();  
    }  
  
}  

运行结果：
第一种情况：李四（线程）当执行到30时会CPU时间让掉，这时张三（线程）抢到CPU时间并执行。

第二种情况：李四（线程）当执行到30时会CPU时间让掉，这时李四（线程）抢到CPU时间并执行。
```

#### sleep()和yield()的区别

        sleep()和yield()的区别):sleep()使当前线程进入停滞状态，所以执行sleep()的线程在指定的时间内肯定不会被执行；yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。
        sleep 方法使当前运行中的线程睡眼一段时间，进入不可运行状态，这段时间的长短是由程序设定的，yield 方法使当前线程让出 CPU 占有权，但让出的时间是不可设定的。实际上，yield()方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU  的占有权交给此线程，否则，继续运行原来的线程。所以yield()方法称为“退让”，它把运行机会让给了同等优先级的其他线程
       另外，sleep 方法允许较低优先级的线程获得运行机会，但 yield()  方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得 CPU 占有权。在一个运行系统中，如果较高优先级的线程没有调用 sleep 方法，又没有受到 I\O 阻塞，那么，较低优先级线程只能等待所有较高优先级的线程运行结束，才有机会运行。 


 ④setPriority(): 更改线程的优先级。

```doc
    
　　　　MIN_PRIORITY = 1
  　　   NORM_PRIORITY = 5
           MAX_PRIORITY = 10

用法：
Thread4 t1 = new Thread4("t1");
Thread4 t2 = new Thread4("t2");
t1.setPriority(Thread.MAX_PRIORITY);
t2.setPriority(Thread.MIN_PRIORITY);
```

⑤interrupt():不要以为它是中断某个线程！它只是线线程发送一个中断信号，让线程在无限等待时（如死锁时）能抛出抛出，从而结束线程，但是如果你吃掉了这个异常，那么这个线程还是不会中断的！



⑥wait()

Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在synchronized(Obj){...}语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。相应的notify()就是对对象锁的唤醒操作。但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。



#### wait和sleep区别

共同点： 

1. 他们都是在多线程的环境下，都可以在程序的调用处阻塞指定的毫秒数，并返回。 
2. wait()和sleep()都可以通过interrupt()方法 打断线程的暂停状态 ，从而使线程立刻抛出InterruptedException。 
   如果线程A希望立即结束线程B，则可以对线程B对应的Thread实例调用interrupt方法。如果此刻线程B正在wait/sleep /join，则线程B会立刻抛出InterruptedException，在catch() {} 中直接return即可安全地结束线程。 
   需要注意的是，InterruptedException是线程自己从内部抛出的，并不是interrupt()方法抛出的。对某一线程调用 interrupt()时，如果该线程正在执行普通的代码，那么该线程根本就不会抛出InterruptedException。但是，一旦该线程进入到 wait()/sleep()/join()后，就会立刻抛出InterruptedException 。 
   不同点： 
3. Thread类的方法：sleep(),yield()等 
   Object的方法：wait()和notify()等 
4. 每个对象都有一个锁来控制同步访问。Synchronized关键字可以和对象的锁交互，来实现线程的同步。 
   sleep方法没有释放锁，而wait方法释放了锁，使得其他线程可以使用同步控制块或者方法。 
5. wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用 
6. sleep必须捕获异常，而wait，notify和notifyAll不需要捕获异常
   所以sleep()和wait()方法的最大区别是：
   　　　　sleep()睡眠时，保持对象锁，仍然占有该锁；
   　　　　而wait()睡眠时，释放对象锁。
   　　但是wait()和sleep()都可以通过interrupt()方法打断线程的暂停状态，从而使线程立刻抛出InterruptedException（但不建议使用该方法）。
   sleep（）方法
   sleep()使当前线程进入停滞状态（阻塞当前线程），让出CUP的使用、目的是不让当前线程独自霸占该进程所获的CPU资源，以留一定时间给其他线程执行的机会;
   　　 sleep()是Thread类的Static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用Sleep()方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）。
   　　在sleep()休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级。 
   wait（）方法
   wait()方法是Object类里的方法；当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，wait(long timeout)超时时间到后还需要返还对象锁）；其他线程可以访问；
   　　wait()使用notify或者notifyAlll或者指定睡眠时间来唤醒当前等待池中的线程。
   　　wiat()必须放在synchronized block中，否则会在program runtime时扔出”java.lang.IllegalMonitorStateException“异常。



### 七、常见线程名词解释

主线程：JVM调用程序main()所产生的线程。
当前线程：这个是容易混淆的概念。一般指通过Thread.currentThread()来获取的进程。
后台线程：指为其他线程提供服务的线程，也称为守护线程。JVM的垃圾回收线程就是一个后台线程。用户线程和守护线程的区别在于，是否等待主线程依赖于主线程结束而结束
前台线程：是指接受后台线程服务的线程，其实前台后台线程是联系在一起，就像傀儡和幕后操纵者一样的关系。傀儡是前台线程、幕后操纵者是后台线程。由前台线程创建的线程默认也是前台线程。可以通过isDaemon()和setDaemon()方法来判断和设置一个线程是否为后台线程。
线程类的一些常用方法： 

　　sleep(): 强迫一个线程睡眠Ｎ毫秒。 
　　isAlive(): 判断一个线程是否存活。 
　　join(): 等待线程终止。 
　　activeCount(): 程序中活跃的线程数。 
　　enumerate(): 枚举程序中的线程。 
    currentThread(): 得到当前线程。 
　　isDaemon(): 一个线程是否为守护线程。 
　　setDaemon(): 设置一个线程为守护线程。(用户线程和守护线程的区别在于，是否等待主线程依赖于主线程结束而结束) 
　　setName(): 为线程设置一个名称。 
　　wait(): 强迫一个线程等待。 
　　notify(): 通知一个线程继续运行。 
　　setPriority(): 设置一个线程的优先级。



### 八、线程同步

#### 1、synchronized关键字的作用域有二种： 

1）是某个对象实例内，synchronized aMethod(){}可以防止多个线程同时访问这个对象的synchronized方法（如果一个对象有多个synchronized方法，只要一个线程访问了其中的一个synchronized方法，其它线程不能同时访问这个对象中任何一个synchronized方法）。这时，不同的对象实例的synchronized方法是不相干扰的。也就是说，其它线程照样可以同时访问相同类的另一个对象实例中的synchronized方法； 

2）是某个类的范围，synchronized static aStaticMethod{}防止多个线程同时访问这个类中的synchronized static 方法。它可以对类的所有对象实例起作用。 

#### 2、用于方法中的某个区块中

除了方法前用synchronized关键字，synchronized关键字还可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。用法是: synchronized(this){/*区块*/}，它的作用域是当前对象； 

#### 3、synchronized关键字是不能继承的

也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法； 

#### 4、小结

总的说来，synchronized关键字可以作为函数的修饰符，也可作为函数内的语句，也就是平时说的同步方法和同步语句块。如果再细的分类，synchronized可作用于instance变量、object reference（对象引用）、static函数和class literals(类名称字面常量)身上。

在进一步阐述之前，我们需要明确几点：

A．无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象，而不是把一段代码或函数当作锁――而且同步方法很可能还会被其他线程的对象访问。

B．每个对象只有一个锁（lock）与之相关联。

C．实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

1、线程同步的目的是为了保护多个线程访问一个资源时对资源的破坏。
2、线程同步方法是通过锁来实现，每个对象都有切仅有一个锁，这个锁与一个特定的对象关联，线程一旦获取了对象锁，其他访问该对象的线程就无法再访问该对象的其他非同步方法。
3、对于静态同步方法，锁是针对这个类的，锁对象是该类的Class对象。静态和非静态方法的锁互不干预。一个线程获得锁，当在一个同步方法中访问另外对象上的同步方法时，会获取这两个对象锁。
4、对于同步，要时刻清醒在哪个对象上同步，这是关键。
5、编写线程安全的类，需要时刻注意对多个线程竞争访问资源的逻辑和安全做出正确的判断，对“原子”操作做出分析，并保证原子操作期间别的线程无法访问竞争资源。
6、当多个线程等待一个对象锁时，没有获取到锁的线程将发生阻塞。
7、死锁是线程间相互等待锁锁造成的，在实际中发生的概率非常的小。真让你写个死锁程序，不一定好使，呵呵。但是，一旦程序发生死锁，程序将死掉。



### 九、线程数据传递

#### 9.1、通过构造方法传递数据 

在创建线程时，必须要建立一个Thread类的或其子类的实例。因此，我们不难想到在调用start方法之前通过线程类的构造方法将数据传入线程。并将传入的数据使用类变量保存起来，以便线程使用(其实就是在run方法中使用)。下面的代码演示了如何通过构造方法来传递数据： 

```java
package mythread;   
public class MyThread1 extends Thread   
{   
private String name;   
public MyThread1(String name)   
{   
this.name = name;   
}   
public void run()   
{   
System.out.println("hello " + name);   
}   
public static void main(String[] args)   
{   
Thread thread = new MyThread1("world");   
thread.start();   
}   
}   
```



由于这种方法是在创建线程对象的同时传递数据的，因此，在线程运行之前这些数据就就已经到位了，这样就不会造成数据在线程运行后才传入的现象。如果要传递更复杂的数据，可以使用集合、类等数据结构。使用构造方法来传递数据虽然比较安全，但如果要传递的数据比较多时，就会造成很多不便。由于Java没有默认参数，要想实现类似默认参数的效果，就得使用重载，这样不但使构造方法本身过于复杂，又会使构造方法在数量上大增。因此，要想避免这种情况，就得通过类方法或类变量来传递数据。 



#### 9.2、通过变量和方法传递数据 

向对象中传入数据一般有两次机会，第一次机会是在建立对象时通过构造方法将数据传入，另外一次机会就是在类中定义一系列的public的方法或变量（也可称之为字段）。然后在建立完对象后，通过对象实例逐个赋值。下面的代码是对MyThread1类的改版，使用了一个setName方法来设置 name变量： 

```java
package mythread;   
public class MyThread2 implements Runnable   
{   
private String name;   
public void setName(String name)   
{   
this.name = name;   
}   
public void run()   
{   
System.out.println("hello " + name);   
}   
public static void main(String[] args)   
{   
MyThread2 myThread = new MyThread2();   
myThread.setName("world");   
Thread thread = new Thread(myThread);   
thread.start();   
}   
}   
```

#### 9.3、通过回调函数传递数据 

上面讨论的两种向线程中传递数据的方法是最常用的。但这两种方法都是main方法中主动将数据传入线程类的。这对于线程来说，是被动接收这些数据的。然而，在有些应用中需要在线程运行的过程中动态地获取数据，如在下面代码的run方法中产生了3个随机数，然后通过Work类的process方法求这三个随机数的和，并通过Data类的value将结果返回。从这个例子可以看出，在返回value之前，必须要得到三个随机数。也就是说，这个 value是无法事先就传入线程类的。 

```java
package mythread;   
class Data   
{   
public int value = 0;   
}   

class Work   
{   
public void process(Data data, Integer numbers)   
{   
for (int n : numbers)   
{   
data.value += n;   
}   
}   
}

public class MyThread3 extends Thread   
{   
private Work work;   
public MyThread3(Work work)   
{   
this.work = work;   
}   
public void run()   
{   
java.util.Random random = new java.util.Random();   
Data data = new Data();   
int n1 = random.nextInt(1000);   
int n2 = random.nextInt(2000);   
int n3 = random.nextInt(3000);   
work.process(data, n1, n2, n3); // 使用回调函数   
System.out.println(String.valueOf(n1) + "+" + String.valueOf(n2) + "+"   
+ String.valueOf(n3) + "=" + data.value);   
}   
public static void main(String[] args)   
{   
Thread thread = new MyThread3(new Work());   
thread.start();   
}   
}   
```

### 十、线程同步 synchronized 同步代码块 同步方法 同步锁

#### 1、同步代码块

1.为了解决并发操作可能造成的异常，java的多线程支持引入了同步监视器来解决这个问题，使用同步监视器的通用方法就是同步代码块。其语法如下：
synchronized(obj){
//同步代码块
}
其中obj就是同步监视器，它的含义是：线程开始执行同步代码块之前，必须先获得对同步监视器的锁定。任何时刻只能有一个线程可以获得对同步监视器的锁定，当同步代码块执行完成后，该线程会释放对该同步监视器的锁定。虽然java程序允许使用任何对象作为同步监视器，但 是同步监视器的目的就是为了阻止两个线程对同一个共享资源进行并发访问，因此通常推荐使用可能被并发访问的共享资源充当同步监视器。

example:

```java
public class Account {
    private String accountNo;
    private double balance;
    public Account(String accountNo,double balance){
        this.accountNo=accountNo;
        this.balance=balance;
    }
 
    public double getBalance() {
        return balance;
    }
 
    public void setBalance(double balance) {
        this.balance = balance;
    }
 
    public String getAccountNo() {
        return accountNo;
    }
 
    public void setAccountNo(String accountNo) {
        this.accountNo = accountNo;
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
 
        Account account = (Account) o;
 
        return accountNo.equals(account.accountNo);
 
    }
 
    @Override
    public int hashCode() {
        return accountNo.hashCode();
    }
}




public class DrawThread extends Thread {
    private Account account;
    private double drawAmount;
 
    public DrawThread(String name, Account account, double drawAmount) {
        super(name);
        this.account = account;
        this.drawAmount = drawAmount;
    }
    public void run(){
        synchronized (account){
            if(account.getBalance()>=drawAmount){
                System.out.println(getName() + "取钱成功，吐出钞票： " + drawAmount);
                try{
                    Thread.sleep(1);
                }catch(InterruptedException ex){
                    ex.getStackTrace();
                }
                account.setBalance(account.getBalance()-drawAmount);
                System.out.println("\t余额为："+account.getBalance());
            }else{
                System.out.println(getName()+"取钱失败，余额不足");
            }
        }
    }
}


public class DrawTest {
    public static void main(String[] args){
        Account acct=new Account("1234567",1000);
        new DrawThread("甲",acct,800).start();
        new DrawThread("乙",acct,800).start();
    }
}

甲取钱成功，吐出钞票： 800.0
    余额为：200.0
乙取钱失败，余额不足
3.如果将DrawThread的同步去掉：
  
  会出现这些情况的结果：

乙取钱成功，吐出钞票： 800.0
甲取钱成功，吐出钞票： 800.0
    余额为：200.0
    余额为：-600.0
 
甲取钱成功，吐出钞票： 800.0
乙取钱成功，吐出钞票： 800.0
    余额为：200.0
    余额为：200.0

```

  程序使用synchronized将run()方法里的方法体修改成同步代码块，同步监视器就是account对象，这样的做法符合“加锁-修改-释放锁”的逻辑，这样就可以保证并发线程在任一时刻只有一个线程进入修改共享资源的代码区。多次运行，结果只有一个。



#### 2、同步方法

1.同步方法就是使用synchronized关键字修饰某个方法，这个方法就是同步方法。这个同步方法(非static方法)无须显式指定同步监视器，同步方法的同步监视器是this，也就是调用该方法的对象。通过同步方法可以非常方便的实现线程安全的类，线程安全的类有如下特征：
该类的对象可以方便的被多个线程安全的访问；
每个线程调用该对象的任意方法之后都能得到正确的结果；
每个线程调用该对象的任意方法之后，该对象状态依然能保持合理状态。
2.不可变类总是线程安全的，因为它的对象状态不可改变可变类需要额外的方法来保证其线程安全，在Account类中我们只需要把balance的方法变成同步方法即可。

```java
public class Account {
    private String accountNo;
    private double balance;
    public Account(String accountNo,double balance){
        this.accountNo=accountNo;
        this.balance=balance;
    }
 
    //因为账户余额不可以随便更改，所以只为balance提供getter方法
    public double getBalance() {
        return balance;
    }
 
    public String getAccountNo() {
        return accountNo;
    }
 
    public void setAccountNo(String accountNo) {
        this.accountNo = accountNo;
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
 
        Account account = (Account) o;
 
        return accountNo.equals(account.accountNo);
 
    }
 
    @Override
    public int hashCode() {
        return accountNo.hashCode();
    }
 
    //提供一个线程安全的draw()方法来完成取钱操作
    public synchronized void draw(double drawAmount){
        if(balance>=drawAmount){
            System.out.println(Thread.currentThread().getName()+"取钱成功！吐出钞票："+drawAmount);
            try{
                Thread.sleep(1);
            }catch (InterruptedException ex){
                ex.printStackTrace();
            }
            balance-=drawAmount;
            System.out.println("\t余额为："+balance);
        }else{
            System.out.println(Thread.currentThread().getName()+"取钱失败，余额不足");
        }
    }
}


public class DrawThread extends Thread {
    private Account account;
    private double drawAmount;
 
    public DrawThread(String name, Account account, double drawAmount) {
        super(name);
        this.account = account;
        this.drawAmount = drawAmount;
    }
    public void run(){
        account.draw(drawAmount);
    }
}


public class DrawTest {
    public static void main(String[] args){
        Account acct=new Account("1234567",1000);
        new DrawThread("甲",acct,800).start();
        new DrawThread("乙",acct,800).start();
    }
}
```

注意，synchronized可以修饰方法，修饰代码块，但是不能修饰构造器、成员变量等。在Account类中定义draw()方法，而不是直接在 run()方法中实现取钱逻辑，这种做法更符合面向对象规则。DDD设计方式，即Domain Driven Design(领域驱动设计)，认为每个类都应该是完备的领域对象，Account代表用户账户，就应该提供用户账户的相关方法。通过draw()方法来执行取钱操作，而不是直接将setBalance()方法暴露出来任人操作。

但是，可变类的线程安全是以降低程序的运行效率为代价的，不要对线程安全类的所有方法都进行同步，只对那些会改变竞争资源(共享资源)的方法进行同步。同时，可变类有两种运行环境：单线程环境和多线程环境， 则应该为可变类提供两种版本，即线程安全版本和线程不安全版本。如JDK提供的StringBuilder在单线程环境下保证更好的性能，StringBuffer可以保证多线程安全。



#### 3、释放同步监视器的锁定

1.任何线程进入同步代码块，同步方法之前，必须先获得对同步监视器的锁定，那么如何释放对同步监视器的锁定呢，线程会在一下几种情况下释放同步监视器：

当前线程的同步方法、同步代码块执行结束，当前线程即释放同步监视器；
当前线程在同步代码块、同步方法中遇到break,return终止了该代码块、方法的继续执行；
当前线程在同步代码块、同步方法中出现了未处理的Error或Exception，导致了该代码块、方法的异常结束；
当前线程执行同步代码块或同步方法时，程序执行了同步监视器对象的wait()方法，则当前线程暂停，并释放同步监视器；

2.以下几种情况，线程不会释放同步监视器：
线程执行同步代码块或同步方法时，程序调用Thread.sleep(),Thread.yield()方法来暂停当前线程的执行，当前线程不会释放同步监视器；
线程执行同步代码块时，其他线程调用了该线程的suspend()方法将该线程挂起，该线程不会释放同步监视器，当然，程序应尽量避免使用suspend()和resume()方法来控制线程。



#### 4、 同步锁：

1.Java5开始，Java提供了一种功能更加强大的线程同步机制——通过显式定义同步锁对象来实现同步，这里的同步锁由Lock对象充当。
Lock 对象提供了比synchronized方法和synchronized代码块更广泛的锁定操作，Lock是控制多个线程对共享资源进行访问的工具。通常， 锁提供了对共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应该先获得Lock对象。
某些锁可能允许对共享资源并发访问，如ReadWriteLock(读写锁)，Lock,ReadWriteLock是Java5提供的两个根接口，并为 Lock提供了ReentrantLock实现类，为ReadWriteLock提供了ReentrantReadWriteLock实现类。在 Java8中提供了新型的StampLock类，在大多数场景下它可以替代传统的ReentrantReadWriteLock。 ReentrantReadWriteLock为读写操作提供了三种锁模式：Writing,ReadingOptimistic,Reading。
2.在实现线程安全的控制中，比较常用的是ReentrantLock(可重入锁)。主要的代码格式如下：

```java
class X{
    //定义锁对象
    private final ReentrantLock lock=new ReentrantLock();
    //定义需要保证线程安全的方法
    public void m(){
        //加锁
        lock.lock();
        try{
            //...method body
        }
        //使用finally块来保证释放锁
        finally{
            lock.unlock();
        }
    }
}




public class Account {
    private final ReentrantLock lock=new ReentrantLock();
    private String accountNo;
    private double balance;
    public Account(String accountNo,double balance){
        this.accountNo=accountNo;
        this.balance=balance;
    }
 
    //因为账户余额不可以随便更改，所以只为balance提供getter方法
    public double getBalance() {
        return balance;
    }
 
    public String getAccountNo() {
        return accountNo;
    }
 
    public void setAccountNo(String accountNo) {
        this.accountNo = accountNo;
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
 
        Account account = (Account) o;
 
        return accountNo.equals(account.accountNo);
 
    }
 
    @Override
    public int hashCode() {
        return accountNo.hashCode();
    }
 
    //提供一个线程安全的draw()方法来完成取钱操作
    public void draw(double drawAmount){
        //加锁
        lock.lock();
        try {
            if (balance >= drawAmount) {
                System.out.println(Thread.currentThread().getName() + "取钱成功！吐出钞票：" + drawAmount);
                try {
                    Thread.sleep(1);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
                balance -= drawAmount;
                System.out.println("\t余额为：" + balance);
            } else {
                System.out.println(Thread.currentThread().getName() + "取钱失败，余额不足");
            }
        }finally {
            lock.unlock();
        }
    }
}
```

使用Lock与使用同步代码有点相似，只是使用Lock时可以显式使用Lock对象作为同步锁，而使用同步方法时系统隐式使用当前对象作为同步监视器。使用 Lock时每个Lock对象对应一个Account对象，一样可以保证对于同一个Account对象，同一个时刻只能有一个线程进入临界区。Lock提供 了同步方法和同步代码块所没有的其他功能，包括使用非块结构的tryLock()方法，以及试图获取可中断锁的lockInterruptibly()方法，还有获取超时失效锁的tryLock(long,TimeUnit)方法。
ReentrantLock可重入锁的意思是，一个线程可以对已被加锁的ReentrantLock锁再次加锁，ReentrantLock对象会维持一个计数器来追踪lock()方法的嵌套调用，线程在每次调用lock()加锁后，必须显式调用unlock()来释放锁，所以一段被锁保护的代码可以调用另一个被相同锁保护的方法。



5、 死锁
当两个线程相互等待对方释放同步监视器时就会发生死锁，Java虚拟机没有检测，也没有采取措施来处理死锁情况，所以多线程编程时应该采取措施避免死锁出现。一旦出现死锁，程序既不会发生任何异常，也不会给出任何提示，只是所有线程都处于阻塞状态，无法继续。

```java
class A{
    public synchronized void foo(B b){
        System.out.println("当前线程名为："+Thread.currentThread().getName()+"进入了A实例的foo()方法");
        try{
            Thread.sleep(200);
        }catch(InterruptedException ex){
            ex.printStackTrace();
        }
        System.out.println("当前线程名为："+Thread.currentThread().getName()+"试图调用B实例的last()方法");
        b.last();
    }
    public synchronized void last(){
        System.out.println("进入了A类的last()方法内部");
    }
}
class B{
    public synchronized void bar(A a){
        System.out.println("当前线程名为："+Thread.currentThread().getName()+"进入了B实例的bar()方法");
        try{
            Thread.sleep(200);
        }catch(InterruptedException ex){
            ex.printStackTrace();
        }
        System.out.println("当前线程名为："+Thread.currentThread().getName()+"试图调用A实例的last()方法");
        a.last();
    }
    public synchronized void last(){
        System.out.println("进入了B类的last()方法内部");
    }
}
public class DeadLock implements Runnable{
    A a =new A();
    B b=new B();
    public void init(){
        Thread.currentThread().setName("主线程");
        a.foo(b);
        System.out.println("进入了主线程之后...");
    }
    public void run(){
        Thread.currentThread().setName("副线程");
        b.bar(a);
        System.out.println("进入了副线程之后...");
    }
    public static void main(String[] args){
        DeadLock d1=new DeadLock();
        new Thread(d1).start();
        d1.init();
    }
}
```

结果：

当前线程名为：主线程进入了A实例的foo()方法

当前线程名为：副线程进入了B实例的bar()方法

当前线程名为：主线程试图调用B实例的last()方法

当前线程名为：副线程试图调用A实例的last()方法



## Java并发编程：线程池的使用

#### 为什么用线程池

1创建/销毁线程伴随着系统开销，过于频繁的创建/销毁线程，会很大程度上影响处理效率

例如：

记创建线程消耗时间T1，执行任务消耗时间T2，销毁线程消耗时间T3

如果T1+T3>T2，那么是不是说开启一个线程来执行这个任务太不划算了！

正好，线程池缓存线程，可用已有的闲置线程来执行新任务，避免了T1+T3带来的系统开销

2线程并发数量过多，抢占系统资源从而导致阻塞

我们知道线程能共享系统资源，如果同时执行的线程过多，就有可能导致系统资源不足而产生阻塞的情况

运用线程池能有效的控制线程最大并发数，避免以上的问题

3对线程进行一些简单的管理

比如：延时执行、定时循环执行的策略等

运用线程池都能进行很好的实现

#### 线程池任务执行流程：

当线程池小于corePoolSize时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程。
当线程池达到corePoolSize时，新提交任务将被放入workQueue中，等待线程池中任务调度执行
当workQueue已满，且maximumPoolSize>corePoolSize时，新提交任务会创建新线程执行任务
当提交任务数超过maximumPoolSize时，新提交任务由RejectedExecutionHandler处理
当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程
当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭

### 一、线程池ThreadPoolExecutor

java.uitl.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，因此如果要透彻地了解Java中的线程池，必须先了解这个类。

ThreadPoolExecutor提供了四个构造函数

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    .....
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
            BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler);
 
    public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,
        BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler);
    ...
}


```



各个参数的含义：

#### 1.corePoolSize：核心池的大小

该线程池中核心线程数最大值。

核心线程：

线程池新建线程的时候，如果当前线程总数小于corePoolSize，则新建的是核心线程，如果超过corePoolSize，则新建的是非核心线程

核心线程默认情况下会一直存活在线程池中，即使这个核心线程啥也不干(闲置状态)。

如果指定ThreadPoolExecutor的allowCoreThreadTimeOut这个属性为true，那么核心线程如果不干活(闲置状态)的话，超过一定时间(时长下面参数决定)，就会被销毁掉。



#### 2.maximumPoolSize

该线程池中线程总数最大值

线程总数 = 核心线程数 + 非核心线程数。它表示在线程池中最多能创建多少个线程。

#### 3.keepAliveTime

该线程池中非核心线程闲置超时时长

一个非核心线程，如果不干活(闲置状态)的时长超过这个参数所设定的时长，就会被销毁掉

如果设置allowCoreThreadTimeOut = true，则会作用于核心线程

#### 4.unit

参数keepAliveTime的时间单位，有7种取值，在TimeUnit类中有7种静态属性：

```doc
TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒
```

#### 5.workQueue

```doc
该线程池中的任务队列：维护着等待执行的Runnable对象。一个阻塞队列，用来存储等待执行的任务。

当所有的核心线程都在干活时，新添加的任务会被添加到这个队列中等待处理，如果队列满了，则新建非核心线程执行任务
常用的workQueue类型：

SynchronousQueue：这个队列接收到任务的时候，会直接提交给线程处理，而不保留它，如果所有线程都在工作怎么办？那就新建一个线程来处理这个任务！所以为了保证不出现<线程数达到了maximumPoolSize而不能新建线程>的错误，使用这个类型队列的时候，maximumPoolSize一般指定成Integer.MAX_VALUE，即无限大

LinkedBlockingQueue：这个队列接收到任务的时候，如果当前线程数小于核心线程数，则新建线程(核心线程)处理任务；如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize

ArrayBlockingQueue：可以限定队列的长度，接收到任务的时候，如果没有达到corePoolSize的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务，又如果总线程数到了maximumPoolSize，并且队列也满了，则发生错误

DelayQueue：队列内元素必须实现Delayed接口，这就意味着你传进去的任务必须先实现Delayed接口。这个队列接收到任务时，首先先入队，只有达到了指定的延时时间，才会执行任务

```

#### 6.threadFactory

线程工厂，主要用来创建线程。

#### 7.handler

抛出异常。表示当拒绝处理任务时的策略，有以下四种取值：

```doc
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务 
```



在ThreadPoolExecutor类中有几个非常重要的方法：


execute()
submit()
shutdown()
shutdownNow()
 　　execute()方法实际上是Executor中声明的方法，在ThreadPoolExecutor进行了具体的实现，这个方法是ThreadPoolExecutor的核心方法，通过这个方法可以向线程池提交一个任务，交由线程池去执行。

　　submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService就已经有了具体的实现，在ThreadPoolExecutor中并没有对其进行重写，这个方法也是用来向线程池提交任务的，但是它和execute()方法不同，它能够返回任务执行的结果，去看submit()方法的实现，会发现它实际上还是调用的execute()方法，只不过它利用了Future来获取任务执行结果

新建一个线程池的时候，一般只用5个参数的构造函数。

还有很多其他的方法：

　　比如：getQueue() 、getPoolSize() 、getActiveCount()、getCompletedTaskCount()等获取与线程池相关属性的方法

### 二、深入剖析线程池实现原理

#### 1.线程池状态

　　在ThreadPoolExecutor中定义了一个volatile变量，另外定义了几个static final变量表示线程池的各个状态：

volatile int runState;
static final int RUNNING    = 0;
static final int SHUTDOWN   = 1;
static final int STOP       = 2;
static final int TERMINATED = 3;
 　　runState表示当前线程池的状态，它是一个volatile变量用来保证线程之间的可见性；

　　下面的几个static final变量表示runState可能的几个取值。

　　当创建线程池后，初始时，线程池处于RUNNING状态；

　　如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；

　　如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；

　　当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。

#### 2.任务的执行

　　在了解将任务提交给线程池到任务执行完毕整个过程之前，我们先来看一下ThreadPoolExecutor类中其他的一些比较重要成员变量：

```java
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状态（比如线程池大小
                                                              //、runState等）的改变都要使用这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集
 
private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
 
private volatile int   poolSize;       //线程池中当前的线程数
 
private volatile RejectedExecutionHandler handler; //任务拒绝策略
 
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
 
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
 
private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```

　　下面我们进入正题，看一下任务从提交到最终执行完毕经历了哪些过程。

　　在ThreadPoolExecutor类中，最核心的任务提交方法是execute()方法，虽然通过submit也可以提交任务，但是实际上submit方法里面最终调用的还是execute()方法，所以我们只需要研究execute()方法的实现原理即可：

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
        if (runState == RUNNING && workQueue.offer(command)) {
            if (runState != RUNNING || poolSize == 0)
                ensureQueuedTaskHandled(command);
        }
        else if (!addIfUnderMaximumPoolSize(command))
            reject(command); // is shutdown or saturated
    }
}
```

#### 3.线程池中的线程初始化

　　默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线程。

　　在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到：

prestartCoreThread()：初始化一个核心线程；
prestartAllCoreThreads()：初始化所有核心线程

```java
public boolean prestartCoreThread() {
    return addIfUnderCorePoolSize(null); //注意传进去的参数是null
}
 
public int prestartAllCoreThreads() {
    int n = 0;
    while (addIfUnderCorePoolSize(null))//注意传进去的参数是null
        ++n;
    return n;
}
```

#### 4.任务缓存队列及排队策略

　　在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。

　　workQueue的类型为BlockingQueue<Runnable>，通常可以取下面三种类型：

　　1）ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；

　　2）LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；

　　3）synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

#### 5.任务拒绝策略

　　当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略，通常有以下四种策略：

ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

### 6.线程池的关闭

　　ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：

shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

#### 7.线程池容量的动态调整

　　ThreadPoolExecutor提供了动态调整线程池容量大小的方法：setCorePoolSize()和setMaximumPoolSize()，

setCorePoolSize：设置核心池大小
setMaximumPoolSize：设置线程池最大能创建的线程数目大小
　　当上述参数从小变大时，ThreadPoolExecutor进行线程赋值，还可能立即创建新的线程来执行任务。

### 三、使用示例

```java
public class Test {
     public static void main(String[] args) {   
         ThreadPoolExecutor executor = new ThreadPoolExecutor(5, 10, 200, TimeUnit.MILLISECONDS,
                 new ArrayBlockingQueue<Runnable>(5));
          
         for(int i=0;i<15;i++){
             MyTask myTask = new MyTask(i);
             executor.execute(myTask);
             System.out.println("线程池中线程数目："+executor.getPoolSize()+"，队列中等待执行的任务数目："+
             executor.getQueue().size()+"，已执行玩别的任务数目："+executor.getCompletedTaskCount());
         }
         executor.shutdown();
     }
}
 
 
class MyTask implements Runnable {
    private int taskNum;
     
    public MyTask(int num) {
        this.taskNum = num;
    }
     
    @Override
    public void run() {
        System.out.println("正在执行task "+taskNum);
        try {
            Thread.currentThread().sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("task "+taskNum+"执行完毕");
    }
}
```



从执行结果可以看出，当线程池中线程的数目大于5时，便将任务放入任务缓存队列里面，当任务缓存队列满了之后，便创建新的线程。如果上面程序中，将for循环中改成执行20个任务，就会抛出任务拒绝异常了。

#### 使用Executors类中提供的几个静态方法来创建线程池：

不过在java doc中，并不提倡我们直接使用ThreadPoolExecutor，而是使用Executors类中提供的几个静态方法来创建线程池：

```java
Executors.newCachedThreadPool();        //创建一个缓冲池，缓冲池容量大小为Integer.MAX_VALUE
Executors.newSingleThreadExecutor();   //创建容量为1的缓冲池
Executors.newFixedThreadPool(int);    //创建固定容量大小的缓冲池
```

```java
下面是这三个静态方法的具体实现;
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

```doc
　　从它们的具体实现来看，它们实际上也是调用了ThreadPoolExecutor，只不过参数都已配置好了。

　　newFixedThreadPool创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue；

　　newSingleThreadExecutor将corePoolSize和maximumPoolSize都设置为1，也使用的LinkedBlockingQueue；

　　newCachedThreadPool将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。

　　实际中，如果Executors提供的三个静态方法能满足要求，就尽量使用它提供的三个方法，因为自己去手动配置ThreadPoolExecutor的参数有点麻烦，要根据实际任务的类型和数量来进行配置。

　　另外，如果ThreadPoolExecutor达不到要求，可以自己继承ThreadPoolExecutor类进行重写。
```



### 四、如何合理配置线程池的大小　

一般需要根据任务的类型来配置线程池大小：

　　如果是CPU密集型任务，就需要尽量压榨CPU，参考值可以设为 NCPU+1

　　如果是IO密集型任务，参考值可以设置为2*NCPU

　　当然，这只是一个参考值，具体的设置还需要根据实际情况进行调整，比如可以先将线程池大小设置为参考值，再观察任务运行情况和系统负载、资源利用率来进行适当调整。



2018/04/04   星期三     阴天

## Java泛型

### 1.概述

泛型在java中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用。

什么是泛型？为什么要使用泛型？

```doc
泛型，即“参数化类型”。一提到参数，最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。那么参数化类型怎么理解呢？顾名思义，就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。

泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。
```



### 2.一个被举了无数次的例子

```java
List arrayList = new ArrayList();
arrayList.add("aaaa");
arrayList.add(100);

for(int i = 0; i< arrayList.size();i++){
    String item = (String)arrayList.get(i);
    Log.d("泛型测试","item = " + item);
}

/*毫无疑问，程序的运行结果会以崩溃结束：
java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
ArrayList可以存放任意类型，例子中添加了一个String类型，添加了一个Integer类型，再使用时都以String的方式使用，因此程序崩溃了。为了解决类似这样的问题（在编译阶段就可以解决），泛型应运而生。

*/

List<String> arrayList = new ArrayList<String>();
...
//arrayList.add(100); 在编译阶段，编译器就会报错
```



### 3.特性

泛型只在编译阶段有效。

```java
List<String> stringArrayList = new ArrayList<String>();
List<Integer> integerArrayList = new ArrayList<Integer>();

Class classStringArrayList = stringArrayList.getClass();
Class classIntegerArrayList = integerArrayList.getClass();

if(classStringArrayList.equals(classIntegerArrayList)){
    Log.d("泛型测试","类型相同");
}
// 输出结果：D/泛型测试: 类型相同。
```

**对此总结成一句话：泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。**

### 4.泛型的使用

**泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法**

#### 1)泛型类

```doc
泛型类型用于类的定义中，被称为泛型类。
通过泛型可以完成对一组类的操作对外开放相同的接口。
最典型的就是各种容器类，如：List、Set、Map。
```

一个最普通的泛型类：

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
}




//泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
Generic<Integer> genericInteger = new Generic<Integer>(123456);

//传入的实参类型需与泛型的类型参数类型相同，即为String.
Generic<String> genericString = new Generic<String>("key_vlaue");
Log.d("泛型测试","key is " + genericInteger.getKey());
Log.d("泛型测试","key is " + genericString.getKey());
```

```java
package com.ahhf.ljxbw.dao;


import org.apache.ibatis.annotations.Mapper;

import com.ahhf.ljxbw.entity.UserlogininfoEntity;

/**
 * 
 * 
 * @author wenjin.zhu
 * @email 15156980156@163.com
 * @date 2018-04-04 13:12:55
 */
@Mapper
public interface UserlogininfoDao extends BaseDao<UserlogininfoEntity> {

	/*UserlogininfoEntity queryObject(Integer id);

	List<UserlogininfoEntity> queryList(Map<String, Object> map);

	int queryTotal(Map<String, Object> map);

	void save(UserlogininfoEntity userlogininfo);

	void update(UserlogininfoEntity userlogininfo);

	void delete(Integer id);

	void deleteBatch(Integer[] ids);*/
	
}


package com.ahhf.ljxbw.dao;

import java.util.List;
import java.util.Map;

public interface BaseDao<T> {
	T queryObject(Integer id);

	List<T> queryList(Map<String, Object> map);

	int queryTotal(Map<String, Object> map);

	void save(T t);

	void update(T t);

	void delete(Integer id);

	void deleteBatch(Integer[] ids);
}
```

定义的泛型类，就一定要传入泛型类型实参么？并不是这样，在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。

**注意**

1.泛型的类型参数只能是类类型，不能是简单类型。

2.不能对确切的泛型类型使用instanceof操作。如下面的操作是非法的，编译时会出错。

if(ex_num instanceof Generic<Number>){   
} 

```java
在比较一个类是否和另一个类属于同一个类实例的时候，我们通常可以采用instanceof和getClass两种方法通过两者是否相等来判断，但是两者在判断上面是有差别的，下面从代码中看看区别：
public class Test
{
	public static void testInstanceof(Object x)
	{
		System.out.println("x instanceof Parent:  "+(x instanceof Parent));
		System.out.println("x instanceof Child:  "+(x instanceof Child));
		System.out.println("x getClass Parent:  "+(x.getClass() == Parent.class));
		System.out.println("x getClass Child:  "+(x.getClass() == Child.class));
	}
	public static void main(String[] args) {
		testInstanceof(new Parent());
		System.out.println("---------------------------");
		testInstanceof(new Child());
	}
}
class Parent {

}
class Child extends Parent {

}
/*
输出:
x instanceof Parent:  true
x instanceof Child:  false
x getClass Parent:  true
x getClass Child:  false
---------------------------
x instanceof Parent:  true
x instanceof Child:  true
x getClass Parent:  false
x getClass Child:  true
*/

instanceof进行类型检查规则是:你属于该类吗？或者你属于该类的派生类吗？而通过getClass获得类型信息采用==来进行检查是否相等的操作是严格的判断。不会存在继承方面的考虑；
  
```

#### 2)泛型接口



泛型接口与泛型类的定义及使用基本相同。泛型接口常被用在各种类的生产器中

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}


/**
 * 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中
 * 即：class FruitGenerator<T> implements Generator<T>{
 * 如果不声明泛型，如：class FruitGenerator implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}



/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```





#### 3)泛型通配符

```doc
我们知道Ingeter是Number的一个子类，同时在特性章节中我们也验证过Generic<Ingeter>与Generic<Number>实际上是相同的一种基本类型。那么问题来了，在使用Generic<Number>作为形参的方法中，能否使用Generic<Ingeter>的实例传入呢？在逻辑上类似于Generic<Number>和Generic<Ingeter>是否可以看成具有父子关系的泛型类型呢？

为了弄清楚这个问题，我们使用Generic<T>这个泛型类继续看下面的例子：

public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

showKeyValue(gNumber);

// showKeyValue这个方法编译器会为我们报错：Generic<java.lang.Integer> 
// cannot be applied to Generic<java.lang.Number>
// showKeyValue(gInteger);

通过提示信息我们可以看到Generic<Integer>不能被看作为`Generic<Number>的子类。由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

回到上面的例子，如何解决上面的问题？总不能为了定义一个新的方法来处理Generic<Integer>类型的类，这显然与java中的多台理念相违背。因此我们需要一个在逻辑上可以表示同时是Generic<Integer>和Generic<Number>父类的引用类型。由此类型通配符应运而生。

我们可以将上面的方法改一下：

public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

类型通配符一般是使用？代替具体的类型实参，注意了，此处’？’是类型实参，而不是类型形参 。重要说三遍！此处’？’是类型实参，而不是类型形参 ！ 此处’？’是类型实参，而不是类型形参 ！再直白点的意思就是，此处的？和Number、String、Integer一样都是一种实际的类型，可以把？看成所有类型的父类。是一种真实的类型。

可以解决当具体类型不确定的时候，这个通配符就是 ?  ；当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用 ? 通配符来表未知类型。
```



## Java Socket 通信

### 一、Socket通信基本示例

```java
package yiwangzhibujian.onlysend;

import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
  public static void main(String[] args) throws Exception {
    // 监听指定的端口
    int port = 55533;
    ServerSocket server = new ServerSocket(port);
    
    // server将一直等待连接的到来
    System.out.println("server将一直等待连接的到来");
    Socket socket = server.accept();
    // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
    InputStream inputStream = socket.getInputStream();
    byte[] bytes = new byte[1024];
    int len;
    StringBuilder sb = new StringBuilder();
    while ((len = inputStream.read(bytes)) != -1) {
      //注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
      sb.append(new String(bytes, 0, len,"UTF-8"));
    }
    System.out.println("get message from client: " + sb);
    inputStream.close();
    socket.close();
    server.close();
  }
}
```

服务端监听一个端口，等待连接的到来。

```java
package yiwangzhibujian.onlysend;

import java.io.OutputStream;
import java.net.Socket;

public class SocketClient {
  public static void main(String args[]) throws Exception {
    // 要连接的服务端IP地址和端口
    String host = "127.0.0.1"; 
    int port = 55533;
    // 与服务端建立连接
    Socket socket = new Socket(host, port);
    // 建立连接后获得输出流
    OutputStream outputStream = socket.getOutputStream();
    String message="你好  yiwangzhibujian";
    socket.getOutputStream().write(message.getBytes("UTF-8"));
    outputStream.close();
    socket.close();
  }
}
```

客户端通过ip和端口，连接到指定的server，然后通过Socket获得输出流，并向其输出内容，服务器会获得消息。

```doc
通过这个例子应该掌握并了解：

Socket服务端和客户端的基本编程
传输编码统一指定，防止乱码
```

### 二、消息通信优化

#### 2.1 双向通信，发送消息并接受消息

```java
package yiwangzhibujian.waitreceive;

import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
  public static void main(String[] args) throws Exception {
    // 监听指定的端口
    int port = 55533;
    ServerSocket server = new ServerSocket(port);
    
    // server将一直等待连接的到来
    System.out.println("server将一直等待连接的到来");
    Socket socket = server.accept();
    // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
    InputStream inputStream = socket.getInputStream();
    byte[] bytes = new byte[1024];
    int len;
    StringBuilder sb = new StringBuilder();
    //只有当客户端关闭它的输出流的时候，服务端才能取得结尾的-1
    while ((len = inputStream.read(bytes)) != -1) {
      // 注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
      sb.append(new String(bytes, 0, len, "UTF-8"));
    }
    System.out.println("get message from client: " + sb);

    OutputStream outputStream = socket.getOutputStream();
    outputStream.write("Hello Client,I get the message.".getBytes("UTF-8"));

    inputStream.close();
    outputStream.close();
    socket.close();
    server.close();
  }
}
```

与之前server的不同在于，当读取完客户端的消息后，打开输出流，将指定消息发送回客户端，客户端程序为：

```java
package yiwangzhibujian.waitreceive;

import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

public class SocketClient {
  public static void main(String args[]) throws Exception {
    // 要连接的服务端IP地址和端口
    String host = "127.0.0.1";
    int port = 55533;
    // 与服务端建立连接
    Socket socket = new Socket(host, port);
    // 建立连接后获得输出流
    OutputStream outputStream = socket.getOutputStream();
    String message = "你好  yiwangzhibujian";
    socket.getOutputStream().write(message.getBytes("UTF-8"));
    //通过shutdownOutput高速服务器已经发送完数据，后续只能接受数据
    socket.shutdownOutput();
    
    InputStream inputStream = socket.getInputStream();
    byte[] bytes = new byte[1024];
    int len;
    StringBuilder sb = new StringBuilder();
    while ((len = inputStream.read(bytes)) != -1) {
      //注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
      sb.append(new String(bytes, 0, len,"UTF-8"));
    }
    System.out.println("get message from server: " + sb);
    
    inputStream.close();
    outputStream.close();
    socket.close();
  }
}
```

　　客户端也有相应的变化，在发送完消息时，调用关闭输出流方法，然后打开输出流，等候服务端的消息。

#### 2.2 使用场景

　　这个模式的使用场景一般用在，客户端发送命令给服务器，然后服务器相应指定的命令，如果只是客户端发送消息给服务器，然后让服务器返回收到消息的消息，这就有点过分了，这就是完全不相信Socket的传输安全性，要知道它的底层可是TCP，如果没有发送到服务器端是会抛异常的，这点完全不用担心。

#### 2.3 如何告知对方已发送完命令

　　其实这个问题还是比较重要的，正常来说，客户端打开一个输出流，如果不做约定，也不关闭它，那么服务端永远不知道客户端是否发送完消息，那么服务端会一直等待下去，直到读取超时。所以怎么告知服务端已经发送完消息就显得特别重要。

#### 2.3.1 通过Socket关闭

　　这个是第一章介绍的方式，当Socket关闭的时候，服务端就会收到响应的关闭信号，那么服务端也就知道流已经关闭了，这个时候读取操作完成，就可以继续后续工作。

　　但是这种方式有一些缺点

客户端Socket关闭后，将不能接受服务端发送的消息，也不能再次发送消息
如果客户端想再次发送消息，需要重现创建Socket连接

#### 2.3.2 通过Socket关闭输出流的方式

　　这种方式调用的方法是：

socket.shutdownOutput();
　　而不是（outputStream为发送消息到服务端打开的输出流）：

outputStream.close();
　　如果关闭了输出流，那么相应的Socket也将关闭，和直接关闭Socket一个性质。

　　调用Socket的shutdownOutput()方法，底层会告知服务端我这边已经写完了，那么服务端收到消息后，就能知道已经读取完消息，如果服务端有要返回给客户的消息那么就可以通过服务端的输出流发送给客户端，如果没有，直接关闭Socket。

　　这种方式通过关闭客户端的输出流，告知服务端已经写完了，虽然可以读到服务端发送的消息，但是还是有一点点缺点：

不能再次发送消息给服务端，如果再次发送，需要重新建立Socket连接
　　这个缺点，在访问频率比较高的情况下将是一个需要优化的地方。

#### 2.3.3 通过约定符号

　　这种方式的用法，就是双方约定一个字符或者一个短语，来当做消息发送完成的标识，通常这么做就需要改造读取方法。

　　假如约定单端的一行为end，代表发送完成，例如下面的消息，end则代表消息发送完成：

hello yiwangzhibujian
end

服务端响应的读取操作需要进行如下改造：

```JAVA
Socket socket = server.accept();
// 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
BufferedReader read=new BufferedReader(new InputStreamReader(socket.getInputStream(),"UTF-8"));
String line;
StringBuilder sb = new StringBuilder();
while ((line = read.readLine()) != null && "end".equals(line)) {
  //注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
  sb.append(line);
}
```

可以看见，服务端不仅判断是否读到了流的末尾，还判断了是否读到了约定的末尾。

　　这么做的优缺点如下：

优点：不需要关闭流，当发送完一条命令（消息）后可以再次发送新的命令（消息）
缺点：需要额外的约定结束标志，太简单的容易出现在要发送的消息中，误被结束，太复杂的不好处理，还占带宽
　　经过了这么多的优化还是有缺点，难道就没有完美的解决方案吗，答案是有的，看接下来的内容。

#### 2.3.4 通过指定长度

　　如果你了解一点class文件的结构（后续会写，敬请期待），那么你就会佩服这么设计方式，也就是说我们可以在此找灵感，就是我们可以先指定后续命令的长度，然后读取指定长度的内容做为客户端发送的消息。

　　现在首要的问题就是用几个字节指定长度呢，我们可以算一算：

1个字节：最大256，表示256B
2个字节：最大65536，表示64K
3个字节：最大16777216，表示16M
4个字节：最大4294967296，表示4G
依次类推
　　这个时候是不是很纠结，最大的当然是最保险的，但是真的有必要选择最大的吗，其实如果你稍微了解一点UTF-8的编码方式（字符编码后续会写，敬请期待），那么你就应该能想到为什么一定要固定表示长度字节的长度呢，我们可以使用变长方式来表示长度的表示，比如：

第一个字节首位为0：即0XXXXXXX，表示长度就一个字节，最大128，表示128B
第一个字节首位为110，那么附带后面一个字节表示长度：即110XXXXX 10XXXXXX，最大2048，表示2K
第一个字节首位为1110，那么附带后面二个字节表示长度：即110XXXXX 10XXXXXX 10XXXXXX，最大131072，表示128K
依次类推
　　上面提到的这种用法适合高富帅的程序员使用，一般呢，如果用作命名发送，两个字节就够了，如果还不放心4个字节基本就能满足你的所有要求，下面的例子我们将采用2个字节表示长度，目的只是给你一种思路，让你知道有这种方式来获取消息的结尾：

　　服务端程序：

```java
package yiwangzhibujian.waitreceive2;

import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
  public static void main(String[] args) throws Exception {
    // 监听指定的端口
    int port = 55533;
    ServerSocket server = new ServerSocket(port);

    // server将一直等待连接的到来
    System.out.println("server将一直等待连接的到来");
    Socket socket = server.accept();
    // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
    InputStream inputStream = socket.getInputStream();
    byte[] bytes;
    // 因为可以复用Socket且能判断长度，所以可以一个Socket用到底
    while (true) {
      // 首先读取两个字节表示的长度
      int first = inputStream.read();
      //如果读取的值为-1 说明到了流的末尾，Socket已经被关闭了，此时将不能再去读取
      if(first==-1){
        break;
      }
      int second = inputStream.read();
      int length = (first << 8) + second;
      // 然后构造一个指定长的byte数组
      bytes = new byte[length];
      // 然后读取指定长度的消息即可
      inputStream.read(bytes);
      System.out.println("get message from client: " + new String(bytes, "UTF-8"));
    }
    inputStream.close();
    socket.close();
    server.close();
  }
}
```

此处的读取步骤为，先读取两个字节的长度，然后读取消息，客户端为：

```java
package yiwangzhibujian.waitreceive2;

import java.io.OutputStream;
import java.net.Socket;

public class SocketClient {
  public static void main(String args[]) throws Exception {
    // 要连接的服务端IP地址和端口
    String host = "127.0.0.1";
    int port = 55533;
    // 与服务端建立连接
    Socket socket = new Socket(host, port);
    // 建立连接后获得输出流
    OutputStream outputStream = socket.getOutputStream();
    String message = "你好  yiwangzhibujian";
    //首先需要计算得知消息的长度
    byte[] sendBytes = message.getBytes("UTF-8");
    //然后将消息的长度优先发送出去
    outputStream.write(sendBytes.length >>8);
    outputStream.write(sendBytes.length);
    //然后将消息再次发送出去
    outputStream.write(sendBytes);
    outputStream.flush();
    //==========此处重复发送一次，实际项目中为多个命名，此处只为展示用法
    message = "第二条消息";
    sendBytes = message.getBytes("UTF-8");
    outputStream.write(sendBytes.length >>8);
    outputStream.write(sendBytes.length);
    outputStream.write(sendBytes);
    outputStream.flush();
    //==========此处重复发送一次，实际项目中为多个命名，此处只为展示用法
    message = "the third message!";
    sendBytes = message.getBytes("UTF-8");
    outputStream.write(sendBytes.length >>8);
    outputStream.write(sendBytes.length);
    outputStream.write(sendBytes);    
    
    outputStream.close();
    socket.close();
  }
}
```

客户端要多做的是，在发送消息之前先把消息的长度发送过去。

　　这种事先约定好长度的做法解决了之前提到的种种问题，Redis的Java客户端Jedis就是用这种方式实现的这种方式的缺点：

暂时还没发现
　　当然如果是需要服务器返回结果，那么也依然使用这种方式，服务端也是先发送结果的长度，然后客户端进行读取。当然现在流行的就是，长度+类型+数据模式的传输方式。

### 三、服务端优化

#### 3.1 服务端并发处理能力

　　在上面的例子中，服务端仅仅只是接受了一个Socket请求，并处理了它，然后就结束了，但是在实际开发中，一个Socket服务往往需要服务大量的Socket请求，那么就不能再服务完一个Socket的时候就关闭了，这时候可以采用循环接受请求并处理的逻辑：

```java
package yiwangzhibujian.multiserver;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
  public static void main(String args[]) throws IOException {
    // 监听指定的端口
    int port = 55533;
    ServerSocket server = new ServerSocket(port);
    // server将一直等待连接的到来
    System.out.println("server将一直等待连接的到来");
    
    while(true){
      Socket socket = server.accept();
      // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
      InputStream inputStream = socket.getInputStream();
      byte[] bytes = new byte[1024];
      int len;
      StringBuilder sb = new StringBuilder();
      while ((len = inputStream.read(bytes)) != -1) {
        // 注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
        sb.append(new String(bytes, 0, len, "UTF-8"));
      }
      System.out.println("get message from client: " + sb);
      inputStream.close();
      socket.close();
    }
    
  }
}
```

　　这种一般也是新手写法，但是能够循环处理多个Socket请求，不过当一个请求的处理比较耗时的时候，后面的请求将被阻塞，所以一般都是用多线程的方式来处理Socket，即每有一个Socket请求的时候，就创建一个线程来处理它。

　　不过在实际生产中，创建的线程会交给线程池来处理，为了：

线程复用，创建线程耗时，回收线程慢
防止短时间内高并发，指定线程池大小，超过数量将等待，方式短时间创建大量线程导致资源耗尽，服务挂掉

```java
package yiwangzhibujian.threadserver;

import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SocketServer {
  public static void main(String args[]) throws Exception {
    // 监听指定的端口
    int port = 55533;
    ServerSocket server = new ServerSocket(port);
    // server将一直等待连接的到来
    System.out.println("server将一直等待连接的到来");

    //如果使用多线程，那就需要线程池，防止并发过高时创建过多线程耗尽资源
    ExecutorService threadPool = Executors.newFixedThreadPool(100);
    
    while (true) {
      Socket socket = server.accept();
      
      Runnable runnable=()->{
        try {
          // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
          InputStream inputStream = socket.getInputStream();
          byte[] bytes = new byte[1024];
          int len;
          StringBuilder sb = new StringBuilder();
          while ((len = inputStream.read(bytes)) != -1) {
            // 注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
            sb.append(new String(bytes, 0, len, "UTF-8"));
          }
          System.out.println("get message from client: " + sb);
          inputStream.close();
          socket.close();
        } catch (Exception e) {
          e.printStackTrace();
        }
      };
      threadPool.submit(runnable);
    }

  }
}
```

使用线程池的方式，算是一种成熟的方式。可以应用在生产中。

#### 3.2 服务端其他属性

　　ServerSocket有以下3个属性。

SO_TIMEOUT：表示等待客户连接的超时时间。一般不设置，会持续等待。
SO_REUSEADDR：表示是否允许重用服务器所绑定的地址。一般不设置，经我的测试没必要，下面会进行详解。
SO_RCVBUF：表示接收数据的缓冲区的大小。一般不设置，用系统默认就可以了。



## JAVA 网络编程

### 一、网络编程概述

```doc
1.网络编程 
　　1.1计算机网络概述 
　　网络编程的实质就是两个(或多个)设备(例如计算机)之间的数据传输。 
　　按照计算机网络的定义，通过一定的物理设备将处于不同位置的计算机连接起来组成的网络，这个网络中包含的设备有：计算机、路由器、交换机等等。 
　　其实从软件编程的角度来说，对于物理设备的理解不需要很深刻，就像你打电话时不需要很熟悉通信网络的底层实现是一样的，但是当深入到网络编程的底层时，这些基础知识是必须要补的。 
　　路由器和交换机组成了核心的计算机网络，计算机只是这个网络上的节点以及控制等，通过光纤、网线等连接将设备连接起来，从而形成了一张巨大的计算机网络。 
　　网络最主要的优势在于共享：共享设备和数据，现在共享设备最常见的是打印机，一个公司一般一个打印机即可，共享数据就是将大量的数据存储在一组机器中，其它的计算机通过网络访问这些数据，例如网站、银行服务器等等。 
　　如果需要了解更多的网络硬件基础知识，可以阅读《计算机网络》教材，对于基础进行强化，这个在基础学习阶段不是必须的，但是如果想在网络编程领域有所造诣，则是一个必须的基本功。 
　　对于网络编程来说，最主要的是计算机和计算机之间的通信，这样首要的问题就是如何找到网络上的计算机呢？这就需要了解IP地址的概念。 
　　为了能够方便的识别网络上的每个设备，网络中的每个设备都会有一个唯一的数字标识，这个就是IP地址。在计算机网络中,现在命名IP地址的规定是IPv4协议，该协议规定每个IP地址由4个0-255之间的数字组成，例如10.0.120.34。每个接入网络的计算机都拥有唯一的IP地址，这个IP地址可能是固定的，例如网络上各种各样的服务器，也可以是动态的，例如使用ADSL拨号上网的宽带用户，无论以何种方式获得或是否是固定的，每个计算机在联网以后都拥有一个唯一的合法IP地址，就像每个手机号码一样。 
　　但是由于IP地址不容易记忆，所以为了方便记忆，有创造了另外一个概念——域名(Domain Name)，例如sohu.com等。一个IP地址可以对应多个域名，一个域名只能对应一个IP地址。域名的概念可以类比手机中的通讯簿，由于手机号码不方便记忆，所以添加一个姓名标识号码，在实际拨打电话时可以选择该姓名，然后拨打即可。 
　　在网络中传输的数据，全部是以IP地址作为地址标识，所以在实际传输数据以前需要将域名转换为IP地址，实现这种功能的服务器称之为DNS服务器，也就是通俗的说法叫做域名解析。例如当用户在浏览器输入域名时，浏览器首先请求DNS服务器，将域名转换为IP地址，然后将转换后的IP地址反馈给浏览器，然后再进行实际的数据传输。 
　　当DNS服务器正常工作时，使用IP地址或域名都可以很方便的找到计算机网络中的某个设备，例如服务器计算机。当DNS不正常工作时，只能通过IP地址访问该设备。所以IP地址的使用要比域名通用一些。 
　　IP地址和域名很好的解决了在网络中找到一个计算机的问题，但是为了让一个计算机可以同时运行多个网络程序，就引入了另外一个概念——端口(port)。 
　　在介绍端口的概念以前，首先来看一个例子，一般一个公司前台会有一个电话，每个员工会有一个分机，这样如果需要找到这个员工的话，需要首先拨打前台总机，然后转该分机号即可。这样减少了公司的开销，也方便了每个员工。在该示例中前台总机的电话号码就相当于IP地址，而每个员工的分机号就相当于端口。 
　　有了端口的概念以后，在同一个计算机中每个程序对应唯一的端口，这样一个计算机上就可以通过端口区分发送给每个端口的数据了，换句话说，也就是一个计算机上可以并发运行多个网络程序，而不会在互相之间产生干扰。 
　　在硬件上规定，端口的号码必须位于0-65535之间，每个端口唯一的对应一个网络程序，一个网络程序可以使用多个端口。这样一个网络程序运行在一台计算上时，不管是客户端还是服务器，都是至少占用一个端口进行网络通讯。在接收数据时，首先发送给对应的计算机，然后计算机根据端口把数据转发给对应的程序。 
　　有了IP地址和端口的概念以后，在进行网络通讯交换时，就可以通过IP地址查找到该台计算机，然后通过端口标识这台计算机上的一个唯一的程序。这样就可以进行网络数据的交换了。 
　　但是，进行网络编程时，只有IP地址和端口的概念还是不够的，下面就介绍一下基础的网络编程相关的软件基础知识。

　　１.2网络编程概述 
　　网络编程中有两个主要的问题，一个是如何准确的定位网络上一台或多台主机，另一个就是找到主机后如何可靠高效的进行数据传输。在TCP/IP协议中IP层主要负责网络主机的定位，数据传输的路由，由IP地址可以唯一地确定Internet上的一台主机。而TCP层则提供面向应用的可靠的或非可靠的数据传输机制，这是网络编程的主要对象，一般不需要关心IP层是如何处理数据的。 
　　按照前面的介绍，网络编程就是两个或多个设备之间的数据交换，其实更具体的说，网络编程就是两个或多个程序之间的数据交换，和普通的单机程序相比，网络程序最大的不同就是需要交换数据的程序运行在不同的计算机上，这样就造成了数据好换的复杂。虽然通过IP地址和端口号可以找到网络上运行的一个程序，但是如果需要进行网络编程，则需要了解网络通讯的过程。 
　　网络通讯基于“请求—响应”模型。在网络通讯中，第一次主动发起通讯的程序被称为客户端(client)程序，简称客户端，而第一次通讯中等待链接的程序被称为服务器端(Server)程序，简称服务器。一旦通讯建立，则客户端和服务器端完全一样，没有本质区别。 
　　由此，网络编程中的两种程序就分别是客户端和服务器端，例如ＱＱ程序，每个ＱＱ用户安装的都是ＱＱ客户端程序，而ＱＱ服务器端程序则在腾讯公司的机房中，为大量的ＱＱ用户提供服务。这种网络编程的结构被称为客户端／服务器结构，也叫Client/Serverj结构，简称C/S结构。 
　　使用C/S结构的程序，在开发时需要分别开发客户端和服务器端，这种结构的优势在于客户端是专门开发的，所以根据需要实现各种效果，专业点的说就是表现力丰富，而服务器端也需要专门进行开发。但是这种结构也存在着很多不足，例如通用性差，几乎不能通用，也就是说一种程序的客户端只能和对应的服务器端通讯，而不能和其他服务器端通讯，在实际维护中，也需要维护专门的客户端和服务器端，维护的压力比较大。 
　　其实在运行很多程序时，没有必要使用专门的客户端，而需要使用通用的客户端，例如浏览器，使用浏览器作为客户端的结构称为浏览器/服务器结构，也叫做Browser/Server结构，简称B/S结构。 
　　使用B/S结构的程序，在开发时只需要开发服务器端即可，这种优势在于开发压力比较小，不需要维护客户端，但是这种结构也存在这很多不足，例如浏览器的限制比较大，表现了不强，不能进行系统级别的操作等。 
　　总之C/S结构和B/S结构是现在网络编程中常见的两种结构，B/S结构其实也就是一种特殊的C/S结构。 
　　另外简单的介绍一下P2P(Point to Point)程序，常见的如BT、电驴等。P2P程序是一种特殊的程序，应该一个P2P程序中既包含客户端程序，也包含服务器端程序，例如BT，使用客户端程序部分连接其它的种子(服务器端)，而使用服务器端向其它的BT客户端传输数据。如果这个还不是很清楚，其实P2P程序和手机是一样的，当手机拨打电话时就是使用客户端的作用，而手机处于待机状态时，可以接收到其它用户拨打的电话则起的就是服务器端的功能，只是一般的手机不能同时使用拨打电话和接听电话的功能，而P2P程序实现了该功能。 
　　最后介绍一下网络编程中最重要的，也是最复杂的概念——协议(protocol)。按照前面的介绍，网络编程就是运行在不同计算机中两个程序之间的数据交换。在实际进行数据交换时，为了让接收端理解该数据，计算机比较笨，什么都不懂的，那么久需要规定该数据的格式，这个数据的格式就是协议。 
　　如果没有理解协议的概念，那么再举一个例子，记得有个电影叫《永不消逝的电波》，讲述的是地下党通过电台发送情报的故事，这里我们不探讨电影的剧情，而只关 心电台发送的数据。在实际发报时，需要首先将需要发送的内容转换为电报编码，然后将电报编码发送出去，而接收端接收的是电报编码，如果需要理解电报的内容 则需要根据密码本翻译出该电报的内容。这里的密码本就规定了一种数据格式，这种对于网络中传输的数据格式在网络编程中就被称作协议。 
　　那么如何编写协议格式呢？答案是随意。只要按照这种协议格式能够生成唯一的编码，按照该编码可以唯一的解析出发送数据的内容即可。也正因为各个网络程序之间协议格式的不同，所以才导致了客户端程序都是专用的结构。 
　　在实际的网络编程中，最麻烦的内容不是数据的发送和接受，因为这个功能在几乎所有编程语言中都提供了封装好的API进行调用，最麻烦的内容就是协议的设计及协议的生产和解析，这个才是网络编程最核心的内容。 
　　１.3网络通讯方式 
　　在现有的网络中，网络通讯的方式主要有两种： 
　　1.TCP(传输控制协议)方式。 
　　2.UDP(用户数据协议)方式。 
　　为了方便理解这两种方式，还是先来看个例子。大家使用手机时，向别人传递信息时有两种方式：拨打电话和发送短信。使用拨打电话的方式可以保证该信息传递给别人，因为别人接电话时本身就确认收到了该信息。而发送短信的方式价格低廉，使用方便，但是接受人可能收不到。 
　　在网络通讯中，TCP方式就类似于拨打电话，使用该种方式进行网络通讯时，需要建立专门的虚拟连接，然后进行可靠的数据传输，如果数据发送失败，则客户端会自动重发该数据，而UDP方式就类似于发送短信，使用这种方式进行网络通讯时，不需要建立专门的虚拟连接，传输也不是很可靠，如果发送失败则客户端无法获得。 
　　这两种传输方式都是实际的网络编程中进行使用，重要的数据一般使用TCP方式进行数据传输，而大量的非核心数据则都通过UDP方式进行传递，在一些程序中甚至结合使用这两种方式进行数据的传递。 
　　由于TCP需要建立专用的虚拟连接以及确认传输是否正确，所以使用TCP方式的速度稍微慢一些，而且传输时产生的数据量要比UDP稍微大一些。 
　　关于网络编程的基础知识就介绍这么多，如果需要深入了解相关知识请阅读专门的计算机网络书籍，下面开始介绍Java语言中网络编程的相关技术。
　　
　　１.3网络编程步骤 
　　按照前面的基础知识介绍，无论使用TCP方式还是UDP方式进行网络通讯，网络编程都是由客户端和服务器端组成，所以，下面介绍网络编程的步骤时，均以C/S结构为基础进行介绍。 
　　1.3.1客户端网络编程步骤 
　　客户端是指网络编程中首先发起连接的程序，客户端一般实现程序界面和基本逻辑实现，在进行实际的客户端编程时，无论客户端复杂还是简单，以及客户端实现的方式，客户端的编程主要由三个步骤实现： 
　　1.建立网络连接 
　　客户端网络编程的第一步都是建立网络连接。在建立网络连接时需要指定连接的服务器的IP地址和端口号，建立完成以后，会形成一条虚拟的连接，后续的操作就可以通过该连接实现数据交换了。 
　　2.交换数据 
　　连接建立以后，就可以通过这个连接交换数据了，交换数据严格要求按照请求响应模型进行，由客户端发送一个请求数据到服务器，服务器反馈一个响应数据后给客户端，如果客户端不发送请求则服务器就不响应。 
　　根据逻辑需要，可以多次交换数据，但是还是必须遵循请求响应模型。 
　　3.关闭网络连接 
　　在数据交换完成后，关闭网络连接，释放程序占用的端口、内存等系统资源，结束网络编程。 
　　最基本的步骤一般都是这三个步骤，在实际实现时，步骤2会出现重复，在进行代码组织时，由于网络编程是比较耗时的操作，所以一般开启专门的现场进行网络通讯。

　　1.4服务器端网络编程步骤 
　　服务器是指网络编程中被等待连接的程序，服务器端一般实现程序的核心逻辑以及数据存储等核心功能。服务器端的编程步骤和客户端不同，是由四个步骤实现，依次是： 
　　1.监听端口 
　　服务器端属于被动等待连接，所以服务器端启动以后，不需要发起连接，而只需要监听本地计算机的某个固定端口即可。这个端口就是服务器端开放给客户端的端口，服务器端程序运行的本地计算机的IP地址就是服务器端程序的IP地址。 
　　2.获得连接 
　　当客户端连接到服务器端时，服务器端就可以获得一个连接，这个连接包含客户端信息，例如客户端IP地址等，服务器端和客户端通过该连接进行数据交换。 
　　一般在服务器端编程中，当获得连接时，需要开启专门的线程处理该连接，每个连接都由独立的线程实现。 
　　3.交换数据 
　　服务器端通过获得的连接进行数据交换。服务器端的数据交换步骤是首先接收客户端发送过来的数据，然后进行逻辑处理，再把处理以后的结果数据发送给客户端。简单来说，就是先接收再发送，这个和客户端的数据交换顺序不同。 
　　其实，服务器端获得的连接和客户端的连接是一样的，只是数据交换的步骤不同。当然，服务器端的数据交换也是可以多次进行的。在数据交换完成以后，关闭和客户端的连接。 
　　4.关闭连接 
　　当服务器程序关闭时，需要关闭服务器端，通过关闭服务器端使得服务器监听的端口以及占用的内存可以释放出来，实现了连接的关闭。 
　　其实服务器端编程的模型和呼叫中心的实现是类似的，例如移动的客服电话10086就是典型的呼叫中心，当一个用户拨打10086时，转接给一个专门的客服人员，由该客服实现和该用户的问题解决，当另外一个用户拨打10086时，则转接给另一个客服，实现问题解决，依次类推。 
　　在服务器端编程时，10086这个电话号码就类似于服务器端的端口号码，每个用户就相当于一个客户端程序，每个客服人员就相当于服务器端启动的专门和客户端连接的线程，每个线程都是独立进行交互的。 
　　这就是服务器端编程的模型，只是TCP方式是需要建立连接的，对于服务器端的压力比较大，而UDP是不需要建立连接的，对于服务器端的压力比较小罢了。 
　　总之，无论使用任何语言，任何方式进行基础的网络编程，都必须遵循固定的步骤进行操作，在熟悉了这些步骤以后，可以根据需要进行逻辑上的处理，但是还是必须遵循固定的步骤进行。 
　　其实，基础的网络编程本身不难，也不需要很多的基础网络知识，只是由于编程的基础功能都已经由API实现，而且需要按照固定的步骤进行，所以在入门时有一定的门槛，希望下面的内容能够将你快速的带入网络编程技术的大门。
```

### 二、Java网络编程技术

　和网络编程有关的基本API位于Java.NET包中。

介绍一下基础的网络类-InetAddress类。该类的功能是代表一个IP地址，并且将IP地址和域名相关的操作方法包含在该类的内部。

```java
package com.ahhf.ljxbw.socket;

import java.net.InetAddress;
import java.net.UnknownHostException;

public class InetAddressDemo {

	public static void main(String args[]) throws UnknownHostException {
		InetAddress inetAddress = InetAddress.getByName("www.baidu.com");
		System.out.println(inetAddress);
		InetAddress inetAddress1 = InetAddress.getByName("www.stubook.com.cn");
		System.out.println(inetAddress1);
		InetAddress inet2 = InetAddress.getByName("127.0.0.1");
		System.out.println(inet2);
		InetAddress inet3 = InetAddress.getLocalHost();
		System.out.println("---"+inet3);
		String host = inet3.getHostName();
		System.out.println("域名：" + host);
	}
}
/*
www.baidu.com/111.13.100.91
www.stubook.com.cn/59.110.148.91
/127.0.0.1
---YBSNRH3JH6DY110/192.168.88.173
域名：YBSNRH3JH6DY110

*/



/// 说明：由于该代码中包含一个互联网的网址，所以运行该程序时需要联网，否则将产生异常。 
```

### 三、TCP编程 

在Java语言中，对于TCP方式的网络编程提供了良好的支持。

在实际实现时，以java.net.socket类代表客户端连接，以java.net.ServerSocket类作为服务器端连接。

在进行网络编程时，底层网络通讯的细节已经实现了比较高的封装，所以在程序员实际编程时，**只需要指定IP地址和端口号就可以建立连接了**。正是由于这种***高度的封装***，一方面，简化了Java语言网络编程的难度，另外也使得使用Java语言进行网络编程无法深入到网络的底层，所以使用Java语言进行网络底层系统编程很困难，具体点说，***Java语言无法事先底层的网络嗅探以及获得IP包结构等消息***。但是由于Java语言的网络编程比较简答，所以还是获得了广泛的使用。 

在使用TCP方式进行网络编程时，需要按照前面介绍的网络编程的步骤进行，下面分别介绍一下在Java语言中客户端和服务器端的实现步骤。在客户端网络编程中，首先需要**建立连接**，在Java API中以及java.net.socket类的对象代表网络连接，所以建立客户端网络连接，也就是创建Socket类型的对象，该对象代表网络连接，示例如下： 
　　Socket socket1=new Socket(“192.168.1.103”,10000); 
　　Socket socket2=new Socket(“www.sohu.com”,80); 
　　上面的代码中，socket1实现的是连接到IP地址是192.168.1.103的计算机的10000号端口，而socket2实现的是连接到域名是www.sohu.com的计算机的80号端口，至于底层网络如何实现建立连接，对于程序员来说是完全透明的。如果建立连接时，本机网络不通，或服务器端程序未开启，则会抛出异常。 
　　连接一旦建立，则完成了客户端编程的第一步，紧接着的步骤就是按照“请求-响应”模型进行网络数据交换，在Java语言中，数据传输功能由Java IO实现，也就是说只需要**从连接中获得输入流和输出流即可，然后将需要发送的数据写入连接对象的输出流中，在发送完成后从输入流中读取数据即可**。示例代码如下： 
　　OutputStream os=socket1.getOutputStream(); 
　　InputStream is=socket1,getInputStream(); 
　　上面的代码中，分别从socket1这个连接对象获得了输出流和输入流对象，在整个网络编程中，后续的数据交换就变成了IO操作，也就是遵循“请求-响应”模式的规定，先向输出流中写入数据，这些数据会被系统发送出去，然后再从输入流中读取服务器端的反馈信息，这样就完成了一次数据交换工作，当然这个数据交换可以多次进行。 
　　这里获得的只是最基本的输出流和输入流对象，还可以根据前面学习到的IO知识，使用流的嵌套将这些获得的基本流对象转换成需要的装饰流对象，从而方便数据的操作。 
　　最后当数据交换完成以后，**关闭网络连接，释放网络连接占用的系统端口和内存等资源**，完成网络操作，示例代码如下： 
　　socket1.close（）; 

```java
package tcp;
import java.io.*;
import java.net.*;
/**
 * 简单的Socket客户端
 * 功能为：发送字符串“Hello”到服务器端，并打印出服务器端的反馈
 */
public class SimpleSocketClient {
         public static void main(String[] args) {
                   Socket socket = null;
                   InputStream is = null;
                   OutputStream os = null;
                   //服务器端IP地址
                   String serverIP = "127.0.0.1";
                   //服务器端端口号
                   int port = 10000;
                   //发送内容
                   String data = "Hello";
                   try {
                            //建立连接
                            socket = new Socket(serverIP,port);
                            //发送数据
                            os = socket.getOutputStream();
                            os.write(data.getBytes());
                            //接收数据
                            is = socket.getInputStream();
                            byte[] b = new byte[1024];
                            int n = is.read(b);
                            //输出反馈数据
                            System.out.println("服务器反馈：" + new String(b,0,n));
                   } catch (Exception e) {
                            e.printStackTrace(); //打印异常信息
                   }finally{
                            try {
                                     //关闭流和连接
                                     is.close();
                                     os.close();
                                     socket.close();
                            } catch (Exception e2) {}
                   }
         }
}
```



在服务器端程序编程中，由于服务器端实现的是被动等待连接，所以服务器端编程的第一个步骤是监听端口，也就是监听是否有客户端连接到达。实现服务器端监听的代码为： 
　　ServerSocket ss = new ServerSocket(10000); 
　　该代码实现的功能是监听当前计算机的10000号端口，如果在执行该代码时，10000号端口已经被别的程序占用，那么将抛出异常。否则将实现监听。 
　　服务器端编程的第二个步骤是获得连接。该步骤的作用是当有客户端连接到达时，建立一个和客户端连接对应的Socket连 接对象，从而释放客户端连接对于服务器端端口的占用。实现功能就像公司的前台一样，当一个客户到达公司时，会告诉前台我找某某某，然后前台就通知某某某， 然后就可以继续接待其它客户了。通过获得连接，使得客户端的连接在服务器端获得了保持，另外使得服务器端的端口释放出来，可以继续等待其它的客户端连接。 实现获得连接的代码是： 
　　 Socket socket = ss.accept(); 
　　 该代码实现的功能是获得当前连接到服务器端的客户端连接。需要说明的是accept和前面IO部分介绍的read方法一样，都是一个阻塞方法，也就是当无连接时，该方法将阻塞程序的执行，直到连接到达时才执行该行代码。另外获得的连接会在服务器端的该端口注册，这样以后就可以通过在服务器端的注册信息直接通信，而注册以后服务器端的端口就被释放出来，又可以继续接受其它的连接了。 
　　 连接获得以后，后续的编程就和客户端的网络编程类似了，这里获得的Socket类型的连接就和客户端的网络连接一样了，只是服务器端需要首先读取发送过来的数据，然后进行逻辑处理以后再发送给客户端，也就是交换数据的顺序和客户端交换数据的步骤刚好相反。这部分的内容和客户端很类似，所以就不重复了，如果还不熟悉，可以参看下面的示例代码。 
　　最后，在服务器端通信完成以后，关闭服务器端连接。实现的代码为：ss.close(); 

```java
package tcp;

import java.io.*;
import java.net.*;
/**
 * echo服务器
 * 功能：将客户端发送的内容反馈给客户端
 */
public class SimpleSocketServer {
         public static void main(String[] args) {
                   ServerSocket serverSocket = null;
                   Socket socket = null;
                   OutputStream os = null;
                   InputStream is = null;
                   //监听端口号
                   int port = 10000;
                   try {
                            //建立连接
                            serverSocket = new ServerSocket(port);
                            //获得连接
                            socket = serverSocket.accept();
                            //接收客户端发送内容
                            is = socket.getInputStream();
                            byte[] b = new byte[1024];
                            int n = is.read(b);
                            //输出
                            System.out.println("客户端发送内容为：" + new String(b,0,n));
                            //向客户端发送反馈内容
                            os = socket.getOutputStream();
                            os.write(b, 0, n);
                   } catch (Exception e) {
                            e.printStackTrace();
                   }finally{
                            try{
                                     //关闭流和连接
                                     os.close();
                                     is.close();
                                     socket.close();
                                     serverSocket.close();
                            }catch(Exception e){}
                   }
         }
}
```



#### 1、如何复用Socket连接？ 

　　在前面的示例中，客户端中建立了一次连接，只发送一次数据就关闭了，这就相当于拨打电话时，电话打通了只对话一次就关闭了，其实更加常用的应该是拨通一次电话以后多次对话，这就是复用客户端连接。 
　　那么如何实现建立一次连接，进行多次数据交换呢？其实很简单，建立连接以后，将数据交换的逻辑写到一个循环中就可以了。这样只要循环不结束则连接就不会被关 闭。按照这种思路，可以改造一下上面的代码，让该程序可以在建立连接一次以后，发送三次数据，当然这里的次数也可以是多次，示例代码如下：

```java

package tcp;
import java.io.*;
import java.net.*;
/**
 * 复用连接的Socket客户端
 * 功能为：发送字符串“Hello”到服务器端，并打印出服务器端的反馈
 */
public class MulSocketClient {
         public static void main(String[] args) {
                   Socket socket = null;
                   InputStream is = null;
                   OutputStream os = null;
                   //服务器端IP地址
                   String serverIP = "127.0.0.1";
                   //服务器端端口号
                   int port = 10000;
                   //发送内容
                   String data[] ={"First","Second","Third"};
                   try {
                            //建立连接
                            socket = new Socket(serverIP,port);
                            //初始化流
                            os = socket.getOutputStream();
                            is = socket.getInputStream();
                            byte[] b = new byte[1024];
                            for(int i = 0;i < data.length;i++){
                                     //发送数据
                                     os.write(data[i].getBytes());
                                     //接收数据
                                     int n = is.read(b);
                                     //输出反馈数据
                                     System.out.println("服务器反馈：" + new String(b,0,n));
                            }
                   } catch (Exception e) {
                            e.printStackTrace(); //打印异常信息
                   }finally{
                            try {
                                     //关闭流和连接
                                     is.close();
                                     os.close();
                                     socket.close();
                            } catch (Exception e2) {}
                   }
         }
}
```













## SQL

```doc
SQL中PK，FK意思：
--主键
constraint PK_字段 primary key(字段),
--唯一约束
constraint UK_字段 unique key(字段),
--默认约束
constrint DF_字段 default('默认值') for 字段,
--检查约束
constraint CK_字段 check(约束。如：len(字段)>1),
--主外键关系
constraint FK_主表_从表 foreign(外键字段) references 主表(主表主键字段)
```





## Shiro

## 一、简介



Shiro可以帮助我们完成：认证、授权、加密、会话管理、与Web集成、缓存等。

### 1.Shiro的API

```doc

Authentication：身份认证/登录，验证用户是不是拥有相应的身份；

Authorization：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

Session Manager：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；

Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

Web Support：Web支持，可以非常容易的集成到Web环境；

Caching：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；

Concurrency：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

Testing：提供测试支持；

Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

```

### 2.从外部和内部来看看Shiro的架构

对于一个好的框架，从外部来看应该具有非常简单易于使用的API，且API契约明确；从内部来看的话，其应该有一个可扩展的架构，即非常容易插入用户自定义实现.

#### 1.1从外部看如何使用Shiro完成工作

````doc
应用代码直接交互的对象是Subject，也就是说Shiro的对外API核心就是Subject

Subject：主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject.
所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；
可以把Subject认为是一个门面；SecurityManager才是实际的执行者；


SecurityManager：安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互.

Realm：域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。


````

##### 1.1.1简单的实现逻辑

```doc
也就是说对于我们而言，最简单的一个Shiro应用：

1、应用代码通过Subject来进行认证和授权，而Subject又委托给SecurityManager；

2、我们需要给Shiro的SecurityManager注入Realm，从而让SecurityManager能得到合法的用户及其权限进行判断。

从以上也可以看出，Shiro不提供维护用户/权限，而是通过Realm让开发人员自己注入。
```

#### 1.2从内部看

```doc
Subject：主体，可以看到主体可以是任何可以与应用交互的“用户”；

SecurityManager：相当于SpringMVC中的DispatcherServlet或者Struts2中的FilterDispatcher；是Shiro的心脏；所有具体的交互都通过SecurityManager进行控制；它管理着所有Subject、且负责进行认证和授权、及会话、缓存的管理。

Authenticator：认证器，负责主体认证的，这是一个扩展点，如果用户觉得Shiro默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；

Authrizer：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；

Realm：可以有1个或多个Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是JDBC实现，也可以是LDAP实现，或者内存实现等等；由用户提供；注意：Shiro不知道你的用户/权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的Realm；

SessionManager：如果写过Servlet就应该知道Session的概念，Session呢需要有人去管理它的生命周期，这个组件就是SessionManager；而Shiro并不仅仅可以用在Web环境，也可以用在如普通的JavaSE环境、EJB等环境；所有呢，Shiro就抽象了一个自己的Session来管理主体与应用之间交互的数据；这样的话，比如我们在Web环境用，刚开始是一台Web服务器；接着又上了台EJB服务器；这时想把两台服务器的会话数据放到一个地方，这个时候就可以实现自己的分布式会话（如把数据放到Memcached服务器）；

SessionDAO：DAO大家都用过，数据访问对象，用于会话的CRUD，比如我们想把Session保存到数据库，那么可以实现自己的SessionDAO，通过如JDBC写到数据库；比如想把Session放到Memcached中，可以实现自己的Memcached SessionDAO；另外SessionDAO中可以使用Cache进行缓存，以提高性能；

CacheManager：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能

Cryptography：密码模块，Shiro提高了一些常见的加密组件用于如密码加密/解密的。
```





## 二、身份验证

```doc
身份验证-------------即在应用中谁能证明他就是他本人。一般提供如他们的身份ID一些标识信息来表明他就是他本人，如提供身份证，用户名/密码来证明。

在shiro中，用户需要提供principals （身份）和credentials（证明）给shiro，从而应用能验证用户身份：
principals：身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个principals，但只有一个Primary principals，一般是用户名/密码/手机号。

credentials：证明/凭证，即只有主体知道的安全值，如密码/数字证书等。

最常见的principals和credentials组合就是用户名/密码了。
```

## 

