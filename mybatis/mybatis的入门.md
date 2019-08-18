#  5. mybatis的入门

## mybatis的环境搭建

第一步：创建maven工程并导入坐标

第二步：创建实体类和dao的接口

第三步：创建 mybatis 的主配置文件	SqlMapConfig.xml

第四步：创建映射配置文件	IUserDao.xml

### 环境搭建的注意事项

第一个：创建 IUserDao.xml 和 IUserDao.java 时名称是为了和我们之前的知识保持一致。
				在 Mybatis 中它把持久层的操作接口名称和映射文件也叫做：Mapper

​				所以：IUserDao 和 IUserMapper是一样的

第二个：在idea中创建目录的时候，它和包是不一样的

​				包在创建时：com.itheima.dao它是三级结构

​				目录在创建是：com.itheima.dao是一级目录

第三个：mybatis 的映射配置文件位置必须和dao接口的包结构相同

第四个：映射配置文件的 mapper 标签 namespace 属性的取值必须是dao接口的全限定类名



第五个：映射配置文件的操作配置（select），id属性的取值必须是dao接口的方法名

当我们遵从了第三，四，五点之后，我们在开发中就无需再写dao的实现类

## mybatis入门案例

第一步：读取配置文件

第二步：创建SqlSessionFactory工厂

第三步：创建SqlSession

第四步：创建Dao接口的代理对象

第五步：执行dao中的方法

第六步：释放资源

**注意事项：**

​				不要忘记在映射配置中告知 mybatis 要封装到哪个实体类中

​				配置的方式：指定实体类的全限定类名

### mybatis基于注解的入门案例：

把IUserDao.xml移除，在dao接口的方法上时使用@Select注解，并且指定SQL语句，同时需要在SqlMapConfig.xml中的mapper配置时，使用class属性指定dao接口的全限定类名



明确：

​			我们在实际开发中，都是越简便越好，所以都是采用不写dao实现类的方式。不管使用XML还是注解配置。但是Mybatis它是支持写dao实现类的。



```java
public static void main(String[] args) throws Exception {
        //1.读取配置文件
    /*
    *	绝对路径：d:/xxx/xxx.xml(x)
    *	相对路径：src/java/main/xxx.xml(x)
    *	第一个：使用类加载器，它只能读取类路径的配置文件
    *	第二个：使用ServletContext对象的getRealPath()
    */
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建SqlSesssionFactory工厂
    /*
    *	创建工厂mybatis使用了构建者模式
    *		优势：把对象的创建细节隐藏，让使用者直接调用方法即可拿到对象
    *	builder就是构建者
    */
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(in);
        //3.使用工厂生产SqlSession对象
    /*
    *	生产SqlSession使用了工厂模式
    *		优势：解耦（降低类之间的依赖关系）
    */
        SqlSession session = factory.openSession();
        //4.使用SqlSession创建dao接口的代理对象
    /*
    *	创建dao接口实现类使用了代理模式
    *		优势：不修改源码的基础上对已有方法增强
    */
        IUserDao userDao = session.getMapper(IUserDao.class);
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll();
        for(User user : users){
            System.out.println(user);
        }
        //6.释放资源
        session.close();
        in.close();
    }
```

# 6.自定义Mybatis的分析

mybatis在使用代理dao的方式实现增删改查时做什么事呢？

​	只有两件事：

​						第一：创建代理对象

​						第二：在代理对象中调用selectList

```xml
连接数据库的信息，有了他们就能创建Connection对象
<property name="driver" value="com.mysql.jdbc.Driver"></property>
                <property name="url" value="jdbc:mysql://localhost:3306/users?useSSL=false"></property>
                <property name="username" value="root"></property>
                <property name="password" value="root"></property>
```

```xml
有了它就有了映射配置信息
<mappers>
        <mapper resource="com/itheima/dao/IUserDao.xml"/>
</mappers>
```

```xml
有了它就有了执行的SQL语句，就可以获取PreparedSatement
<mapper namespace="com.itheima.dao.IUserDao">
    
    <select id="findAll" resultType="com.itheima.domain.User">
        select * from user 
    </select>
</mapper>此配置中还有封装的实体类全限定类名
```

**读取配置文件：用到的技术就是解析XML的技术，此处用的是dom4j解析xml技术**

1.  根据配置文件的信息创建Connection对象
    
    -  注册驱动，获取连接
    
2.  获取预处理对象PreparedSatement
    -  此时需要SQL语句
    -  conn.prepareStatement(sql);
    
3.  执行查询
    
    -   ResultSet resultSet = preparedStatement.executeQuery();
    
4.  遍历结果集用于封装
    -   List<E> list = new ArrayList();
    
    -   while(resultSet.next()){
        -   E element = (E)Class.forName(配置的全限定类名).newInstance();
        -   进行封装，把每个rs的内容都添加到element中
            -   此处使用反射封装
            -   我们的实体类属性和表中的列名是一致的
            -   于是我们就可以把表的列名看成是实体类的属性名称
            -   就可以使用反射的方式来根据名称获取每个属性，并把值赋进去
        -   把element加入到list中
        -   list.add(element)
    
    -   }
5.  返回list

    -   return list;

**要想让上方方法执行，我们需要给方法提供两个信息**

 1.    连接信息

 2.    映射信息

       -   它包含了两个部分

           1.  执行的SQL语句

           2.  封装结果的实体类全限定类名（把这两个信息组合起来定义成一个对象 -> Mapper

               1.  |              String              |               Mapper                |
                   | :------------------------------: | :---------------------------------: |
                   | com.itheima.dao.IUserDao.findAll | Mapper对象：包含全限定类名和sql语句 |

               



```java
 //4.使用SqlSession创建dao接口的代理对象
	IUserDao userDao = session.getMapper(IUserDao.class);

//根据dao接口的字节码创建dao的代理对象
public <T> T getMapper(Class<T> daoInterfaceClass){
    /**
    *	类加载器:它使用的和被代理对象是相同的类加载器
    *	代理对象要实现的接口：和被代理对象实现相同的接口
    *	如何代理：它就是增强的方法，我们需要自己来提供
    *		此处是一个InvocationHandler的接口，我们需要写一个该接口的实现类
    *		在实现类中调用selectList方法
    */
    Proxy.newProxyInstance(类加载器，代理对象要实现的接口字节码数组，如何代理);
}
```



自定义mybatis能通过入门案例看到的类

	- class	Resource
	- class    SqlSessionFactoryBuilder
	- interface SqlSessionFactory
	- interface SqlSession