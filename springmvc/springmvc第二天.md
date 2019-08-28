#  响应数据和结果视图

## 转发和重定向

```java
/**
* 返回值是void
* 请求转发一次请求，不用编写项目的名称
*/
@RequestMapping("/testVoid")
public void testVoid(HttpServletRequest request, HttpServletResponse response) throws Exception {
    System.out.println("testVoid方法执行了。。。");
    //编写请求转发的程序
    //        request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request, response);

    //        重定向
    //        response.sendRedirect(request.getContextPath()+"/index.jsp");

    //设置中文乱码
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=UTF-8");

    //直接会进行响应
    response.getWriter().print("你好 ");

    return;
}
```

## ResponseBody响应json数据

```java
	/**
     * 模拟异步请求响应
     * @param user
     * @return
     */
@RequestMapping("/testAjax")
public @ResponseBody User testAjax(@RequestBody User user){
    System.out.println("testAjax方法执行了。。。");
    //客户端发送ajax的请求，传的是json字符串，后端把json字符串封装到user对象中
    System.out.println(user);
    //做响应，模拟查数据库
    user.setUsername("gaga");
    user.setAge(10);
    return user;
}
```

```javascript
<script>
    //页面加载，绑定单击事件
    $(function () {
    $("#btn").click(function () {
        // alert("hello btn");
        // 发送ajax请求
        $.ajax({
            //编写json格式
            url:"user/testAjax",
            contentType:"application/json;charset=UTF-8",
            data:'{"username":"hehe","password":"wqw","age":30}',
            dataType:"json",
            type:"post",
            success:function (data) {
                //data服务器端相应的json数据，进行解析
                alert(data);
                alert(data.username);
                alert(data.password);
                alert(data.age);
            }
        })
    });
});
</script>
```

## 文件上传

### 文件上传的必要条件

1.  form表单的 enctype 取值必须是：multipart/form-data
    (默认值是：application/x-www-form-urlencoded)
    enctype:是表单请求正文的类型
2.  method 属性取值必须是Post
3.  提供一个文件选择域\<input type="file"/\>

**springmvc文件上传**

```java
/**
     * springmvc文件上传
     * @return
     */
    @RequestMapping("/fileUpload2")
    public String fileUpload2(HttpServletRequest request, MultipartFile upload) throws Exception {
        System.out.println("springmvc文件上传...");

        //使用fileupload组件完成文件上传
        //上传的文职
        String path = request.getSession().getServletContext().getRealPath("/uploads/");
        //判断，该路径是否存在
        File file = new File(path);
        if(!file.exists()){
            //创建该文件夹
            file.mkdirs();
        }

        //说明上传文件项
        //获取上传文件的名称
        String filename = upload.getOriginalFilename();
        //把文件名称设置唯一值,uuid
        String uuid = UUID.randomUUID().toString().replace("-", "");
        filename += uuid + "_" +filename;
        //完成文件上传
        upload.transferTo(new File(path,filename));

        return "success";
    }
```

```xml
<!--    配置文件解析器对象-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<!--        配置上传最大值为10M-->
    <property name="maxUploadSize" value="10485760"></property>
</bean>
```

```html
<h3>springmvc文件上传</h3>

<form action="user/fileUpload2" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="upload"><br/>
    <input type="submit" value="上传"/>
</form>
```

### springmvc跨服务器方式的文件上传

```java
/**
     * 跨服务器文件上传
     * @return
     */
@RequestMapping("/fileUpload3")
public String fileUpload3(HttpServletRequest request, MultipartFile upload) throws Exception {
    System.out.println("跨服务器文件上传...");

    //定义上传服务器路径
    String path = "http://localhost:9090/fileuploadserver_war_exploded/uploads/";

    //说明上传文件项
    //获取上传文件的名称
    String filename = upload.getOriginalFilename();
    //把文件名称设置唯一值,uuid
    String uuid = UUID.randomUUID().toString().replace("-", "");
    filename += uuid + "_" +filename;
    //完成文件上传，跨服务器上传

    //创建客户端的对象
    Client client = Client.create();

    //和图片服务器进行连接
    WebResource webResource = client.resource(path + filename);

    //上传文件
    webResource.put(upload.getBytes());

    return "success";
}
```

```html
<h3>跨服务器文件上传</h3>

<form action="user/fileUpload3" method="post" enctype="multipart/form-data">
    选择文件：<input type="file" name="upload"><br/>
    <input type="submit" value="上传"/>
</form>
```

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>cn.itcast</groupId>
  <artifactId>fileuploadserver</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>fileuploadserver Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <finalName>fileuploadserver</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

