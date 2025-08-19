---
title: Maven依赖冲突解决方法
catalog: true
date: 2022-04-28 02:34:17
subtitle: Maven依赖冲突解决方法
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- Maven
categories:
- Maven
---

##### Maven依赖冲突解决方法

项目的pom.xml形如下：

项目会引用5.2.7.RELEASE的spring core jar包

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
    </dependencies>


```





依赖冲突是指项目依赖中的jar包有多个版本，造成了版本冲突，经常是由于类包之间的间接依赖（隐式包）（造成了不同版本依赖的重复引入和共存）形成的

###### 如何解决：

首先查看产生依赖冲突的jar，通过dependency:tree命令来检查版本冲突(不建议)

使用maven helper插件进行排查依赖，解决冲突使用版本锁定的方法

安装插件之后打开pom文件，下面会出现一个“依赖分析”选项，插件中有三个选项，Conflicts代表查看冲突、下两个选项是已列表或者树的形式展示所有依赖，点击以来冲突依赖之后，点击产生冲突的jar包，右侧会显示具体产生的jar版本

##### 之后再手动去pom修改依赖，具体可以根据三个原则进行依赖冲突解决:

###### a、使用第一声明者优先原则

```xml
  <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>

    </dependencies>


```

依赖都放在dependencies中，谁先定义的就引用谁的依赖，自上而下

###### b、使用路径近者优先原则：即直接依赖级别高于传递依赖。

直接指定具体的依赖的版本：下面项目引入是 spring core 5.2.0的包

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>



    </dependencies>

```

###### c、排除依赖

排除依赖如果是idea，可以使用maven helper插件进行排除。点开pom.xml，切换到Dependency Analyzer视图，选择All Dependencies as Tree，在依赖树图中进行排除，右键jar，直接选择Exclude选项进行删除

产生的pom影响如下：项目引入是 spring core 5.2.0的包，在pom中使用exclusions标签对依赖中的传递依赖进行排除

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.7.RELEASE</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring-core</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>

    </dependencies>


```

###### d、版本锁定（推荐）

使用dependencyManagement 进行版本锁定，dependencyManagement可以统一管理项目的版本号，确保应用的各个项目的依赖和版本一致。

下面可以看到项目引入是 spring core 5.2.0的包

```xml
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <--!>
                <version>5.2.0.RELEASE</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.0.RELEASE</version>
        </dependency>

    </dependencies>


```

## 总结

综上就是maven如何排查依赖冲突以及解决方法，对于排查依赖个人比较推荐使用maven helper插件，至于解决依赖冲突个人推荐使用**版本锁定**的方法，此外dependencyManagement只是声明依赖，并不自动实现引入，因此子项目需要显示的声明需要用的依赖