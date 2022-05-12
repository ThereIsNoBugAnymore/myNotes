[TOC]

# 一. SpringBoot安装及配置

## 1. Hello SpringBoot

使用Maven方式构建SpringBoot项目

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.ysu</groupId>
    <artifactId>SpringBoot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>{Sprintboot-version}</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

**SpringBoot Hello World Demo**

+ 需求：在浏览器中访问http://localhost:8080/hello获取一串字符

+ 步骤：

  1. 创建工程

  2. 添加依赖（启动器依赖，spring-boot-starter-web）（如上所示）

  3. 创建启动类（启动类表示项目的启动入口）

     ```java
     package com.ysu;
     
     import org.springframework.boot.SpringApplication;
     import org.springframework.boot.autoconfigure.SpringBootApplication;
     
     /**
      * spring boot都有一个启动引导类，这是工程的入口类
      * 在引导类上添加@SrpingBootApplication注解
      * 使用该注解会自动扫描当前包下的所有使用Spring注解的包或者类
      * 因此注意要将其他包或者类放在同一个包下
      */
     @SpringBootApplication
     public class Application {
         public static void main(String[] args) {
             SpringApplication.run(Application.class, args);
         }
     }
     ```
     
     
     
4. 创建处理器Controller
  
   ```java
     package com.ysu.controller;
     
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;
     
     /**
      * @ RestController是一个组合注解，组合了@Controller和@ResponseBody
      * 该注解表示返回的结果都将当做一个字符内容来处理（ResponseBody的效用）
      */
     @RestController
     public class HelloController {
         /**
          * 映射路径方法
          * 1️⃣使用注解@RequestMapping，需要指定限定方法
          * 2️⃣使用注解@GetMapping/PostMapping，即已指定为get方法方法
          * @return
          */
         @GetMapping("hello")
         public String hello() {
             return "Hello SpringBoot!";
         }
     }
   ```
  
     
  
  5. 测试
  
     运行启动类即启动服务

在 pom.xml 中添加如下插件，即可将项目打成jar包直接执行，而无需达成war包后在tomcat部署

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

## 2. 自动配置原理

### 2.1 SpringBoot 特点

- 依赖管理

    > ```xml
    > <parent>
    >     <groupId>org.springframework.boot</groupId>
    >     <artifactId>spring-boot-starter-parent</artifactId>
    >     <version>{SpringBoot-version}</version>
    > </parent>
    > ```
    >
    > 它的父项目为
    >
    > ```xml
    > <parent>
    >     <groupId>org.springframework.boot</groupId>
    >     <artifactId>spring-boot-dependencies</artifactId>
    >     <version>{SpringBoot-version}</version>
    > </parent>
    > ```
    >
    > 其中几乎声明了所有开发中常用的依赖的版本号；开发者无需关注版本号，自动版本仲裁；
    >
    > 如需修改或使用特定版本的依赖，可以先查看spirng-boot-dependencies中规定的当前依赖ban的版本使用的key，再在本项目中重写配置。如在spring-boot-dependencies-2.6.4中，默认mysql版本为`<mysql.version>8.0.28</mysql.version>`，则可以在pom中添加节点
    >
    > ```xml
    > <properties>
    >     <mysql.version>5.1.2</mysql.version>
    > </properties>
    > ```
    >
    > 将数据库版本指定为5.1.2版本

- starter 场景启动器

    > - 在引入依赖时，存在许多 spring-boot-starter-* 的依赖，*即指某种场景，如sprint-boot-starter-web则是指web开发场景
    > - 只要引入starter，这个场景中的所有常规依赖都会被自动引入
    > - SpringBoot支持的场景可点击链接查看https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters
    > - \*-starter-\*：一般是第三方开发的简化启动器

- 无需关注版本号，自动版本仲裁

    > - 引入依赖时都可以不写版本号
    > - 引入非仲裁的jar包时要手动添加版本号

### 2.2 自动配置

- 自动配置 Tomcat

- 自动配置 SpringMVC

- 自动配置 Web 常用功能，如字符编码问题

- 默认的包结构

    主程序所在包及其下属所有子包均会被扫描，如需改变扫描路径，可以使用在注解中使用`@SpringBootApplication(scanBasePackages="package")`或者`@ComponmentScan（"package"）`指定扫描路径

- 各种配置拥有默认值

    默认配置最终都是映射到 MultipartProperties，配置文件的值最终会绑定在每个类上，这个类会在容器中创建对象

- 按需加载所有自动配置项

    引入哪些场景，哪些场景的自动配置才会开启，SpringBoot 所有的自动配置功能都在 spring-boot-autoconfigure 包里面

# 二. 属性配置方式

## 1. 容器功能

### 1.1 组件添加

- `@Bean`：声明在方法上，将该方法的返回值作为一个Bean容器，代替`<beans>`标签

- `@Component`

- `@ComponentScan`

- `@Import`：在容器中自动创建组件，默认组件的名字就是全类名

