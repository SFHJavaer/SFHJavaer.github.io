---
layout: post
title: 'SpringBoot运维测试和配置问题'
date: 2022-01-29
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringBoot
---

application配置文件优先级：properties > yml >yaml

配置文件所在目录所决定的配置优先级要大于本身文件属性决定的优先级

环境简写：

设置：pro

开发：dev

测试：test



```  java

SpringBootApplication扫描注解的默认规则是：

扫描@SpringBootApplication注解所在类（引导类）所在的包和其子包中的注解

BUG注意：这里如果引导类所在的是二级包（com.asb），那么可以扫描到注解

如果是直接在一级包下，则扫描不到bean相关注解
```







##### 运维篇：多环境开发的注意

根据功能对配置文件的信息进行拆分，制作成独立的配置文件,如：

application-devDB.yml

application-devRedis.yml

application-devMVC.yml

> 使用active属性进行环境的配置选择，使用include属性在激活指定环境的情况下，同时对多个环境进行加载使其生效，多环境用分隔：

```yml
spring: 

 profiles: 

  	active: dev

 		include: devDB,devRedis,.....................


```

###### 				





#### SpringBoot日志（log）：

在类中创建日志对象（Logger类）：

###### 日志对象的六个操作，四个最常用：debug,info,warn,error

在配置文件里设置日志的输出级别

使用@lombok提供的@Slf4j注解快速开发，减少日志对象的声明

（下面这里没看懂，String.format和sout都可以使用占位符，但是网上的logger的占位符是{},和c的占位差不多，但是占位符始终是{}）
为什么logger要启动占位符，原因是直接进行+拼接日志，就会在字符串池重新创建对象，而使用占位符提升了速度并且减少了内存开支。

设置日志输出格式：百分号为占位符，而且必须写入到双引号里作为字符串进行解析

%d:日期

%m:消息

%n:换行

pattern：模板，日志也可以进行输出格式控制





#### 热部署

添加spring-boot-devtools依赖坐标

手动部署需要构建项目进行激活热部署Ctrl+F9，代表restart过程而不是reload，不加载jar的资源

自动build，在setting中进行设置

> 也可以在配置文件中设置热部署的范围，注意static的前端页面是不需要热部署的，因为在网页上是直接请求资源文件的，服务器上修改资源文件请求的也会直接改变（注意清除缓存）



也可以通过设置优先级属性修改配置文件来禁用热部署（官网有属性加载优先顺序）





#### 第三方bean属性绑定

@EnabledConfigurationProperties与@Component不能同时使用

因为前者会自动声明一个对应的类的bean对象，所以同时有的话会导致bena对象定位异常

一般使用@ConfigurationProperties(prefix = "配置文件属性名")来将配置文件中的属性注入到类中



##### 松散绑定

@ConfigurationProperties绑定属性支持属性名松散绑定

但是要注意绑定的前缀名仅能用纯小写字母、数字、下划线等作为合法字符

一般最标准的命名格式为烤肉串型，即aa-bb式

###### @Value不支持松散绑定



JDK8提供了时间与空间的计量单位

Duiation类：时间类

Datasize类：空间（大小）类

上面都可以通过添加注解的方式进行指定类中的数据所要使用的单位



为了避免yml属性名输错，可以使用属性校验

添加JSr303和Hibernate校验框架的坐标

使用@Validated开启Bean的校验功能，可以设具体的校验规则@Max@Min等等

类似于JDBC是接口，而mysql驱动是实现类，同理引入的校验框架也是接口，直接用注解省略了手动创建实现类的步骤



###### 注意yaml中的对于数字的书写格式，使用纯字符串监建议用双引号进行包裹，不然可能产生自动的进制转换等，出现错误要自己进行测试找字符串进行了何种转换变成了什么



####  测试

使用@import注解加载当前测试类专用的配置

@import(xxx.class)



@SpringBootTest有很多属性，比如设置web启动环境（指定默认端口或者随机端口）或者启动参数





开启虚拟Mvc调用：@AutoCOnfiguraeMovkMvc

注入虚拟MVC对象：@Autowired

创建虚拟请求访问指定的测试页面（MockHttpServletRequestBuilder接口）

执行虚拟请求：对象.perform

###### 匹配响应执行状态，使用MockMvcResultMatchers获取其属性作为对象，和虚拟MVC对象进行比对（比对什么由方法决定），返回测试结果，这里的比对有很多，那比如响应状态，响应头，响应体等等

> 因为测试删改等会对数据库中产生影响，所以需要启用事务回滚功能

> 为测试用例添加事务，SpringBoot默认自动对事务提交操作进行回滚

> 如果不需要回滚操作，那么就要用@Rollback(flase)取消自动回滚































































0 0 2 * * ? python3 /root/疫情上报.py
