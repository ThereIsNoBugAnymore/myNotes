[TOC]

# 一. Maven下载安装及配置

## 1. 下载

下载地址：https://maven.apache.org/download.html

![clip_image001](D:\笔记\assets\maven_download.png)



## 2. 目录结构

+ bin

  mvn在bin目录中，主要用来构建项目

+ boot

  maven自身运行所需要的类加载器

+ conf

  settings.xml是maven的主要配置文件

+ lib

  maven自身运行所需要的jar包



## 3. 环境变量配置

1. 添加环境变量**MAVEN_HOME**，变量值为maven目录路径
2. 添加环境变量**PATH**，变量值为maven下bin目录
3. 检测环境变量配置是否成功，cmd运行**mvn -v**查看maven版本信息



## 4. 设置本地仓库和中央仓库

修改conf/settings.xml文件修改相关设置

+ 修改本地仓库位置

  添加`localRepository`节点，设置本地仓库位置

  ```xml
  <localRepository>D:/maven/repository</localRepository>
  ```

  

+ 修改中央仓库位置

  阿里云中央仓库镜像：https://maven.aliyun.com/mvn/view

  在`mirrors`结点中添加`mirror`节点，设置相关信息

  ```xml
  <mirrors>
  	<mirror>
  		<id>aliyunmaven</id>
  		<mirrorOf>*</mirrorOf>
  		<name>阿里云公共仓库</name>
  		<url>https://maven.aliyun.com/repository/public</url>
  	</mirror>
  </mirrors>
  ```



# 二. Maven标准目录及简单命令

## 1. 目录结构

+ src
  + main
    + java	——核心代码部分
    + resources ——配置文件部分
  + test
    + java ——测试代码部分
    + resources ——测试配置文件
  + webapp（web工程存在该目录，保存页面资源文件，js/css/html等）

## 2. 命令

+ mvn clean——清除之前构建的项目
+ mvn compile——编译当前项目
+ mvn test——编译核心代码及测试代码
+ mvn package——项目编译及打包，打包格式受pom中<package>标签控制，默认为jar包
+ mvn install——项目编译并打包，并将该包安装到本地仓库
+ mvn deploy——项目发布

# 三. IDEA/Maven集成配置

## 1. 基本环境配置

设置为本地maven

![基本环境配置](D:\笔记\assets\maven_environment.png)

## 2. 参数设置（非必须）

一般情况下创建Maven项目是需要联网的，设置此参数后会直接从本地需要相应插件而不从网络下载，防止在网络不良的情况下不能创建项目

参数：-DarchetypeCatalog=internal

![参数设置](D:\笔记\assets\maven_parameter.png)



# 四. Maven工程环境修改

在pom.xml中添加相关结点，修改环境配置

## 1. 修改tomcat版本

```xml
<plugin>
	<groupId>org.apache.tomcat.maven</groupId>
	<artifactId>tomcat7-maven-plugin</artifactId>
	<version>2.2</version>
	<configuration>
		<port>8080</port>
	</configuration>
</plugin>
```



## 2. 修改jdk版本

```
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<configuration>
		<target>1.8</target>
		<source>1.8</target>
		<encoding>UTF-8</encoding>
	</configuration>
</plugin>
```