**web.xml**

```xml
<servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
        <param-name>debug</param-name>
        <param-value>0</param-value>
    </init-param>
    
    
    
    <init-param>
        <param-name>readonly</param-name>
        <param-value>false</param-value>
    </init-param>
    
    
    
    <init-param>
        <param-name>listings</param-name>
        <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

## springmvc的异常处理

1.  异常处理思路
    controller 调用service，service调用dao，异常哦都是向上抛出的，最终由DispatcherServlet找异常处理器进行异常处理。
2.  springmvc的异常处理

### 处理具体步骤

1.  编写自定义异常类（做提示信息的）

2.  编写异常处理器

3.  配置异常处理器（跳转到提示页面）

    **error.jsp**

    ```jsp
    <%--
      Created by IntelliJ IDEA.
      User: 喻浩
      Date: 2019/8/25
      Time: 9:42
      To change this template use File | Settings | File Templates.
    --%>
    <%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
        ${errorMsg}
    </body>
    </html>
    
    ```

    **springmvc.xml**

    ```xml
    <!--    配置异常处理器-->
        <bean id="sysExceptionResolver" class="cn.itcast.exception.SysExceptionResolver"></bean>
    ```

    **SysException.java**

    ```java
    package cn.itcast.exception;
    
    /**
     * @author 喻浩
     * @create 2019-08-25-9:34
     */
    public class SysException extends Exception {
        
        //存储提示信息的
        private String message;
    
        @Override
        public String getMessage() {
            return message;
        }
    
        public void setMessage(String message) {
            this.message = message;
        }
    
        public SysException(String message) {
            this.message = message;
        }
    }
    ```

    **SysExceptionResolver.java**

    ```java
    package cn.itcast.exception;
    
    import org.springframework.web.servlet.HandlerExceptionResolver;
    import org.springframework.web.servlet.ModelAndView;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    
    /**
     * 异常处理器
     * @author 喻浩
     * @create 2019-08-25-9:38
     */
    public class SysExceptionResolver implements HandlerExceptionResolver {
    
        /**
         * 处理异常业务逻辑
         * @param request
         * @param response
         * @param handler
         * @param ex
         * @return
         */
        @Override
        public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
            //获取到异常对象
            SysException e = null;
    
            if (ex instanceof SysException) {
                e= (SysException)ex;
            }else {
                e = new SysException("系统正在维护....");
            }
            
            //创建 ModelAndView 对象
            ModelAndView mv = new ModelAndView();
            mv.addObject("errorMsg", e.getMessage());
            mv.setViewName("error");
            return mv;
        }
    }
    
    ```

    **UserController.java**

    ```java
    package cn.itcast.controller;
    
    import cn.itcast.exception.SysException;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    /**
     * @author 喻浩
     * @create 2019-08-25-9:27
     */
    @Controller
    @RequestMapping("/user")
    public class UserController {
        
        @RequestMapping("/testException")
        public String testException() throws SysException{
            System.out.println("testException执行了。。。");
    
            try {
                //模拟异常
                int i = 1/0;
            } catch (Exception e) {
                //打印异常信息
                e.printStackTrace();
                //抛出自定义异常信息
                throw new SysException("查询所有用户出现错误了");
            }
    
            return "success";
        }
    }
    
    ```

    **index.jsp**

    ```jsp
    <%--
      Created by IntelliJ IDEA.
      User: 喻浩
      Date: 2019/8/25
      Time: 9:26
      To change this template use File | Settings | File Templates.
    --%>
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
        <h3>异常处理</h3>
    
        <a href="user/testException">异常处理</a>
    </body>
    </html>
    
    ```

## springmvc中的拦截器

SpringMVC 的处理器拦截器类似于 Servlet 开发中的过滤器 Filter，用于对处理器进行预处理和后处理。
用户可以自定义一些拦截器来实现特定的功能。
谈到拦截器，还要向大家提一个词-拦截器链。拦截器链就是将拦截器按一定的顺序联结成一条链。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。
两者的区别：

​					过滤器是 servlet 规范中的一部分，任何java web 工程都可以使用。
​					拦截器是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用。
​					过滤器在 url-pattern 中配置了 /* 之后，可以对所有要访问的资源拦截。
​					拦截器它是指挥拦截访问的控制器方法，如果访问的是 jsp,html,css,image 或者 js 是不会进行拦截的。
​					它也是 AOP 思想的具体应用。
​					我们想自定义拦截器，要求必须实现： HandlerInterceptor接口

### 拦截器入门

#### 1.  编写拦截器类，配置HandlerInterceptor接口

**MyInterceptor1.java**

```java
package cn.itcast.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 喻浩
 * @create 2019-08-25-11:02
 */
