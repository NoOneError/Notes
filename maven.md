



# Maven简介

## 没有Maven存在的问题

①、一个项目就是一个web工程。如果项目比较庞大，那么利用包名package来划分模块，显然容易造成混淆而且不利于分工合作；

②、项目中需要的 jar 包必须手动 复制，粘贴 到 WEB-INF/lib 目录下。这会导致每创建一个新的工程就需要将 jar 包重复复制到 lib 目录下，从而造成工作区存在大量重复的文件；

③、jar需要我们手动去官网上或者其他途径下载；

④、一个 jar 包依赖的其他 jar 包，需要自己手动加入到项目中，而且很有可能我们漏掉了某个依赖关系，导致项目运行报错。

 　那么如何解决这些问题呢？本系列的主角 Maven 应运而生了。

## Maven能做什么

1. 项目的自动创建，帮助开发人员做项目代码的编译，测试，打包，安装，部署的工作
2. 管理依赖（管理项目中使用的各种jar包）

# Maven中的核心概念

## pom

pom.xml

## 坐标

坐标组成是groupid、artifactId、version。坐标概念来自数学

坐标作用：确定资源的，是资源的唯一标识。在maven中，每个资源都是坐标。坐标值都是唯一的。简称						gav。

```xml
<groupId>com.datayes.pfyh</groupId>
<artifactId>mom</artifactId>
<version>20.0.0-BUILD_NUMBER</version>、
<packaging>jar</packaging>


groupId: 组织名称，代码。公司，团体或者单位的标识。这个值通常是公司域名的倒写。和package包名一样
		例如， 公司域名： www.xxx.com  groupId： com.xxx
artifactId: 项目名称，如果groupId中有项目，此时当前的值就是子项目名。项目名是唯一的。
version: 项目的版本号，使用的数字。三位数组成。例如 主版本号，次版本号，小版本号。例如：5.2.5.
		 注意，版本号中有-SNAPSHOT,标识快照，不是稳定的版本

packaging: 项目打包的类型，有jar，war，ear，pom。默认是jar  放在gav下面
```

## 依赖 dependency

依赖：项目中要使用的其他资源  jar包。

需要使用maven表示依赖，管理依赖。永昌使用dependency和gav一起完成依赖使用

```xml
    <dependencies>
       <!-- springboot依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
	</dependencies>
```

## 继承，聚合

在父工程中统一管理项目的依赖信息，具体来说是管理依赖信息的版本。

在子工程中使用坐标就不需要指定版本号，实现统一管理。



他的背景是：

- 对一个比较大型的项目进行了模块的拆分
- 一个project下面，创建 了很多个moudel
- 每一个moudel都需要配置自己的依赖信息



背后的需求是：

- **每一个moudel中各自维护各自的依赖信息容易发生出入，不好管理**
- 使用同一个框架，版本信息应该一致
- 依赖信息组合，需要经过长时间摸索和反复调试，最终确定一个可用的组合



> 父工程pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <!--modelVersion：指定了当前POM模型的版本，对于Maven2及Maven 3来说，它只能是4.0.0；-->
    <modelVersion>4.0.0</modelVersion>
    <!-- GAV -->
    <groupId>com.datayes.pfyh</groupId>
    <artifactId>mom</artifactId>
    <version>20.0.0-BUILD_NUMBER</version>
    <!--使用maven分模块管理，都会有一个父级项目，pom文件一个重要的属性就是packaging（打包类型），一般来说所有的父级项目的packaging都为pom，packaging默认类型jar类型，如果不做配置，maven会将该项目打成jar包-->
    <packaging>pom</packaging>   父工程中必须指定为  pom  
    
    <! --聚合 -- >
    <modules>
    	<module>dg</module>   子工程的artifactId
    </modules>
    
    <properties>
         <java.version>1.8</java.version>
         <spring-boot.version>2.2.0.RELEASE</spring-boot.version>
    </properties>
    
    <dependencyManagement>
        提供了一种管理依赖版本号的方式。在dependencyManagement元素中声明所依赖的jar包的版本号等信息，那么所有子项目再次引入此依赖jar包时则无需显式的列出版本号---------！！！！只s
        <dependencys>
        	<dependency>
           		 <groupId>mysql</groupId>
          		  <artifactId>mysql-connector-java</artifactId>
          		  <version>8.0.17</version>
       		 </dependency>
        </dependencys>
    </dependencyManagement>
```

> 子工程pom

```xml
   <parent>
        <groupId>com.datayes.pfyh</groupId>
    	<artifactId>mom</artifactId>
   	    <version>20.0.0-BUILD_NUMBER</version>
    </parent>

		<groupId>com.datayes.pfyh</groupId>
    	<artifactId>dg</artifactId>
   	    <version>20.0.0-BUILD_NUMBER</version>

       <dependencys>
           <! -- 子工程中使用父工程声明的依赖，不需要指定版本 -- >
           <dependency>
           		 <groupId>mysql</groupId>
          		 <artifactId>mysql-connector-java</artifactId>
       		 </dependency>
        </dependencys>

```



## 仓库

仓库是存放jar包的

1. maven工具自己的jar包
2. 第三方的其他jar包
3. 自己写的项目打成jar



仓库的分类：

1. 本地仓库：位于自己的计算机，在磁盘中的某个目录。默认路径是   C:/账号/.m2/respository
2. 中央仓库：maven官方仓库
3. 私服：在局域网中使用的，私服就是自己的仓库服务器，公司内部使用

## Maven的生命周期，插件，命令

生命周期： 项目构建的各个阶段，包括清理，编译，测试，报告，打包，安装，部署

插件： 要完成构建项目的各个阶段，要使用maven的命令。执行命令的功能是通过插件完成的。插件就是    	   		jar的一些类。

命令： 执行maven功能是由命令发出的，例如  mvn clean

**注意：**打包的时候，排除测试文件 mvn install -DskipTests





# jar包冲突

一个java项目需要多个依赖来完成。打成jar包之后，依赖也会集成到jar包里面。

**但是**，当我们引入多个jar包，都有同一个依赖，而且版本号不相同。就会发生依赖冲突



==排除依赖冲突：==





# 导入Maven项目

1、idea打开file，选择导入

![image-20220902173827023](F:\note\images\image-20220902173827023.png)

2、选择项目的pom文件（父pom）

![image-20220902173848458](F:\note\images\image-20220902173848458.png)

3、选择maven、以及仓库

![image-20220902173950888](F:\note\images\image-20220902173950888.png)

4、清理仓库里面的_remote和.latst文件

5、reload project、clean 、install

![image-20220902174125371](F:\note\images\image-20220902174125371.png)

6、选择启动类，配置启动参数，**其他**

- JVM启动参数，包括内存最小最大值，本地配置文件路径、项目路径
- Working directory：$MODULE_WORKING_DIR$
- 选择 Include dependencies with "Provided" scope
- 选择 classpath file

7、run 启动类

