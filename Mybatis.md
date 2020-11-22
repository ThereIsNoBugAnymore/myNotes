[TOC]

`MyBatis`是一个持久层框架，用Java编写。它封装了`jdbc`操作的很多细节，使开发者只需要关注`SQL`语句本身，而无需关注注册驱动、创建连接等繁杂过程。使用`ORM`（Object Relational Mapping，对象关系映射）思想实现结果集的封装，简单的说就是把数据库表和实体类及实体类的属性对应起来，让我们可以操作实体类就实现操作数据库表。

# 一、`Mybatis`安装及配置

## 1. 环境搭建

以Maven方式搭建`Mybatis`环境

修改`pom.xml`文件以添加`Mybatis`依赖及数据库相关依赖

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>x.x.x</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>x.x.x</version>
</dependency>
```

## 2. 开发配置

### 2.1 通过配置文件开发

创建步骤：

- 创建maven工程并导入相关依赖

- 创建实体类和`dao`的接口

    + 实体类（变量名称需跟数据库中名称一致）

    ```java
    package com.ysu.domain;
    
    import java.io.Serializable;
    
    public class User implements Serializable {
        private String id;
        private String pwd;
    
        public String getId() {
            return id;
        }
    
        public void setId(String id) {
            this.id = id;
        }
    
        public String getPwd() {
            return pwd;
        }
    
        public void setPwd(String pwd) {
            this.pwd = pwd;
        }
    }
    ```

    + 接口文件

    ```java
    package com.ysu.dao;
    
    import com.ysu.domain.User;
    
    import java.util.List;
    
    public interface UserDao {
        List<User> getAllUser();
    }
    ```

    

- 创建`Mybatis`的主配置文件（resources目录下）

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!-- 配置环境 -->
        <environments default="mysql">
            <!-- 配置可以使用的环境 -->
            <environment id="mysql">
                <!-- 配置事务类型 -->
                <transactionManager type="JDBC"></transactionManager>
                <!-- 配置数据源（连接池）-->
                <dataSource type="POOLED">
                    <!-- 配置数据库的4个基本信息 -->
                    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/数据库"/>
                    <property name="user" value="root"/>
                    <property name="password" value="123456"/>
                </dataSource>
            </environment>
        </environments>
    
    <!--    指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件-->
        <mappers>
            <mapper resource="com/ysu/dao/UserDao.xml"/>
        </mappers>
    </configuration>
    ```

    

- 创建映射配置文件（resources目录下）

    注意：***`Mybatis`的映射配置文件位置必须和`dao`接口的包结构相同***

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    
    <!--命名空间为对应接口的全限定类名-->
    <mapper namespace="com.ysu.dao.UserDao">
    <!--    id为对应的方法名称
            parameterType定义参数类型
            resultType指定返回值数据类型
    -->
        <select id="getAllUser" resultType="com.ysu.domain.User">
            <!-- sql语句 -->
            SELECT * FROM user
    	</select>
    </mapper>
    ```
    
    

+ 测试（只有在单独使用`Mybatis`时使用，整合`SSM`时如下代码不需要编写）

    ```java
    package com.ysu.test;
    
    import com.ysu.dao.UserDao;
    import com.ysu.domain.User;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    
    import java.io.InputStream;
    import java.util.List;
    
    public class Test {
        public static void main(String[] args) throws Exception {
    //        1.读取配置文件
            InputStream in = Resources.getResourceAsStream("mybatis.xml");
    //        2.创建SqlSessionFactory工厂
            SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
            SqlSessionFactory factory = builder.build(in);
    //        3.使用工厂生产SqlSession对象
            SqlSession session = factory.openSession();
    //        4.使用SqlSession创建Dao接口的代理对象
            UserDao userDao = session.getMapper(UserDao.class);
    //        5.使用代理对象执行方法
            List<User> userList = userDao.getAllUser();
    
            for (User user: userList) {
                System.out.println(user);
            }
    //        6.释放资源
            session.close();
            in.close();
        }
    }
    ```

### 2.2 通过注解开发

创建步骤

+ 创建maven工程并导入相关依赖

+ 创建实体类和`dao`的接口

    + 实体类（同上）

    + ```java
        package com.ysu.dao;
        
        import com.ysu.domain.User;
        import org.apache.ibatis.annotations.Select;
        
        import java.util.List;
        
        public interface UserDao {
            /**
             * 使用注解方式配置
             */
            @Select("SELECT * FROM user")
            List<User> getAllUser();
        }
        
        ```

        

+ 创建`Mybatis`主配置文件

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        …………………………
    <!--    指定映射配置类的位置-->
        <mappers>
            <mapper class="com.ysu.dao.UserDao"></mapper>
        </mappers>
    </configuration>
    ```