public class MyInterceptor1 implements HandlerInterceptor {

    /**
     * 预处理，controller方法执行前
     * return true 放行，执行下一个拦截器，如果没有，执行controller中的方法
     * return false 不放行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1执行了...");
        return true;
    }
}

```

#### 2. 配置拦截器

**springmvc.xml**

```xml
<!--    配置拦截器-->
    <mvc:interceptors>
<!--        配置拦截器-->
        <mvc:interceptor>
<!--            要拦截的具体的方法-->
            <mvc:mapping path="/user/*"/>
<!--            不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
<!--            配置拦截器对象-->
            <bean class="cn.itcast.interceptor.MyInterceptor1" />
        </mvc:interceptor>
    </mvc:interceptors>
```

#### 3. HandlerInterceptor接口中的方法

1.  preHandle方法是controller方法执行前拦截的方法
    1.  可以使用request或者response跳转到指定的页面
    2.  return true放行，执行下一个拦截器，如果没有拦截器，执行controller中的方法。
    3.  return false不放行，不会执行controller中的方法
2.  postHandle是controller方法执行后执行的方法，在JSP视图执行前
    1.  可以使用request或者response跳转到指定的页面
    2.  如果指定了跳转的页面，那么controller放跳转的页面将不会显示。
3.  afterCompletion方法是在JSP执行后执行
    1.  request或者response不能再跳转页面了

#### 4. 配置多个拦截器

**springmvc.xml**

```xml
<!--    配置拦截器-->
    <mvc:interceptors>
<!--        配置拦截器-->
        <mvc:interceptor>
<!--            要拦截的具体的方法-->
            <mvc:mapping path="/user/*"/>
<!--            不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
<!--            配置拦截器对象-->
            <bean class="cn.itcast.interceptor.MyInterceptor1" />
        </mvc:interceptor>
        
        <!--        配置第二个拦截器-->
        <mvc:interceptor>
            <!--            要拦截的具体的方法-->
            <mvc:mapping path="/**"/>
            <!--            不要拦截的方法
            <mvc:exclude-mapping path=""/>
            -->
            <!--            配置拦截器对象-->
            <bean class="cn.itcast.interceptor.MyInterceptor2" />
        </mvc:interceptor>

    </mvc:interceptors>
```

**MyInterceptor1.java**

```java
package cn.itcast.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 喻浩
 * @create 2019-08-25-11:02
 */
public class MyInterceptor1 implements HandlerInterceptor {

    /**
     * 预处理，controller方法执行前
     * return true 放行，执行下一个拦截器，如果没有，执行controller中的方法
     * return false 不放行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1执行了...前11111");
//        request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request, response);
        return true;
    }

    /**
     * 后处理方法,controller方法执行后，success.jsp执行之前
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor1执行了...后111111");
//        request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request, response);
    }

    /**
     * success.jsp页面执行后，该方法会执行
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor1执行了...最后11111");
    }
}

```

**MyInterceptor2.java**

```java
package cn.itcast.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author 喻浩
 * @create 2019-08-25-11:02
 */
public class MyInterceptor2 implements HandlerInterceptor {

    /**
     * 预处理，controller方法执行前
     * return true 放行，执行下一个拦截器，如果没有，执行controller中的方法
     * return false 不放行
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("MyInterceptor1执行了...前22222");
//        request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request, response);
        return true;
    }

    /**
     * 后处理方法,controller方法执行后，success.jsp执行之前
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("MyInterceptor1执行了...后2222");
//        request.getRequestDispatcher("/WEB-INF/pages/error.jsp").forward(request, response);
    }

    /**
     * success.jsp页面执行后，该方法会执行
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("MyInterceptor1执行了...最后2222");
    }
}
```

**打印输出：**

```
MyInterceptor1执行了...前11111
MyInterceptor1执行了...前22222
testInterceptor执行了...
MyInterceptor1执行了...后2222
MyInterceptor1执行了...后111111
success.jsp执行了...
MyInterceptor1执行了...最后2222
MyInterceptor1执行了...最后11111
```

![springmvc拦截器运行过程](D:\学习\文章\images\Mybatis\springmvc拦截器运行过程.bmp)

