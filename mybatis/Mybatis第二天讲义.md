# Mybatis第二天讲义

1.  回顾mybatis的自定义再分析和环境搭建+完善基于注解的mybatis
2.  mybatis的curd（基于代理dao的方式）
3.  mybatis中的参数深入及结果集的深入
4.  mybatis中基于传统dao的方式（编写dao的实现类）--- 了解内容
5.  mybatis中的配置（主配置文件：SqlMapConfig.xml）
    -   properties标签
    -   typeAliases标签
    -   mappers标签

## 回顾mybatis的自定义

1.  SqlSessionFactoryBuilder接收SqlMapConfig.xml文件流，构建出SqlSessionFactory对象
2.  SqlSessionFatory读取SqlMapConfig.xml中连接数据库和mapper映射信息。用来生产出真正操作数据库的SqlSession对象
3.  SqlSession对象有两大作用：（注：无论哪个分支，除了连接数据库信息，还需要得到Sql语句）
    1.  生成接口代理对象
    2.  定义通用增删改查方法
4.  作用1：在SqlSessionImpl对象的getMapper中分两步来实现
    1.  先用SqlSessionFactory读取的数据库连接信息创建Connection对象
    2.  通过jdk代理模式创建出代理对象作为getMapper方法返回值，这里主要工作是在创建代理对象时第三个参数处理类里面得到sql语句，执行对应CRUD操作
    
    作用2：在SqlSessionImpl对象中提供selectList()方法，【当然实际mybatis框架中还有selectOne，insert等方法】这些方法内也分两步
    1.	用SqlSessionFactory读取的数据库连接信息创建出jdbc的Connection对象
    2.	直接得到sql语句，使用jdbc的Connection对象进行对应的CRUD操作
5.	封装结果集：无论使用分支一生成代理对象，还是直接使用分支二提供的通用CRUD方法，我们倒要对返回的数据库结果集进行封装，变成java对象返回给调用者。所以我们还必须要知道调用者所需要的返回类型


## Mybatis的CRUD

1.  查询返回一行一列和占位符分析

    1.  ```sql
        select * from user where username like #{name}
        ```

        ​	此方法使用的是PrepatedStatement的参数占位符（安全，常用）

    2.  ```sql
        select * from user where username like '%${value}%'
        ```

        ​	此方法使用的是Statement对象的字符串拼接SQL（不安全，知道即可）

2.  新增用户id的返回值

    -   ```xml
        <!--    保存用户-->    
        <insert id="saveUser" parameterType="mybatiscrud.domain.User">
            --     配置插入操作后，获取插入数据的id        
            <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
                select last_insert_id()        
            </selectKey>        
            insert into user(username,password)values(#{username},#{password})    
        </insert>
        ```

## Mybatis的参数深入

-   OGNL表达式：Object Graphic Navigation Language
    -   它是通过对象的取值方法来获取数据。在写法上把get省略了。
    -   比如：我们获取用户的名称
        -   类中的写法：user.getUsername()
        -   OGNL表达式写法：user.username
    -   mybatis中为什么能直接写username，而不用user.呢:
        -   因为在parameterType中已经提供了属性所属的类，所以此时不需要写对象名

### 当对应的属性名和数据库的列名不一致时的处理方法：配置XML

1.  

```xml
<!--    配置 查询结果的列名和实体类的属性名的对应关系-->    
<resultMap id="userMap" type="mybatiscrud.domain.User">
    <!--        主键字段的对应-->        
    <id property="userId" column="id"></id>
    <!--        非主键字段的对应-->        
    <result property="userName" column="username"></result>        
    <result property="password" column="password"></result>    
</resultMap>    
<!--    查询所有-->    
<select id="findAll" resultMap="userMap">        
    select * from user     
</select>
```

2.    

```xml
<!--    查询所有-->  
<select id="findAll" resultMap="userMap">        
    select id as UserId,username as userName,password as password from user     
</select>
```

##  Mybatis中使用dao实现类的执行过程分析

```java
@Override
public List<User> findAll() {    
    session.selectList();
}
```

```java
@Override
public void saveUser(User user) {
    session.insert();
}
```
```java
@Override
 public void updateUser(User user) {
    session.update();
}
```
```java
@Override
public void deleteUser(Integer id) {
    session.update();
}
```
```java
@Override
public User findById(Integer id) {   
    User user = session.selectOne();
}
```
```java
    public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
        List var5;
        try {
            MappedStatement ms = this.configuration.getMappedStatement(statement);
            var5 = this.executor.query(ms, this.wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
        } catch (Exception var9) {
            throw ExceptionFactory.wrapException("Error querying database.  Cause: " + var9, var9);
        } finally {
            ErrorContext.instance().reset();
        }

        return var5;
    }
```



**PreparedStatement对象它的执行方法：**

	-	execute：它能执行CRUD中的任意一种语句，它的返回值时boolean类型，表示是否有结果集。有结果集是true，没有结果集是false
	-	executeUpdate：它只能执行CUD语句，查询语句无法只想。他的返回值是影响数据库记录的行数
	-	executeQuery：它只能执行SELECT语句，无法执行增删改。执行结果封装的结果集ResultSet对象