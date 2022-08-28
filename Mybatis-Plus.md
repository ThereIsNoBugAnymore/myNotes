[TOC]

`Mybatis-Plus`可以简单理解为`Mybatis`的加强，是在`Mybatis`的基础上完成的一些封装和扩展，只做增强不做改变，不会对现有`Mybatis`架构产生影响，官网地址：https://mp.baomidou.com/

<font color='red' size=10>WARNING</font> 引入`Mybatis-Plus`后<font color='red' size=5>不要再次引入`MyBatis`及`MyBatis-Spring`</font>，以避免因版本差异导致出现问题！

# 一、优点

- 无侵入：对`Mybatis`只做增强不做改变，支持所有`Mybatis`原生的特性

- 依赖少：仅仅依赖`Mybatis`以及`Mybatis-Spring`
- 损耗小：启动即会自动注入基本CRUD，性能基本无损耗，直接面向对象操作
- 预防`SQL`注入：内置`SQL`注入剥离器，有效预防`SQL`注入攻击
- 通用CRUD操作
- 多种主键策略：支持4种主键策略，可自由配置
- 支持热加载：Mapper对应的XML支持热加载，对于简单的CRUD操作，甚至可以无XML启动
- 支持`ActiveRecord`
- <font color='red'>支持代码生成：采用代码可快速生成`Mapper`/`Model`/`Service`/`Controller`层代码，支持模板引擎</font>
- 支持自定义全局通用操作：支持全局通用方法注入
- 支持关键词自动转义
- <font color='red'>内置分页插件</font>
- <font color='red'>内置性能分析插件：可输出`SQL`语句以及其执行时间</font>
- 内置全局拦截插件：提供全表`delete`/`update`操作智能分析阻断，预防误操作

# 二、自动代码生成

[点击即可查看详细配置参数](https://baomidou.com/pages/981406/#%E6%95%B0%E6%8D%AE%E5%BA%93%E9%85%8D%E7%BD%AE-datasourceconfig)

配置模板类

```java
import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import org.apache.commons.lang3.StringUtils;

import java.util.Collections;
import java.util.Scanner;

public class CodeGenerator {

    /**
     * 读取控制台内容
     */
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append("请输入" + tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("请输入正确的" + tip + "！");
    }

    public static void main(String[] args) {

        String projectPath = System.getProperty("user.dir");


        FastAutoGenerator.create("jdbc:mysql://localhost:3306/smart-admin-dev?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai", "root", "123456")
                .globalConfig(builder -> {
                    builder.fileOverride()  // 覆盖已生成文件
                            .outputDir(projectPath + "/smart_admin/src/main/java")  // 输出目录
                            .author("王铁锤")  // 作者
                            .enableSwagger()    // 开启 swagger
                            .dateType(DateType.TIME_PACK)
                            .commentDate("yyyy-MM-dd");
                })
                .packageConfig(builder -> {
                    builder.parent("com.example.smart_admin")
                            .entity("pojo")
                            .service("service")
                            .serviceImpl("service.impl")
                            .mapper("mapper")
                            .controller("controller")
                            .pathInfo(Collections.singletonMap(OutputFile.mapperXml, projectPath + "/smart_admin/src/main/resources/mapper"));
                })
                .strategyConfig(builder -> {
                    builder.addInclude(scanner("表名，多表用英文逗号','分隔").split(","))
                            .addTablePrefix("t_", "qrtz_")
                            .entityBuilder()
                            .naming(NamingStrategy.underline_to_camel)
                            .columnNaming(NamingStrategy.underline_to_camel)
                            .enableLombok()
                            .enableTableFieldAnnotation()
                            .controllerBuilder()
                            .formatFileName("%sController")
                            .enableRestStyle()
                            .serviceBuilder()
                            .formatServiceFileName("%sService")
                            .formatServiceImplFileName("%sServiceImp")
                            .mapperBuilder()
                            .enableMapperAnnotation()
                            .enableBaseResultMap()
                            .enableBaseColumnList()
                            .superClass(BaseMapper.class)
                            .formatMapperFileName("%sDao")
                            .formatXmlFileName("%sXml");
                }).execute();
    }
}
```