- `@Conditional`：条件装配，满足 Conditionnal 指定的条件则进行组件注入，有诸多派生注解可供使用

    ![@Conditional继承树](D:\workspace\myNotes\assets\@Conditional继承树.png)

    |   @Conditional 常用扩展注解   |                       作用                       |
    | :---------------------------: | :----------------------------------------------: |
    |      @ConditionalOnBean       |           容器中存在指定Bean时添加组件           |
    |   @ConditionalOnMissingBean   |         容器中不存在指定Bean 时添加组件          |
    |    @ConditionalOnProperty     |          系统中指定的属性是否有指定的值          |
    |    @ConditionalOnResource     |           类路径下受否存在指定资源文件           |
    | @ConditionalOnSingleCandidata | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
    |      @ConditionalOnClass      |               系统中有指定类时添加               |
    | @ConditionalOnWebApplication  |                  当前是web环境                   |

- `@Configuration`：声明一个类作为配置类，代替XML文件

    > 配置类中使用`@Bean`标注在方法上给容器注册组件，默认是单实例的；配置类也是组件
    >
    > 配置类又两种模式：
    >
    > - Full模式：`@Configuration(proxyBeanMethod = true)`，默认为该模式
    >
    >     Full模式下，外部无论对配置类中的组件注册方法调用多少次，获取的都是之前注册容器的单实例对象
    >
    >     该模式下，每一次调用SpringBoot都会去容器中检查bean对象是否存在，因此执行较为缓慢
    >
    > - Lite模式：`@Configuration(proxyBeanMethod = false)`
    >
    >     该模式下，SpringBoot不会去容器中检查bean对象是否在容器中存在，直接跳过检查创建新对象，因此SpringBoot执行速度会加快
    >
    > **如果只是单纯注册组件，其他地方也不会依赖该组件，一般建议设置为Lite模式**

### 1.2 原生配置文件引入

- `@ImportResource`：原生配置文件导入

    `@ImportResource("classpath:beans.xml")`

### 1.3 配置绑定

- @ConfigurationProperties：配置绑定，将配置文件中的数据绑定到JavaBean

    `@EnableConfigurationProperties``@ConfigurationProperties`联合使用，或`Component``@ConfigurationProperties`联合使用

## 2. 传统配置方式

demo

+ 需求：使用代码配置数据库连接池，并在处理器中注入使用

+ 步骤：

  1. 添加依赖

     以使用Druid连接池为例

     ```xml
     <dependency>
         <groupId>com.alibaba</groupId>
         <artifactId>druid</artifactId>
         <version>1.1.21</version>
     </dependency>
     ```

     

  2. 创建数据库

  3. 创建数据库连接参数的配置文件

     在resources资源文件夹下创建配置文件jdbc.properties

     ```text
     jdbc.driverClassName = com.mysql.cj.jdbc.Driver	# 数据库连接驱动程序
     jdbc.url = jdbc:mysql://localhost:3306/keep
     jdbc.username = root	# 登录账户
     jdbc.password = 123456	# 登录密码
     ```

     

  4. 创建配置类

     ```java
     package com.ysu.config;
     
     import com.alibaba.druid.pool.DruidDataSource;
     import org.springframework.beans.factory.annotation.Value;
     import org.springframework.context.annotation.Bean;
     import org.springframework.context.annotation.Configuration;
     import org.springframework.context.annotation.PropertySource;
     
     import javax.sql.DataSource;
     
     /**
      * @Configuration表明其是一个配置类
      * @PropertySource表明配置文件路径
      */
     @Configuration
     @PropertySource("classpath:jdbc.properties")
     public class JdbcConfig {
     
         /**
          * @Value表示读取配置文件中的数据
          */
         @Value("${jdbc.driverClassName}")
         String driverClassName;
     
         @Value("${jdbc.url}")
         String url;
     
         @Value("${jdbc.username}")
         String username;
     
         @Value("${jdbc.password}")
         String password;
     
         /**
          * @Bean注册Bean对象
          */
         @Bean
         public DataSource dataSource() {
             DruidDataSource dataSource = new DruidDataSource();
             dataSource.setDriverClassName(driverClassName);
             dataSource.setUrl(url);
             dataSource.setUsername(username);
             dataSource.setPassword(password);
             return dataSource;
         }
     }
     ```

     

  5. 改造处理器类注入数据源并使用

     ```java
     package com.ysu.controller;
     
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.web.bind.annotation.GetMapping;
     import org.springframework.web.bind.annotation.RestController;
     
     import javax.sql.DataSource;
     
     @RestController
     public class HelloController {
     
         /**
          * @Autowired 通过匹配数据类型自动装配Bean
          */
         @Autowired
         private DataSource dataSource;
     
         @GetMapping("hello")
         public String hello() {
             System.out.println("DataSource = " + dataSource);
             return "Hello SpringBoot!";
         }
     }
     ```



## 3. Spring Boot属性注入方式

注解`@Value`不能将配置项读取到对象中，使用注解`@ConfiguratioinProperties`可以将Spring Boot配置文件（默认必须为`application.properties`或者`application.yml`）配置项读取到一个对象中。

**demo**

+ 需求：将配置文件中的配置项读取到一个对象中

