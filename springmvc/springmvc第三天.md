# 搭建整合环境

1.  整合说明：SSM整合可以使用多种方式，咱们会选择XML+注解的方式

2.  整合的思路

    1.  先搭建整合的环境
    2.  先把Spring的配置搭建完成
    3.  再使用Spring整合SpringMVC框架
    4.  最后使用Spring整合Mybatis框架

3.  创建数据库和表结构

    1.  语句

        ```sql
        create database ssm;
        use ssm;
        create table account(
        	id int primary key auto_increment,
            name varchar(20),
            money double
        );
        ```

4.  阿里云使用docker搭建Mysql5.1数据库

    1.  我拉取了一个mysql5.1版本的数据库

        ```sh
        docker pull eccube/mysql51
        ```

    2.  运行起来

        ```shell
        docker run -d -p 13306:3306 eccube/mysql51:latest
        ```

    3.  保存成自己的msyql数据库

        ```shell
        docker commit 565b1122342b(此处是容器id，可以根据自身的情况进行调整，docker ps -a查看所有运行过的容器)mysql:1
        ```

    4.  运行前面保存的mysql镜像

        ```shell
        docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d -p 13306:3306 mysql:1
        ```

        搭建好数据库之后，可以通过数据库连接工具，测试，mysql的连接驱动是5.1的，如果服务器是阿里云的，需要去配置一下安全组规则，设置13306端口可以进出。
    
5.  