+ 测试（同上）

## 3. 环境配置详解

### 3.1 `<transactionManager> `

事务管理方式，type属性可取值`JDBC`或`MANAGED`

| type取值 |                作用                |
| :------: | :--------------------------------: |
|  `JDBC`  | 事务管理使用`JDBC`原生事务管理方式 |
| MANAGED  |      把事务管理转交给其他容器      |

### 3.2 `<dataSource>`

配置数据库连接池，type属性可取值为`UNPOOLED`、`POOLED`或`JNDI`

|  type取值  |                       作用                       |
| :--------: | :----------------------------------------------: |
| `UNPOOLED` |                不使用数据库连接池                |
|  `POOLED`  |                 使用数据库连接池                 |
|   `JNDI`   | `java`命名目录接口技术，调用其他技术所设计的资源 |

+ 数据库连接池

    数据库连接池就是在内存中开辟一块空间，存放多个数据库连接对象

    + 连接池中连接对象状态分布：

        + `active`：当前连接对象被应用程序使用中
        + `Idle`：等待应用程序使用

    + 使用数据库连接池的目的：

        在高频率访问数据库时，使用数据库连接池可以降低服务器系统压力，提高程序运行效率（小型小牧不适用于数据库连接池）

# 二、`Log4j`

`Log4j`是**Apache**推出的开源免费日处理的**类库**

## 1. 环境搭建

+ 为什么需要日志？
    1. 在项目中编写`System.out.println();`输出到控制台，当项目发布到tomcat后，没有控制台（虽然在命令行中可见），但不容易观察一些输出结果
    2. `log4j`的作用，不仅能把内容输出到控制台，还能把内容输出到文件中

+ 配置步骤

    1. 使用Maven方式添加`Log4j`依赖

        ```xml
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>x.x.x</version>
        </dependency>
        ```

    2. `resourses`路径（此为`IDEA`下，`eclipse`需要在`src`目录下创建配置文件）下创建`log4j.properties`配置文件（路径和名称均不可变更）

+ 配置模板

    ```properties
    log4j.rootCategory=DEBUG, CONSOLE, LOGFILE
    
    log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
    log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
    log4j.appender.CONSOLE.layout.ConversionPattern=%C %d{YYYY-MM-dd HH:mm:ss} %m %n
    
    log4j.appender.LOGFILE=org.apache.log4j.FileAppender
    log4j.appender.LOGFILE.File=my.log
    log4j.appender.LOGFILE.Append=true
    log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
    log4j.appender.LOGFILE.layout.ConversionPattern=%C %m %L %n %d{YYYY-MM-dd HH:mm:ss}
    ```

## 2. 参数配置

+ `Log4j`输出内容控制

    在上述配置模板中，`INFO`控制总体输出级别

    输出级别关系为：`FATAL(致命错误)`>`ERROR(错误/异常)`>`WARN(警告)`>`INFO(普通信息)`>`DEBUGZ(调试信息)`，上述配置控制输出大于设置级别的内容，如上述设置内容则输出`FATAL/ERROR/WARN/INFO`的相关日志信息

+ `Log4j`控制输出目的地

    + 控制台配置

        在上述配置模板中，`CONSOLE, LOGFILE`字段控制输出目的地

        如配置内容为`log4j.rootCategory=INFO, CONSOLE`则表示`log4j`信息会输出到控制台

        如配置内容为`log4j.rootCategory=INFO, CONSOLE, LOGFILE`则表示`log4j`信息会输出到控制台和文件

        ```properties
        # 负责输出的类
        log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
        # 负责输出的格式
        log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
        # 具体的表达式内容
        log4j.appender.CONSOLE.layout.ConversionPattern=%C %d{YYYY-MM-dd HH:mm:ss} %m %n
        ```

    + 文件输出配置

        文件输出配置与控制台控制类似

        ```properties
        log4j.appender.LOGFILE=org.apache.log4j.FileAppender
        # 负责控制输出文件
        log4j.appender.LOGFILE.File=my.log
        # 控制是否以追加方式控制文件输出
        log4j.appender.LOGFILE.Append=true
        log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
        log4j.appender.LOGFILE.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
        ```

    + `pattern`中常用的几个表达式

        | 表达式 |                   含义                   |
        | :----: | :--------------------------------------: |
        |  `%C`  |                包名+类名                 |
        | `%d{}` | 时间，可在大括号中使用表达式控制时间格式 |
        |  `%L`  |                   行号                   |
        |  `%m`  |                   信息                   |
        |  `%n`  |                   换行                   |
        |  `%p`  |                 日志级别                 |