+ 步骤：

  1. 创建配置项类JdbcProperties类，在该类名上添加注解`@ConfigurationProperties`

     ```java
     package com.ysu.config;
     
     import org.springframework.boot.context.properties.ConfigurationProperties;
     
     /**
      * 注解@ConfigurationProperties 从application配置文件中读取配置项
      * prefix 表示配置项的前缀
      * 配置类/配置项中的变量名必须要于前缀之后的配置项名称保持松散绑定（相同）
      */
     @ConfigurationProperties(prefix = "jdbc")
     public class JdbcProperties {
         private String url;
         private String driverClassName;
         private String username;
         private String password;
     
         public String getUrl() {
             return url;
         }
     
         public void setUrl(String url) {
             this.url = url;
         }
     
         public String getDriverClassName() {
             return driverClassName;
         }
     
         public void setDriverClassName(String driverClassName) {
             this.driverClassName = driverClassName;
         }
     
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
     }
     ```
     
     
     
2. 创建配置文件`application.properties`（与上节jdbc.properties内容相同）
  
3. 将JdbcProperties对象注入到JdbcConfig
  
   ```java
     package com.ysu.config;
     
     import com.alibaba.druid.pool.DruidDataSource;
     import org.springframework.boot.context.properties.EnableConfigurationProperties;
     import org.springframework.context.annotation.Bean;
     import org.springframework.context.annotation.Configuration;
     
     import javax.sql.DataSource;
     
     @Configuration
     @EnableConfigurationProperties(JdbcProperties.class)
     public class JdbcConfig {
     
         @Bean
         public DataSource dataSource(JdbcProperties jdbcProperties) {
             DruidDataSource dataSource = new DruidDataSource();
             dataSource.setDriverClassName(jdbcProperties.getDriverClassName());
             dataSource.setUrl(jdbcProperties.getUrl());
             dataSource.setUsername(jdbcProperties.getUsername());
             dataSource.setPassword(jdbcProperties.getPassword());
             return dataSource;
         }
     }
   ```
  
   

`@value`与`@ConfigurationProperties`对比

`@ConfigurationProperties`的优势在于：Relaxed blinding：松散绑定。

+ 不严格要求属性文件中的属性名和成员变量名保持一致。支持驼峰，中划线，下划线等等转换，甚至支持转换对象引导。比如user.friend.name：代表的是user对象中的friend属性中的name属性，显然friend也是对象。`@Value`无法完成这样的注入方式
+ meta-data support：元数据支持，帮助IDE生成属性提示

## 4. 更优雅的注入方式

如果一段属性只有一个`Bean`需要使用，无需将其注入到一个类中，而是直接在需要的地方声明即可，如上例中，可将`JdbcConfig`类修改为如下：

```java
package com.ysu.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class JdbcConfig {

    @Bean
//  声明要注入的属性前缀，Spring Boot会自动把相关属性通过set方法注入到DataSource中
    @ConfigurationProperties(prefix = "jdbc")
    public DataSource dataSource() {
        return new DruidDataSource();
    }
}
```

直接将`@ConfigurationProperties`的声明使用在需要使用的`@Bean`的方法上，Spring Boot就会自动调用这个`Bean`（在上述例子中为`DataSource`）的`set`方法然后完成注入。使用的前提是：**该类必须要有对应属性的`set`方法**。

**小结**

1. 使用@ConfigurationProperties编写配置项类将配置文件中的配置项设置到对象中
2. 使用@ConfigurationProperties在方法上面使用

## 5. Yaml配置文件

+ 单一配置文件

配置文件除了可以使用application.properties类型，还可以使用.yml或者.yaml的类型，也就是application.yml或者application.yaml

> YAML是一种间接的非标记语言。YAML以数据为中心，使用空白、缩进、分行组织数据。

yaml与properties配置文件除了展示形式不同以外，其他功能和作用完全相同，因此原来的读取方式不需要改变

```yaml
jdbc:
  url: jdbc:mysql://localhost:3306/keep
  driverClassname: com.mysql.cj.jdbc.Driver
  username: root
  password: 123456
  
# 注释
test:
  string: 字符串无须使用引号
  array:
    - 我
    - 是
    - 列
    - 表
```

+ 多个配置文件

  Spring Boot中允许使用多个配置文件，但配置文件名称必须命名为application-*.yml，并且这些配置文件必须要在application.yml配置文件中激活才可以使用

  如现有application-jdbc.yml、application-c3p0.yml配置文件，则需在application.yml中完成以下配置才可激活使用

  ```yaml
  spring:
    profiles:
      active: jdbc,c3p0
  ```

  

+ properties与yml共存

  允许properties与yml配置文件同时出现在项目中，且两个配置文件均有效。如果在两个配置文件中存在同名的配置项，则会以properties的文件为主。

## 6. 占位符

+ 占位符语法

  语法：`${}`

+ 占位符作用

  1) `${}`中可以获取框架提供的方法中的值，比如`random.int`等

  2) 占位符可以获取配置文件中的键的值赋给另一个键作为值

+ 生成随机数

  **`${random.value}`**-类似uuid的随机数，没有“-”连接

  **`${random.int}`**-随机取整形范围内的一个值

  **`${random.long}`**-随机取长整型范围内的一个值

  **`${random.long(100, 200)}`**-随机生成长整型100-200范围内的一个值

  **`${random.uuid}`**-生成一个uuid，存在“-”连接

