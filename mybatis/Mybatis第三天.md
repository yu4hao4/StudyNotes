# 内容

1.  mybatis中的连接池以及事务控制（原理部分了解，应用部分会用）
    -   mybatis中连接池使用及分析
    -   mybatis事务控制的分析
2.  mybatis基于XML配置的动态SQL语句使用（会用）
    -   mappers配置文件中的几个标签：
        -   <if>
        -   <where>
        -   <foreach>
        -   <sql>
3.  mybatis中的多表操作（掌握应用）
    -   一对多
    -   一对一（？）
    -   多对多

<hr/>
## mybatis连接池与事务深入

### 连接池

1.  连接池：
    -   我们在实际开发中都会使用连接池。
    -   因为它可以减少我们获取连接所消耗的时间。
    -   连接池就是用于存储连接的一个容器
    -   容器其实就是一个集合对象，该集合必须是线程安全的，不能两个线程拿到同一连接
    -   该集合还必须实现队列的特性：先进先出
2.  mybatis中的连接池
    -   mybatis连接池提供了3种方式的配置：
        -   配置的位置
            -   主配置文件SqlMapConfig.xml中的dataSource标签，type属性就是表示采用何种连接池方式。
        -   type属性的取值：
            -   POOLED：
                采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现
            -   UNPOOLED ：
                采用传统的获取连接的方式，虽然也实现了javax.sql.DataSource接口，但是并没有使用池的思想。
            -   JNDI：
                采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到的DataSource是不一样的。
                注意：如果不是web或者maven的war工厂，是不能使用的。
                我们课程中使用的是tomcat服务器，采用连接池就是dbcp连接池

<hr/>
1.  UNPOOLED ：

    -   ```java
        private Connection doGetConnection(Properties properties) throws SQLException {
            this.initializeDriver();    
            Connection connection = DriverManager.getConnection(this.url, properties);    
            this.configureConnection(connection);    
            return connection;
        }
        ```

        直接获取一个连接，用完就关闭

2.  POOLED：

    -   ```java
        private PooledConnection popConnection(String username, String password) throws SQLException {
                boolean countedWait = false;
                PooledConnection conn = null;
                long t = System.currentTimeMillis();
                int localBadConnectionCount = 0;
        
                while(conn == null) {
                    synchronized(this.state) {
                        PoolState var10000;
                        if (!this.state.idleConnections.isEmpty()) {
                            conn = (PooledConnection)this.state.idleConnections.remove(0);
                            if (log.isDebugEnabled()) {
                                log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
                            }
                        } else if (this.state.activeConnections.size() < this.poolMaximumActiveConnections) {
                            conn = new PooledConnection(this.dataSource.getConnection(), this);
                            if (log.isDebugEnabled()) {
                                log.debug("Created connection " + conn.getRealHashCode() + ".");
                            }
                        } else {
                            PooledConnection oldestActiveConnection = (PooledConnection)this.state.activeConnections.get(0);
                            long longestCheckoutTime = oldestActiveConnection.getCheckoutTime(); 
                            if (longestCheckoutTime > (long)this.poolMaximumCheckoutTime) {
                                ++this.state.claimedOverdueConnectionCount;
                                var10000 = this.state;
                                var10000.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
                                var10000 = this.state;
                                var10000.accumulatedCheckoutTime += longestCheckoutTime;
                                this.state.activeConnections.remove(oldestActiveConnection);
                                if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                                    oldestActiveConnection.getRealConnection().rollback();
                                }
        
                                conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
                                oldestActiveConnection.invalidate();
                                if (log.isDebugEnabled()) {
                                    log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");
                                }
                            } else {
                                try {
                                    if (!countedWait) {
                                        ++this.state.hadToWaitCount;
                                        countedWait = true;
                                    }
        
                                    if (log.isDebugEnabled()) {
                                        log.debug("Waiting as long as " + this.poolTimeToWait + " milliseconds for connection.");
                                    }
        
                                    long wt = System.currentTimeMillis();
                                    this.state.wait((long)this.poolTimeToWait);
                                    var10000 = this.state;
                                    var10000.accumulatedWaitTime += System.currentTimeMillis() - wt;
                                } catch (InterruptedException var16) {
                                    break;
                                }
                            }
                        }
        
                        if (conn != null) {
                            if (conn.isValid()) {
                                if (!conn.getRealConnection().getAutoCommit()) {
                                    conn.getRealConnection().rollback();
                                }
        
                                conn.setConnectionTypeCode(this.assembleConnectionTypeCode(this.dataSource.getUrl(), username, password));
                                conn.setCheckoutTimestamp(System.currentTimeMillis());
                                conn.setLastUsedTimestamp(System.currentTimeMillis());
                                this.state.activeConnections.add(conn);
                                ++this.state.requestCount;
                                var10000 = this.state;
                                var10000.accumulatedRequestTime += System.currentTimeMillis() - t;
                            } else {
                                if (log.isDebugEnabled()) {
                                    log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");
                                }
        
                                ++this.state.badConnectionCount;
                                ++localBadConnectionCount;
                                conn = null;
                                if (localBadConnectionCount > this.poolMaximumIdleConnections + 3) {
                                    if (log.isDebugEnabled()) {
                                        log.debug("PooledDataSource: Could not get a good connection to the database.");
                                    }
        
                                    throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
                                }
                            }
                        }
                    }
                }
        
                if (conn == null) {
                    if (log.isDebugEnabled()) {
                        log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
                    }
        
                    throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
                } else {
                    return conn;
                }
            }
        ```

        拿连接的过程：

        ​	首先判断空闲池，如果空闲池还有连接的话，直接拿出一个用。

        ​	当空闲池的连接已经没有了，查看活动连接池中是否已经到了最大的容纳数量，（？）判断活动池中哪个连接是最早进来的，并将其返回给获取连接者