## 3. `Mybatis`整合`Log4j`

通过`Mybatis`全局配置文件的`<settings>`标签控制`Mybatis`全局开关，相关设置会修改`Mybatis`在运行时的行为方式，整合`Log4j`只需关注`logImpl`相关设置内容

| 设置参数  |                             描述                             |                            有效值                            | 默认值 |
| :-------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----: |
| `logImpl` | 指定`Mybatis`应该使用哪个日志实现。如果此设置不存在，将自动发现日志记录实现。 | `SLF4J`|`LOG4J`|`LOG4J2`|`JDK_LOGGING`|`COMMONS_LOGGING`|`STDOUT_LOGGING`|`NO_LOGGING` | 未设置 |

**注意：<font color="red" size="5">一定要严格按照`dtd`的标签个数及顺序编写配置文件！</font>**

**`<settings>`标签必须要在`<environments>`标签之前编写！**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 全局配置 -->
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    ……
</configuration>
```

+ `Log4j`中可以输出指定内容的日志（控制局部内容的日志级别）

    + 命名空间（包）级别
    + 类级别
    + 方法级别

    ```properties
    log4j.rootCategory=DEBUG, CONSOLE, LOGFILE
    
    # 包级别
    log4j.logger.com.ysu.dao=INFO
    # 类级别
    log4j.logger.com.ysu.dao.UserDao=ERROR
    # 方法级别
    log4j.logger.com.ysu.dao.UserDao.getAllUser=FATAL
    ```

    上述配置实现全局日志内容输出级别为DEBUG，`com.ysu.dao`包下日志输出级别为`INFO`，`com.ysu.dao.UserDao`类下日志输出级别为`ERROR`，`com.ysu.dao.UserDao.getAllUser`方法下日志输出级别为`FATAL`

# 三、参数传递 

在配置文件`Mapper`中添加参数

## 1. 使用占位符方式

+ 使用`#{n}`占位符`?`方式代替参数，也可以使用`#{paramN}`指代第N个参数（`param`为固定字符串）

    以`#{n}`指代传递的参数，n表示参数的索引，即：`#{0}`与`#{param1}`含义相同，共同指向第一个参数

+ 如果只有一个参数（基本数据类型或`String`），`Mybatis`对`#{}`中的内容没有要求，只要写内容即可
+ 如果传输参数为对象，则在`SQL`中写作`#{属性名}`
+ 如果传输参数为`Map`，则在`SQL`中写作`#{key}`

```xml
<!-- 传输基本数据类型 -->
<select id="getUserById" resultType="com.ysu.domain.User" parameterType="int">
    select * from user where id = #{0}
</select>
<!-- 传输对象 -->
<select id="getUserById" resultType="com.ysu.domain.User" parameterType="com.ysu.domain.User">
    select * from user where id = #{id}
</select>
```

## 2 使用字符串拼接方式

使用`${}`，字符串拼接方式代替参数

```xml
<!-- 传输基本数据类型 -->
<select id="getUserById" resultType="com.ysu.domain.User" parameterType="int">
    select * from user where id = ${0}
</select>
<!-- 传输对象 -->
<select id="getUserById" resultType="com.ysu.domain.User" parameterType="com.ysu.domain.User">
    select * from user where id = ${id}
</select>
```

## 3. `#{}`与`${}`的区别

+ `#{}`获取参数的内容支持索引或顺序获取，并且相当于使用在`SQL`中使用占位符`?`
+ `${}`使用字符串拼接方式构建`SQL`，默认找`${对象}`中对象的`get`/`set`方法，如果传输数字，则其就是一个数字

# 四、`typeAliases`别名

## 1. 系统内置别名