# 三. 启动器

Spring Boot将所有的功能场景都抽取出来，做成一个个的starter（启动器），只需要在项目中引入这些starter相关场景的所有依赖都会导入进来，要有什么功能就导入什么场景，在jar包管理上非常方便，最终实现一站式开发。

Spring Boot提供多达44种启动器，比如：

**Spring-boot-starter**

Spring Boot的核心启动器，包含自动配置、日志和yaml。

**Spring-boot-starter-actuator**

帮助监控和管理应用

**Spring-boot-starter-web**

支持全栈式Web开发，包含Tomcat和spring-webmvc

# 四. SpringBoot核心注解

+ `@SpringBootApplication`

    是SpringBoot的启动类。

    此注解等同于`@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`的组合

+ `@SpringBootConfiguration`

    `@SpringBootConfiguration`注解是`@Configuration`注解的派生注解，跟`@Configuration`注解的功能一致，标注这个类是一个配置类，只不过`@SrpingBootConfiguration`是SpringBoot的注解，而`@Configuration`是Spring的注解

+ `@Configuration`

    通过`bean`对象的操作替代spring中xml文件

+ `@EnableAutoConfiguration`

    SpringBoot自动配置（auto-configuration）：尝试根据添加的jar依赖自动配置Spring应用。是`@AutoConfigurationPackage`和`@Import(AutoConfigurationImportSelector.class)`注解的组合。

+ `@AutoConfigurationPackage`

    `@AutoConfigurationPackage`注解，自动注入主类下所在包下所有的加了注解（`@Controller`，`@Service`等）的类以及配置类（`@Configuration`）

+ `@Import({AutoConfigurationImportSelector.class})`

    导入普通的类

    导入实现了`ImportSelector`接口的类

    导入实现了`ImportBeanDefinitionRegistrar`接口的类

+  `@ComponentScan`

    组件扫描，可自动发现和装配一些Bean

+ `@ConfigurationPropertiesScan`

    `@ConfigurationPropertiesScan`扫描配置属性。`@EnableConfigurationProperties`注解的作用使用`@ConfigurationProperties`注解的类生效

# 五. SpringBoot在Controller中的常用注解

+  `@RestController`

    `@RetController`相当于`@Controller`+`@ResponseBody`注解

    如果使用`@RestController`注解Controller中的方法无法返回页面，相当于在方法上面自动加了`@ResponseBody`注解，所以没办法跳转并传输数据到另一个页面，所以InteralResourceViewResolver也不起作用，返回的内容就是`return`里的内容。

+ `@GetMappting`

    `@GetMapping`注解是`@RequestMapping(method=RequestMethod.GET)`的缩写

+ `@PostMappting`

    `@PostMappting`注解是`@RequestMapping(method=RequestMethod.POST)`的缩写

+ `@PutMappting`

    `@PutMappting`注解是`@RequestMapping(method=RequestMethod.PUT)`的缩写

+ `@DeleteMappting`

    `@DeleteMappting`注解是`@RequestMapping(method=RequestMethod.DELETE)`的缩写

**请求处理，常用参数注解**

- @PathVariable-路径变量

    ```java
    @GetMapping("/test/{params1}/path/{params2}")
    public Object test(@PathVariable("params1") Integer params1, @PathVariable("params2") Integer params2) {
        ......
    }
    ```

- @RequestHeader-获取请求头

    ```java
    @GetMapping("/test")
    public Object test(@RequestHeader('User-Agent') String userAgent, @RequestHeader Map<String, String> headers)) {
        ......
    }
    ```

    `@RequestHeader('User-Agent')`获取请求头UA信息，`@RequestHeader`不指定键名获取全部请求头的Map

- @RequestParam-获取请求参数

    ```java
    @GetMapping("/test")
    public Object test(@RequestParam("param") String param, @RequestParam("list") List list) {
        ......
    }
    ```

    分别获取名为param与list的参数

- @CookieValue-获取cookie值

    ```java
    @GetMapping("/test")
    public Object test(@CookieValue("_ga") String _ga, @CookieValue("_ga") Cookie _ga) {
        ......
    }
    ```

    获取cookie中_ga的值，分别为String与Cookie类

- @RequestBody-获取请求体（POST方法提交的表单）

    ```java
    @PostMapping("/test")
    public Object test(@RequestBody String content) {
        ......
    }
    ```

    将获取内容以字符串形式获得

- @RequestAttribute-获取request域属性

    可以用来处理页面转发时页面中的属性

    ```java
    @Controller
    public class test {
        @GetMapping("goToPage")
        public String goToPage(HttpServerlet request) {
            request.setAttribute("code", 200);
            request.setAttribute("msg", "自定义信息");
            return "forward:/page";
        }
        
        @ResponseBody
        @GetMapping("page")
        public String page(@RequestAttribute("code") Integer code,
                          @RequestAttrubte("msg") String msg,
                          HttpServletRequest request) {
            String msg1 = request.getAttribute("msg");
        }
    }
    ```

    访问`goToPage`会转发至`page`页，并向request域中添加两个属性值：code与msg。可以通过`@RequestAttribute`注解获取request域中的属性，或者通过`HttpServletRequest`对象来获取

