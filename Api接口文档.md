# 接口详解

## /login 登录

### 请求参数

**username**：用户名

**password**：密码

**savePassword**：保存密码（true/false）

### 返回值

true和false

## /register 注册

### 请求参数

**username**：用户名

**password**：密码

### 返回值

true和false

## /allUserNumbers 所有注册人数

## /addStudent 添加学生

sid=2&name=2&sex=2&phone=2&email=2&qq=2&wx=2&address=2&profession=2&identificationnumber=2&intime=2&paymoney=2

##  /addTeacher 添加老师

tid:2
name:2
sex:2    
phone:2
address:2
coursename:2

tid=2&name=2&sex=2&phone=2&address=2&coursename=2

## /updateTeacher 更新老师信息

tid=2&name=22&sex=22&phone=22&address=22&coursename=22

## /deleteTeacher 删除老师

tid=2

## /addCourse 添加课程

cid=2&coursename=2&teachername=2&time=2&location=2&numberofweek=2&type=2&credit=2