|     别名     |  映射的类型  |
| :----------: | :----------: |
|   `_byte`    |    `byte`    |
|   `_long`    |    `long`    |
|   `_short`   |   `short`    |
|    `_int`    |    `int`     |
|  `_integer`  |    `int`     |
|  `_double`   |   `double`   |
|   `_float`   |   `float`    |
|  `_boolean`  |  `boolean`   |
|   `string`   |   `String`   |
|    `byte`    |    `Byte`    |
|    `long`    |    `Long`    |
|   `short`    |   `Short`    |
|    `int`     |  `Integer`   |
|  `integer`   |  `Integer`   |
|   `double`   |   `Double`   |
|   `float`    |   `Float`    |
|  `boolean`   |  `Boolean`   |
|    `date`    |    `Date`    |
|  `decimal`   | `BigDecimal` |
| `bigdecimal` | `BigDecimal` |
|   `object`   |   `Object`   |
|    `map`     |    `Map`     |
|  `hashmap`   |  `HashMap`   |
|    `list`    |    `List`    |
| `arraylist`  | `ArrayList`  |
| `collection` | `Collection` |
|  `iterator`  |  `Iterator`  |

## 2. 为某个类定义别名

在全局配置文件通过`<typeAliases>`标签添加别名

**注意：<font color="red" size="5">一定要严格按照`dtd`的标签个数及顺序编写配置文件！</font>**

**`<typeAliases>`标签必须要在`<environments>`标签之前、`<settings>`之后编写！**

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>……</settings>

	<!-- 别名 -->
    <typeAliases>
        <typeAlias type="com.ysu.domain.User" alias="user"/>
    </typeAliases>

    <environments></environments>
</configuration>
```

如上配置，则可以在映射配置文件下直接使用

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ysu.dao.UserDao">
    <select id="getUserById" resultType="user" parameterType="int">
        SELECT * FROM user WHERE id = #{0}
    </select>
</mapper>
```

## 3. 为某个包下的所有类定义别名

在全局配置文件通过`<typeAliases>`标签下添加`<package>`为包下所有类添加别名，别名为类名

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>……</settings>
	<!-- 别名 -->
    <typeAliases>
        <package name="com.ysu.dao"/>
    </typeAliases>
    <environments></environments>
</configuration>
```

如上配置时，可直接用`com.ysu.dao`包下的类名作为各类的别名

# 五、增删改操作

## 1. 基本概念

+ 功能

    从应用程序角度出发，软件具有哪些功能

+ 业务

    完成功能时的逻辑，对应service中的一个方法

+ 事务

    从数据库角度出发，完成业务时需要执行的`SQL`集合，统称为一个事务

    + 事务回滚

        如果一个事务中某个`SQL`执行失败，则希望回归到事务的原点，保证数据库数据的完整性

## 2. 增删改操作

`Mybatis`中默认关闭了`JDBC`的自动提交功能，每一个`SqlSession`默认都是不会自动提交事务，通过`session.commit()`提交事务或通过`openSession(true)`设置事务自动提交。

`Mybatis`底层是对`JDBC`的封装，`JDBC`中的`executeUpdate()`执行增删改时返回类型均为`int`，表示受影响的行数，因此`Mybatis`中`<insert>`/`<delete>`/`<update>`均没有`resultType`属性，返回值类型均为`int`

如果在增删改过程中出现数据异常，应通过`session.rollback()`回滚事务

# 六、`Mybatis`接口绑定方案及多参数传递

## 1. 作用

实现创建一个接口后把`mapper.xml`由`Mybatis`生成接口的实现类，通过调用接口对象就可以获取`mapper.xml`中编写的`SQL`

## 2. 实现步骤

1. 创建一个接口

    接口的包名和方法名要与`mapper.xml`中的`namespace`和`id`分别对应（相同）

2. 再主配置文件`mybatis.xml`中使用`<package>`进行扫描接口

    ```xml-dtd
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        …………………………
        <mappers>
            <package class="com.ysu.dao.UserDao"></package>
        </mappers>
    </configuration>
    ```

3. 通过代理对象调用接口

    ```java
    package com.ysu.test;
    
    import com.ysu.dao.UserDao;
    import com.ysu.domain.User;
    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;
    
    import java.io.InputStream;
    import java.util.List;
    
    public class Test {
        public static void main(String[] args) throws Exception {
            InputStream in = Resources.getResourceAsStream("mybatis.xml");
            SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
            SqlSessionFactory factory = builder.build(in);
            SqlSession session = factory.openSession();
    //      使用SqlSession创建Dao接口的代理对象
            UserDao userDao = session.getMapper(UserDao.class);
    //        5.使用代理对象执行方法
            List<User> userList = userDao.getAllUser();
            for (User user: userList) {
                System.out.println(user);
            }
    //        6.释放资源
            session.close();
            in.close();
        }
    }
    ```

## 3. 多参数传递

通过接口多参数传递是不需要设置`parameterType`属性便可直接使用

+ 接口文件

```java
package com.ysu.dao;