- @MatrixVariable-矩阵变量

    什么是矩阵变量？

    ```http
    url?a=1&b=2&c=3
    ```

    如上传递参数的方式称之为queryString，即查询字符串，在SpringBoot中可以通过@RequestParam获取相关参数

    ```http
    url;a=1;b=2;c=3
    ```

    此类使用分号分割参数传递的方式称之为矩阵变量

    ```java
    // test/route;param1=1;param2=a,b,c
    @GetMapping("test/{path}")
    public Map test(@MatrixVariable("parma1") Integer param1,
                    @MatrixVariable("param2") List param2,
                   @PathVariable("path") String path) {
        ......
    }
    ```

    如上则可获取相关参数：param1，param2以及路径path值

    ```java
    // test/route1;param=1/route2;param=2
    @GetMappgin("test/{path1}/{path2}")
    public Map test(@MatrixVariable(value = "param", pathVar = "route1") Integer param1,
                   @MatrixVariable(value = "param", pathVar = "route2") Integer param2) {
        ......
    }
    ```

    如上即可指定路径获取多个不同参数

    但SpringBoot默认是关闭MatrixVariable矩阵变量的，需要自己手动设置开启

# 六. SpringBoot整合Servlet

## 1. Servlet整合

+ 通过注解扫描完成Servlet组件的注册

    + 创建`Servlet`

        添加`@WebServlet(name="", urlPatterns="")`注解，`name`属性就相当于servlet配置文件中的name属性，`urlPatterns`则代表访问路径

        ```java
        package com.ysu.servlet;
        
        import javax.servlet.ServletException;
        import javax.servlet.annotation.WebServlet;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        /**
         * 通过注解方式配置Servlet
         */
        @WebServlet(name = "firstServlet", urlPatterns = "/firstServlet")
        public class FirstServlet extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                System.out.println("Hello FristServlet!");
            }
        
            @Override
            protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                super.doPost(req, resp);
            }
        }
        ```

        

    + 修改启动类

        为启动类添加`@ServletComponentScan`注解，Spring Boot启动时会扫描@WebServlet注解，并将该类实例化

        ```
        package com.ysu;
        
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.boot.web.servlet.ServletComponentScan;
        
        @SpringBootApplication
        @ServletComponentScan
        public class Application() {
            public static void main(String[] args) {
                SpringApplication.run(Application.class, args);
            }
        }
        ```

        

+ 通过方法完成Servlet组件的注册

    + 创建`Servlet`

        ```java
        package com.ysu.servlet;
        
        import javax.servlet.ServletException;
        import javax.servlet.http.HttpServlet;
        import javax.servlet.http.HttpServletRequest;
        import javax.servlet.http.HttpServletResponse;
        import java.io.IOException;
        
        public class SecondServlet extends HttpServlet {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                System.out.println("Hello SecondServlet");
            }
        }
        ```

        

    + 创建`Servlet`配置类

        ```java
        package com.ysu.config;
        
        import com.ysu.servlet.SecondServlet;
        import org.springframework.boot.web.servlet.ServletRegistrationBean;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        
        @Configuration
        public class ServletConfig {
        
            /**
             * 完成Servlet的注册
             * @return
             */
            @Bean
            public ServletRegistrationBean getServletRegistrationBean() {
                ServletRegistrationBean bean = new ServletRegistrationBean(new SecondServlet());
                bean.addUrlMappings("/secondServlet");
                return bean;
            }
        }
        ```

## 2. Filter整合

Spring Boot整合Filter过程与整合Servlet类似

+ 通过注解扫描完成Filter组件的注册
    + 创建`Filter`

        `urlPatterns`为过滤连接，可以为数组/字符串，如{"/first", "*.jsp"}

        ```java
        package com.ysu.filter;
        
        import javax.servlet.*;
        import javax.servlet.annotation.WebFilter;
        import java.io.IOException;
        
        /**
         * 通过注解方式配置filter
         */
        @WebFilter(filterName = "firstFilter", urlPatterns = "/firstServlet")
        public class FirstFilter implements Filter {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
        
            }
        
            @Override
            public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
                System.out.println("Hello FirstFilter");
                filterChain.doFilter(servletRequest, servletResponse);
                System.out.println("Bye FirstFilter");
            }
        
            @Override
            public void destroy() {
        
            }
        }
        ```

        

    + 修改启动类

        为启动类添加`@ServletComponentScan`注解

