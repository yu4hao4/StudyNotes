# 内容介绍

1.  Mybatis中的延迟加载

    什么是延迟加载
    什么是立即加载

2.  Mybatis中的缓存
    什么是缓存
    为什么使用缓存
    什么样的数据能使用缓存，什么样的数据不能使用
    Mybatis中的一级缓存和二级缓存

3.  Mybatis中的注解开发
    环境搭建
    单表CRUD操作（代理Dao方式）
    多表查询操作
    缓存的配置

## Mybatis中的延迟加载

-   问题：
    -   在一对多中，当我们有一个用户，它有100个账户
        -   在查询用户的时候，要不要把关联的账户查出来？
            在查询用户时，用户下的账户信息应该说是，什么时候使用，什么时候查询的。
        -   在查询账户的时候，要不要把关联的用户查出来？
            在查询账户时，账户的所属用户信息应该是随着账户查询时一起查询出来
-   什么是延迟加载
    在真正使用数据时才发起查询，不用的时候不查询。按需加载（懒加载）
-   什么时立即加载
    不管用不用，只要一调用方法，马上发起查询。
-   在对应的四种表关系中：一对多，多对一，一对一，多对多
    一对多，多对多：通常情况下我们都是采用延迟加载。
    多对一，一对一：通常情况下我们都是采用立即加载。

## Mybatis中的缓存

-   什么是缓存
    存在于内存中的临时数据。

-   为什么使用缓存
    减少和数据库的交互次数，提高执行效率。

-   什么样的是数据能使用缓存，什么样的数据不能使用
    适用于缓存：

    -   经常查询并且不经常改变的。
    -   数据的正确与否对最终结果影响不大的。

    不适用于缓存：

    -   经常改变的数据
    -   数据的正确与否对最终结果影响很大的。
    -   例如：商品的库存，银行的汇率，股市的牌价。

-   Mybatis中的一级缓存和二级缓存

    -   一级缓存：
        -   它指的是Mybatis中SqlSession对象的缓存。
        -   当我们执行查询之后，查询的结果会同时存入到SqlSession为我们提供一块区域中。
        -   该区域的结构是一个Map。当我们再次查询同样的数据，mybatis会先去sqlsession中查询是否有，有的话直接拿出来用。
        -   当SqlSession对象消失时，mybatis的一级缓存也就消失了。
        
    -   一级缓存分析：
    
        -   一级缓存时SqlSession范围的缓存，当调用SqlSession的修改，添加，删除，commit()，close()等方法时，就会清空一级缓存。
    
    -   二级缓存：
    
        -   它指的是Mybatis中的SqlSessionFactory对象的缓存。由同一个SqlSessionFactory对象创建的SqlSession共享其缓存
    
        -   二级缓存的使用步骤：
    
            	1. 让Mybatis框架支持二级缓存（在SqlMapConfig.xml中配置）
             	2. 让当前的映射文件支持二级缓存（在IUserDao.xml中配置）
             	3. 让当前的操作支持二级缓存（在select标签中配置）
    
        -   二级缓存：
    
            -   存放的内容时数据，而不是对象
    
                ```java
                /**
                 * 测试二级缓存
                 */
                @Test
                public void testFirstLevelCache(){
                    SqlSession sqlSession1 = factory.openSession();
                    IUserDao dao1 = sqlSession1.getMapper(IUserDao.class);
                    User user1 = dao1.findById(1);
                    System.out.println(user1);
                    //一级缓存消失
                    sqlSession1.close();
                
                    SqlSession sqlSession2 = factory.openSession();
                    IUserDao dao2 = sqlSession2.getMapper(IUserDao.class);
                    User user2 = dao2.findById(1);
                    System.out.println(user2);
                    sqlSession2.close();
                
                    System.out.println(user1 == user2);
                }
                ```
    
                ```
                cache.domain.User@26aa12dd
                cache.domain.User@e874448
                false
                ```
    
                SqlSessionFactory拿到数据之后，创建一个新的对象，并将数据存储到其中，所以两个对象不是一个

## Mybatis的注解开发

**一对多注解配置：**

```java
//一对多关系映射：一个用户对应多个账户
private List<Account> accounts;

public List<Account> getAccounts() {
    return accounts;
}

public void setAccounts(List<Account> accounts) {
    this.accounts = accounts;
}
/**
 * 查询所有用户
 * @return
 */
@Select("select * from user")
@Results(id="userMap",value = {
        @Result(id=true,column = "id",property = "id"),
        @Result(column = "username",property = "username"),
        @Result(column = "password",property = "password"),
        @Result(property = "accounts",column = "id",
                many = @Many(select = "annoonetomany.dao.IAccountDao.findAccountByUid",
                        fetchType = FetchType.LAZY))
})
List<User> findAll();
```

**一对一和多对一注解配置：**

```java
//多对一（mybatis中称之为一对一）的映射：一个账户只能属于一个用户
private User user;

public User getUser() {
    return user;
}

public void setUser(User user) {
    this.user = user;
}
/**
 * 查询所有账户，并且获取每个账户所属的用户信息
 * @return
 */
@Select("select * from account")
@Results(id="",value={
        @Result(id = true,column = "id",property = "id"),
        @Result(column = "uid",property = "uid"),
        @Result(column = "money",property = "money"),
        @Result(property = "user",column = "uid",
                one = @One(select="annoonetomany.dao.IUserDao.findById",
                        fetchType= FetchType.EAGER))
})
List<Account> findAll();
```

## Mybatis的注解开发使用二级缓存

````java
package annoonetomany.dao;

import annoonetomany.domain.User;
import org.apache.ibatis.annotations.*;
import org.apache.ibatis.mapping.FetchType;

import java.util.List;

/**
 * @author 喻浩
 * @create 2019-08-15-22:32
 * 用户的持久层接口
 * 在mybatis中针对CRUD一共有四个注解
 * @Select @Insert @Update @Delete
 */
//开启二级缓存
@CacheNamespace(blocking = true)
public interface IUserDao {

    /**
     * 查询所有用户
     * @return
     */
    @Select("select * from user")
    @Results(id="userMap",value = {
            @Result(id=true,column = "id",property = "id"),
            @Result(column = "username",property = "username"),
            @Result(column = "password",property = "password"),
            @Result(property = "accounts",column = "id",
                    many = @Many(select = "annoonetomany.dao.IAccountDao.findAccountByUid",
                            fetchType = FetchType.LAZY))
    })
    List<User> findAll();

    /**
     * 保存用户
     */
    @Insert("insert into user(username,password)values(#{username},#{password})")
    void saveUser(User user);

    /**
     * 根据id查用户
     * @param userId
     * @return
     */
    @Select("select * from user where id=#{id}")
    @ResultMap("userMap")
    User findById(Integer userId);

    /**
     * 根据用户名称模糊查询
     * @param username
     * @return
     */
//    @Select("select * from user where username like #{username}")
    @Select("select * from user where username like '%${value}%'")
    List<User> findByName(String username);

    /**
     * 查总用户数量
     * @return
     */
    @Select("select count(*) from user ")
    int findTotalUser();
}
````

**测试二级缓存**

```java
@Test
public void testFindOne(){
SqlSession session = factory.openSession();
IUserDao userDao = session.getMapper(IUserDao.class);;
User user = userDao.findById(1);
System.out.println(user);
//释放一级缓存
session.close();

//再次打开session
SqlSession session1 = factory.openSession();
IUserDao userDao1 = session1.getMapper(IUserDao.class);
User user1 = userDao1.findById(1);
System.out.println(user1);

session1.close();
}
```