import com.ysu.domain.User;

import java.util.List;

public interface UserDao {
    User login(String account, String pwd);
}
```

+ 映射文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--命名空间为对应接口的全限定类名-->
<mapper namespace="com.ysu.dao.UserDao">
    <select id="login" resultType="com.ysu.domain.User">
        SELECT * FROM user WHERE account = #{account} AND password = #{password}
	</select>
</mapper>
```

# 七、动态`SQL`

动态`SQL`会根据条件的不同，生成不同的`SQL`语句，这个即称为动态`SQL`

`Mybatis`中动态`SQL`在配置文件`mapper.xml`中添加逻辑判断等

## 1. `if`的使用

+ demo

```xml-dtd
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.ysu.dao.UserDao">
    <select id="searchUser" resultType="com.ysu.domain.User">
    	SELECT * FROM user WHERE 1=1
    	<!-- OGNL表达式，直接写key或对象的属性，不需要添加任何符号 -->
    	<if test="account != null and account != ''">
    		and account = #{account}
    	</if>
		<if test="accin != null andd accout != ''">
			and accin = #{accin}
		</if>
	</select>
</mapper>
```

当传入值`account`与`accin`均不为空时，相当于生成`SQL`语句

```sql
SELECT
	* 
FROM
user 
WHERE
	1 = 1 
	AND account = account 
	AND accin = accin
```

## 2. `where`的使用

当编写`where`标签时，如果内容中第一个含有`AND`，会自动去掉第一个`AND`

如果`<where> </where>`中会内容，则会生成`where`关键字

+ demo

```xml-dtd
<select id="searchUser" resultType="com.ysu.domain.User">
    SELECT * FROM user
    <where>
        <if test="account != null and account != ''">
        	and account = #{account}
        </if>
        <if test="accin != null andd accout != ''">
        	and accin = #{accin}
        </if>
    </where>
</select>
```

+ 当传入值`account`与`accin`均为空时，相当于生成`SQL`语句

```sql
SELECT
	* 
FROM
user
```

+ 当传入值`accin`为空时会自动去掉第一个`AND`关键字，相当于生成`SQL`语句

```sql
SELECT
	* 
FROM
user 
WHERE
	account = account
```

+ 当传入值`account`与`accin`均不为空时，相当于生成`SQL`语句

```sql
SELECT
	* 
FROM
user 
WHERE
	account = account 
	AND accin = accin
```

## 3.  `choose`/`when`/`otherwise`的使用

`Mybatis`提供的`choose`元素与`java`中的`switch`语句相似，为多种情况下选择一种执行情况（只要一个条件成立，其他情况均不执行）

+ demo

    如果`accin`和`account`都不是`null`或不是空串`''`，则生成的`sql`中只有`where accin=accin`

```xml-dtd
<select id="searchUser" resultType="com.ysu.domain.User">
    SELECT * FROM user
    <where>
        <choose>
            <when test="account != null and account != ''">
            	and account = #{account}
            </when>
            <when test="accin != null andd accout != ''">
            	and accin = #{accin}
            </when>
        </choose>
    </where>
</select>
```

## 4. `set`的使用

用在修改`SQL`的`set`从句中，用于去掉最后一个逗号

如果`<set> </set>`里面有内容，则会生成`set`关键字，没有则不生成

+ demo

```xml-dtd
<select id="searchUser" resultType="com.ysu.domain.User">
    SELECT * FROM user
    <where>
        <choose>
            <when test="account != null and account != ''">
            	and account = #{account}
            </when>
            <when test="accin != null andd accout != ''">
            	and accin = #{accin}
            </when>
        </choose>
    </where>
</select>
```