+ 通过方法完成Filter组件的注册

    + 创建`Filter`

        ```java
        package com.ysu.filter;
        
        import javax.servlet.*;
        import java.io.IOException;
        
        public class SecondFilter implements Filter {
        
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
        
            }
        
            @Override
            public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
                filterChain.doFilter(servletRequest, servletResponse);
            }
        
            @Override
            public void destroy() {
        
            }
        }
        ```
        
        

    + 创建`Filter`配置类

        ```java
        package com.ysu.config;
        
        import com.ysu.filter.SecondFilter;
        import org.springframework.boot.web.servlet.FilterRegistrationBean;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        
        /**
         * Filter配置类
         */
        @Configuration
        public class FilterConfig {
            @Bean
            public FilterRegistrationBean getFilterRegistrationBean() {
                FilterRegistrationBean bean = new FilterRegistrationBean(new SecondFilter());
        //        配置多个过滤链接
        //        bean.addUrlPatterns(new String[]{"/first", "*.jsp"});
        //        配置单个过滤连接
                bean.addUrlPatterns("/secondServlet");
                return bean;
            }
        }
        
        ```
    
        

## 3. Listener整合

与整合Servlet、Filter相似

+ 通过注解扫描完成`Listener`组件的注册
    + 创建`Listener`

        ```java
        package com.ysu.listener;
        
        import javax.servlet.ServletContextEvent;
        import javax.servlet.ServletContextListener;
        import javax.servlet.annotation.WebListener;
        
        @WebListener
        public class FirstListener implements ServletContextListener {
            @Override
            public void contextInitialized(ServletContextEvent sce) {
                System.out.println("FirstListener Init!");
            }
        
            @Override
            public void contextDestroyed(ServletContextEvent sce) {
                System.out.println("FirstListener Destory!");
            }
        }
        ```

        

    + 修改启动类

        为启动类添加`@ServletComponentScan`注解

+ 通过方法完成Listener组件的注册
    + 创建`Listener`

        ```java
        package com.ysu.listener;
        
        import javax.servlet.ServletContextEvent;
        import javax.servlet.ServletContextListener;
        
        public class SecondListener implements ServletContextListener {
            @Override
            public void contextInitialized(ServletContextEvent sce) {
                System.out.println("SecondListener Init!");
            }
        
            @Override
            public void contextDestroyed(ServletContextEvent sce) {
                System.out.println("SecondListener Destory!");
            }
        }
        ```

        

    + 创建Listener启动类

        ```java
        package com.ysu.config;
        
        import com.ysu.listener.SecondListener;
        import org.springframework.boot.web.servlet.ServletListenerRegistrationBean;
        import org.springframework.context.annotation.Bean;
        import org.springframework.context.annotation.Configuration;
        
        @Configuration
        public class ListenerConfig {
            @Bean
            public ServletListenerRegistrationBean getListenerRegistrationBean() {
                ServletListenerRegistrationBean bean = new ServletListenerRegistrationBean(new SecondListener());
                return bean;
            }
        }
        ```

        

# 七. SpringBoot访问静态资源

​	在Spring Boot项目中没有常规web开发的WebContent(WebApp)，只有src目录。在src/main/resources下面有两个文件夹，static和templates。SprintBoot默认在**static目录中存放静态页面**，而**templates中存放动态页面**。

## 1. static目录

​	Spring Boot通过`classpath/static`目录访问静态资源。注意存放静态资源的目录名称**必须**是static。

## 2. templates目录

​	在Spring Boot中不推荐使用jsp作为视图层技术，而是默认使用Thymeleaf来做动态页面。templates目录是存放Thymeleaf的页面。**templates下的视图文件不允许通过url直接访问，而是通过controller去访问**。

# 八. Thymeleaf

## 1. 基本使用

+ 修改POM文件添加Thymeleaf启动器依赖

    ```xml
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    ```

+ 创建Controller

    ```java
    package com.ysu.controller;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.GetMapping;
    
    @Controller
    public class PageController {
        @GetMapping("/index")
        public String index(Model model) {
            model.addAttribute("msg", "Hello Thymeleaf");
            return "index";
        }
    }
    ```

+ 创建视图

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <span th:text="${msg}"></span>
    </body>
    </html>
    ```

## 2. Thymeleaf语法

添加命名空间

```html
<html xmlns:th="http://www.w3.org/1999/xhtml"> </html>
```

### 2.1 字符串与变量输出操作

|    语法    |               作用               |
| :--------: | :------------------------------: |
| `th:text`  |          在页面中输出值          |
| `th:value` | 将一个值放入`input`标签的value中 |

### 2.2 字符串操作

Thymeleaf提供了一些内置对象，内置对象可以直接在模板中使用。这些对象是以`#`引用的。

+ 使用内置对象的语法

    |                  语法                  |                        作用                        |
    | :------------------------------------: | :------------------------------------------------: |
    |       `$(#strings.isEmpty(key))`       |       判断字符串是否为空，如果为空返回`True`       |
    |    `$(#strings.contains(msg,'T'))`     | 判断字符串是否包含指定字符串，如果包含则返回`True` |
    |   `$(#strings.startsWith(msg,'a'))`    |    判断字符串是否以子串开头，如果是则返回`True`    |
    |     `$(#strings.endWith(msg,'a'))`     |    判断字符串是否以子串结尾，如果是则返回`True`    |
    |       `$(#strings.length(msg))`        |                   返回字符串长度                   |
    |     `$(#strings.indexOf(msg,'h'))`     |   查找子串位置并返回子串索引，如果不存在则返回-1   |
    |   `$(#strings.substring(msg,start))`   |                     截取字符串                     |
    | `$(#strings.substring(msg,start,end))` |                     截取字符串                     |
    |     `$(#strings.toUpperCase(msg))`     |                    字符串转大写                    |
    |     `$(#strings.tpLowerCase(msg))`     |                    字符串转小写                    |