### 事务

3.  mybatis中的事务

    -   什么是事务

    -   事务的四大特性ACID

    -   不考虑隔离性会产生的3个问题

    -   解决办法：四种隔离级别

    -   它是通过sqlsession对象的commit方法和rollback方法实现事务的提交和回滚

        ```java
        public void commit() throws SQLException {
                if (this.connection != null && !this.connection.getAutoCommit()) {
                    if (log.isDebugEnabled()) {
                        log.debug("Committing JDBC Connection [" + this.connection + "]");
                    }
        
                    this.connection.commit();
                }
        
            }
        
        public void rollback() throws SQLException {
            if (this.connection != null && !this.connection.getAutoCommit()) {
                if (log.isDebugEnabled()) {
                    log.debug("Rolling back JDBC Connection [" + this.connection + "]");
                }
        
                this.connection.rollback();
            }
        
        }
        ```




## mybatis的多表关联查询

表之间的关系有几种：

-   一对多
-   多对一
-   一对一
-   多对多

举例：

-   用户和订单就是一对多
-   订单和用户就是多对一
    -   一个用户可以下多个订单
    -   多个订单属于同一用户
-   人和身份证号就是一对一
    -   一个人只能有一个身份证号
    -   一个身份证号只能属于一个人
-   老师和学生之间就是多对多
    -   一个学生可以被多个老师教过
    -   一个老师可以教多个学生

特例：

-   如果拿出每一个订单，他都只能属于一个用户。
-   所以Mybatis就把多对一看成了一对一。

mybatis中的多表查询：

-   示例：用户和账户

    -   一个用户可以有多个账户
    -   一个账户只能属于一个用户（多个账户也可以属于同一个用户）

-   步骤：

    1.  建立两张表：用户表，账户表

        让用户表和账户表之间具备一对多的关系：需要使用外键在账户表中添加 

    2.  建立两个实体类：用户实体类和账户实体类
        让用户和账户的实体类能够体现出来一对多的关系

    3.  建立两个配置文件
        用户的配置文件
        账户的配置文件

    4.  实现配置：
        当我们查询用户时，可以同时得到用户下所包含的账户信息
        当我们查询账户时，可以同时的得到账户的所属用户信息

-   示例：用户和角色

    -   一个用户可以有多个角色
    -   一个角色赋予多个用户

-   步骤：

    1.  建立两张表：用户表，角色表

        让用户表和角色表之间具备多对多的关系：需要使用中间表，中间表中包含各自的主键，在中间表中是外键。 

    2.  建立两个实体类：用户实体类和账户实体类
        让用户和角色的实体类能够体现出来多对多的关系，各自包含对方一个集合引用

    3.  建立两个配置文件
        用户的配置文件
        账户的配置文件

    4.  实现配置：
        当我们查询用户时，可以同时得到用户下所包含的角色信息
        当我们查询角色时，可以同时的得到角色的所赋予的用户信息

## JNDI数据源

| map结构                                                      |                                                |
| ------------------------------------------------------------ | ---------------------------------------------- |
| key:存的是路径+名称                                          | value:存的就是是数据<br />在JNDI中存的就是对象 |
| HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Accessibility\ATs\soundsentry：description | soundsentry                                    |
| HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Accessibility\ATs\stickykeys：description | stickykeys                                     |

| tomcat服务器一启动                          |                                                              |
| ------------------------------------------- | ------------------------------------------------------------ |
| key：是一个字符串                           | value:是一个Object                                           |
| directory是固定的<br />name是可以自己指定的 | 要存放是什么对象是可以指定的，指定的方式是通过配置文件的方式 |

