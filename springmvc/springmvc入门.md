

# 入门案例流程总结

1.  启动服务器，加载一些配置文件

    -   DispatcherServlet对象创建
    -   springmvc.xml被加载了
    -   HelloController创建成对象

2.  发送请求，后台处理请求
    **springmvc框架基于组件方式执行流程：**

    ![springmvc框架基于组件方式执行流程](D:\学习\文章\images\Mybatis\springmvc框架基于组件方式执行流程.png)

## 入门案例中涉及的组件

1.  DispatcherServlet：前端控制器
    用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。
    
2.  HandlerMapping：处理器映射器
    HandlerMapping负责根据用户请求找到Handler即处理器，SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。
    
3.  Handler：处理器
    它就是我们开发中要编写的具体业务控制器。由DispatcherServlet把用户请求转发到Handler。由Handler对具体的用户请求进行处理。
    
4.  HandlerAdapter：处理器适配器
    通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
    
5.  ViewResolver：视图解析器
    View Resolver负责将处理结果生成View视图，View Resolver 首先根据逻辑视图名解析成物理视图名，即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。

6.  View：视图

    SpringMVC 框架提供了很多的View视图类型的支持，包括jstlView、freemarkView、pdfView等。我们最常用的视图就是jsp
    一般清空下需要通过页面标签或页面模板技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

7.  \<mvc:annotation-driven\>说明：
    再SpringMVC的各个组件中，处理器映射器、处理器是配置、视图解析器称为SpringMVC三大组件。
    使用\<mvc:annotation-driven\>自动加载 RequestMappingHandlerMapping（处理映射器）和RequestMappingHandlerAdapter（处理器适配器），可用在SpringMVC.xml配置文件中使用\<mvc:annotation-driven\>替代注解处理器和适配器的配置。

## RequestMapping注解

**作用：**

-   用于建立请求URL和处理请求方法之间的对应关系

**使用：**

-   RequestMapping可以放在类和方法上，放在类上后，类中的方法配置的参数，在网站中访问时，全都需要加上类上RequestMapping的配置，即可以分为模块开发

```java
@Controller
@RequestMapping(path = "/user")
public class HelloController {
    
    @RequestMapping(path = "/hello")
    public String sayHello(){
        System.out.println("hello springmvc");
        return "success";
    }

    /**
     * RequestMapping注解
     * @return
     */
    @RequestMapping(path = "/testRequestMapping")
    public String testRequestMapping(){
        System.out.println("测试RequestMapping注解。。。");
        return "success";
    }
}
```

​	访问sayHello()：/user/hello

​	访问testRequestMapping()：/user/testRequestMapping

**属性：**

-   value：用于指定请求的URL。它和path属性的作用是一样的。
-   method：用于指定请求的方式。
-   params：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的key和value必须和配置的一模一样
    例如：
    params  = {"accountName"}，表示请求参数必须有accountName
    params = {"money!100"}，表示请求参数中money不能是100
-   headers：用于指定限制请求消息头的条件。
    **注意：**
              以上四个属性只要出现2个或以上时，他们的关系是与的关系。



## 请求参数的绑定

1.  请求参数的绑定说明
    1.  绑定机制
        1.  表单提交的数据都是k=v格式的 username=haha&password=123
        2.  SpringMVC的参数绑定过程是把表单提交的请求参数，作为控制器中方法的参数进行绑定的
        3.  要求：提交表单的name和参数名称是相同的
    2.  支持的数据类型
        1.  基本数据类型和字符串类型
        2.  实体类型（javaBean）
        3.  集合数据类型（List、map集合等）
2.  基本数据类型和字符串类型
    1.  提交表单的name和参数的名称是相同的
    2.  区分大小写
3.  实体类型（JavaBean）
    1.  提交表单的name和JavaBean中的属性名称需要一直
    2.  如果一个JavaBean类中包含其他的引用类型，那么表单的name属性需要编写成：对象.属性
        例如：address.name
4.  给集合属性数据封装
    1.  JSP页面编写方式：list[0].属性
5.  请求参数中文乱码的解决
    1.  在web.xml中配置Spring提供的过滤器类

```xml
<!--  配置解决中文乱码的过滤器-->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

### 特殊情况

1.  自定义类型转换器

    1.  表单提交的任何数据类型都是字符串类型，但是后台定义Integer类型，数据也可以封装上，说明SpringMVC框架内部会默认进行数据转换。

    2.  如果想自定义数据类型转换，可以实现Converter的接口

        1.  自定义类型转换器

            ```java
            package cn.itcast.utils;
            
            import org.springframework.core.convert.converter.Converter;
            
            import java.text.DateFormat;
            import java.text.ParseException;
            import java.text.SimpleDateFormat;
            import java.util.Date;
            
            /**
             * 把字符串转换日期
             * @author 喻浩
             * @create 2019-08-21-23:11
             */
            public class StringToDateConverter implements Converter<String, Date> {
            
                /**
                 * String source     传入进来字符串
                 * @param source
                 * @return
                 */
                @Override
                public Date convert(String source) {
                    //判断
                    if(source == null){
                        throw new RuntimeException("请您传入数据");
                    }
                    DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
                    try {
                        //把字符串转换日期
                        return df.parse(source);
                    } catch (ParseException e) {
                        throw new RuntimeException("数据类型转换出现错误");
                    }
                }
            }
            
            ```

            ```xml
            <!--    配置自定义类型转换器-->
                <bean id="conversionServiceFactoryBean" class="org.springframework.context.support.ConversionServiceFactoryBean">
                    <property name="converters">
                        <set>
                            <bean class="cn.itcast.utils.StringToDateConverter"/>
                        </set>
                    </property>
                </bean>
                
            <!--    开启SpringMVC框架注解的支持-->
                <mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
            ```

2.  在控制器中使用原生的ServletAPI对象

    1.  只需要在控制器的方法参数定义HttpServletRequest和HttpServletResponse对象

        ```java
        /**
        * 原生的API
        * @return
        */
        @RequestMapping("/testServlet")
        public String testServlet(HttpServletRequest request, HttpServletResponse response){
            System.out.println("执行了。。。");
            System.out.println(request);
        
            HttpSession session = request.getSession();
            System.out.println(session);
        
            ServletContext servletContext = session.getServletContext();
            System.out.println(servletContext);
        
            System.out.println(response);
            return "success";
        }
        ```

        

## 常用注解

-   使用说明：
    作用：把请求中指定名称的参数给控制器中的形参赋值
    属性：
    		value：请求参数中的名称
    		required：请求参数中是否必须提供此参数。默认值：true。表示必须提供，如果不提供将报错
-   ModelAttribute
    -   作用：
        该注解是SpringMVC4.3版本以后新加入的。它可以用于修饰方法和参数。
        出现在方法上，表示当前方法会在控制器的方法执行之前，先执行。它可以修饰没有返回值的方法，也可以修饰有具体返回值的方法
        出现在参数上，获取指定的数据给参数赋值
    -   属性：
        value：用于获取数据的key。key可以是POJO的属性名称，也可以是map结构的key。
    -   应用场景：
        当表单提交数据不是完整的实体类数据时，保证没有提交数据的字段使用数据库对象原来的数据。
        例如：
        -   我们在编辑一个用户时，用户有一个创建信息的字段，该字段的值时不允许被修改的。提交表单的数据肯定是没有此字段内容的，一旦更新会把该字段内容设置为nul，此时就可以使用此注解解决问题。