### 2.3 日期格式化处理

|               语法                |                      作用                      |
| :-------------------------------: | :--------------------------------------------: |
|       `$dates.format(key)`        | 格式化日期，默认的以浏览器默认语言为格式化标准 |
| `$dates.format(key,'yyyy/MM/dd')` |           按照自定义的格式做日期转换           |
|        `$dates.year(key)`         |                     取年份                     |
|        `$dates.month(key)`        |                     取月份                     |
|         `$dates.day(key)`         |                     取日期                     |

### 2.4 条件判断

|        语法         |                         作用                          |
| :-----------------: | :---------------------------------------------------: |
|       `th:if`       |                       条件判断                        |
| `th:switch/th:case` | 与Java中语句类似，`th:case="*"`表示switch中的默认选项 |

### 2.5 迭代遍历

+ 迭代集合

|   语法    |             作用             |
| :-------: | :--------------------------: |
| `th:each` | 迭代器，用于循环**迭代集合** |

```html
<table>
    <tr>
        <th>id</th>
        <th>name</th>
        <th>age</th>
    </tr>
    <tr th:each="user: ${list}">
        <td th:text="${user.id}"></td>
        <td th:text="${user.name}"></td>
        <td th:text="${user.age}"></td>
    </tr>
</table>
```

+ 迭代器状态变量

    |  状态变量  |                  作用                  |
    | :--------: | :------------------------------------: |
    |  `index`   |   当前迭代器索引（索引从0开始计算）    |
    |  `count`   |             迭代器对象计数             |
    |   `size`   |            被迭代对象的长度            |
    | `odd/even` |    布尔值，当前索引是否是偶数/奇数     |
    |  `first`   |  当前是否是第一条，如果是则返回`True`  |
    |   `last`   | 当前是否是最后一条，如果是则返回`True` |

    **如果要使用迭代器的状态变量，那么状态变量所定义的变量名应该放到迭代器"`:`"的左侧**，迭代器内的数据对象则放在最左侧

    ```html
    <table>
        <tr>
            <th>id</th>
            <th>name</th>
            <th>age</th>
            <th>index</th>
        </tr>
        <!-- user为数据对象，temp为状态变量 -->
        <tr th:each="user, temp: ${list}">
            <td th:text="${user.id}"></td>
            <td th:text="${user.name}"></td>
            <td th:text="${user.age}"></td>
            <td th:text="${temp.index}"></td>
        </tr>
    </table>
    ```

+ 迭代`Map`

    ```html
    <tr th:each="m: ${map}">
        <td th:text="${m.value.key}"></td>
    </tr>
    ```

### 2.6 操作域对象

+ `HttpServletRequest`

    ```html
    <!-- 方式一 -->
    <span th:text="${#httpServletRequest.getAttribute('key')}"></span>
    <!-- 方式二 -->
    <span th:text="${#request.getAttribute('key')}"></span>
    ```

+ `HttpSession`

    ```html
    <!-- 方式一 -->
    <span th:text="${session.key}"></span>
    <!-- 方式二 -->
    <span th:text="${#session.getAttribute('key')}"></span>
    ```

+ `ServletContext`

    ```html
    <!-- 方式一 -->
    <span th:text="${application.key}"></span>
    <!-- 方式二 -->
    <span th:text="${#servletContext.getAttribute('key')}"></span>
    ```

### 2.7 URL表达式

+ 语法

在Thymeleaf中，URL表达式的语法格式为`@{url}`

|   类型   |                         语法                          |        备注        |
| :------: | :---------------------------------------------------: | :----------------: |
| 绝对路径 |  `<a th:href="@{http://www.baidu.com}">绝对路径</a>`  |                    |
| 相对路径 |         `<a th:href="@{/show}">相对路径</a>`          | 相对于当前项目的根 |
| 相对路径 | `<a th:href="@{~/project/resourcename}">相对路径</a>` |  想对于服务器的根  |

+ 在URL中传递参数

    + 普通格式的URL中传递参数

        ```html
        <a th:href="@{/show?id=18&name=zhangsan}">普通URL格式传参</a>
        <a th:href="@{/show?(id=18,name=zhangsan)}">普通URL格式传参</a>
        <a th:href="@{'/show?id=' + ${id} + '&name=' + ${name}}">普通URL格式传参</a>
        <a th:href="@{/show?(id=${id},name={name})}">普通URL格式传参</a>
        ```

    + restful格式的URL中传递参数

        ```html
        <a th:href="@{/show/{id}(id=1)}">resuful格式传参</a>
        <a th:href="@{/show/{id}/{name}(id=1,name=admin)}">resuful格式传参</a>
        <a th:href="@{/show/{id}(id=1,name=admin)}">resuful格式传参</a>
        <a th:href="@{/show/{id}(id=${id},name=${name})}">resuful格式传参</a>
        ```

        

# 九. SpringBoot异常处理

Spring Boot对于异常处理提供了5种方式

## 1. 自定义错误页面

​	SpringBoot默认的处理异常的机制：SpringBoot默认的已经提供了一套处理异常的机制。一旦程序中出现了异常，SpringBoot会向/error的url发送请求。在SpringBoot中提供了一个名为BasicErrorController来处理/error请求，然后跳转到默认显示异常的页面来展示异常信息。

如果我们需要将所有的异常统一跳转到自定义的错误页面，需要在src/main/resouces/templates目录下创建error.html页面。**注意：页面名称必须叫error**。

## 2. 通过@ExceptionHandler注解处理异常

在同一个Controller下添加`@ExceptionHandler`注解，即可抛出本Controller下的相关异常

+ 修改`Controller`

    为方法添加`@ExceptionHandler`注解，`value`值可指定异常类型（数组，即可指定多种异常）

    ```java
    package com.ysu.controller;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.servlet.ModelAndView;
    
    @Controller
    public class ExceptionController {
    
        @GetMapping("errorTest")
        public String errorTest() {
            String temp = null;
            temp.length();
            return "OK";
        }
    
        @ExceptionHandler(value = {java.lang.NullPointerException.class})
        public ModelAndView nullPointExceptionHandler(Exception e) {
            ModelAndView mv = new ModelAndView();
            mv.addObject("error", e.toString());
            mv.setViewName("nullPointError");
            return mv;
        }
    }
    ```

    

+ 创建异常页面

    此例中异常页面为nullPointError.html

该方法的弊端在于，处理异常的方法必须要和能够产生异常的方法在同一个Controller下才可以，这就导致了可能存在多个异常抛出方法且不能再其他Controller中复用。

## 3. 通过@ControllerAdvice与@ExceptionHandler注解处理异常

+ 创建全局异常处理类

    添加`@ControllerAdvice`注解

    ```java
    package com.ysu.exception;
    
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.servlet.ModelAndView;
    
    /**
     * 全局异常处理类
     */
    @ControllerAdvice
    public class GlobalException {
        @ExceptionHandler(value = {java.lang.NullPointerException.class})
        public ModelAndView nullPointExceptionHandler(Exception e) {
            ModelAndView mv = new ModelAndView();
            mv.addObject("error", e.toString());
            mv.setViewName("nullPointError");
            return mv;
        }
    }
    ```

    

该种方式的弊端在于，异常处理类中的方法会比较多以用来处理多种类型的异常

## 4. 通过SimpleMappingExceptionResolver处理异常对象

```java
package com.ysu.exception;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;

import java.util.Properties;

/**
 * 全局异常处理类
 */
@Configuration
public class GlobalException {

    /**
     * 该方法返回值必须是SimpleMappingExceptionResolver对象
     * @return
     */
    @Bean
    public SimpleMappingExceptionResolver getResolver() {
        SimpleMappingExceptionResolver resolver = new SimpleMappingExceptionResolver();
        Properties properties = new Properties();
        /**
         * 参数一：异常类型，且必须是全名
         * 参数二：视图名称
         */
        properties.put("java.lang.NullPointerException", "nullPointError");
        properties.put("java.lang.ArithmeticException", "arithmeticError");
        resolver.setExceptionMappings(properties);
        return resolver;
    }
}
```

该种方式的弊端在于，只能做异常与视图的映射，不能够传递异常信息

## 5. 通过HandlerExceptionResolver对象处理异常

自定义`HandlerExceptionResolver`对象处理异常，必须要实现`HandlerExceptionResolver`接口

```java
package com.ysu.exception;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 全局异常处理类
 */
@Configuration
public class GlobalException implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object handler, Exception e) {
        ModelAndView mv = new ModelAndView();
//        判断不同异常类型，做不同视图的跳转
        if(e instanceof NullPointerException){
            mv.setViewName("nullPointError");
        }
        if (e instanceof ArithmeticException) {
            mv.setViewName("arithmeticError");
        }
        mv.addObject("error", e.toString());
        return mv;
    }
}
```

# 十. SpringBoot整合Junit单元测试

+ 修改Pom文件添加Test启动器

    ```xml
    <!--	junit-vintage-engine提供了Junit3与Junit4的运行平台	-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    ```

    

+ 编写测试代码

Junit3/4测试类需要添加`@Runwith`与`@SpringBootTest`注解

```java
package com.ysu.test;

import com.ysu.Application;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = Application.class)
public class SprintBootTest {
    @Test
    void Test() {
        ....
    }
}
```

Junit5测试类只需要`@SprintBootTest`注解且无需指定application

```java
package com.ysu.test;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class SprintBootTest {
    @Test
    void Test() {
        ....
    }
}
```

# 十一. SpringBoot 热部署

通过DevTools工具实现热部署

1. 修改Pom文件，添加DevTools依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <version>2.0.4.RELEASE</version>
</dependency>
```

2. 设置IDEA自动编译

    `settings`->`Build, Execution, Deployment`->`compiler`->`Build project automatically`

3. 设置IDEA的Registry

    使用快捷键`Ctrl+Shift+Alt+/`打开Registry页面

    勾选`compiler.automake.allow.when.app.running`