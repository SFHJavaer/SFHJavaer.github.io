---
layout: post
title: \@Autowired同名Bean问题修复
catalog: true
typora-root-url: \@Autowired同名Bean问题修复
author: Futari
date: 2023-09-21 22:08:03
---

### @Autowired同名Bean问题修复

新建一个SpringDemo项目，注意先修改好发行版，出错都是因为出场Setting没设置对，以及自动生成SpringBoot项目时，xml中指定了<java.version>，要将其去掉或修改为8，不然import Maven又会使设置生效。

新建两个不同包下的同名类Test，类结构如图：

![a](https://cdn.nlark.com/yuque/0/2023/png/35478834/1695288946165-9654b0a6-5e79-4c05-a32e-10cb2edc68da.png)

Application：

![a](https://cdn.nlark.com/yuque/0/2023/png/35478834/1695289149568-4acc6196-b5aa-4160-8bde-7863fc740010.png)

Real类：![a](https://cdn.nlark.com/yuque/0/2023/png/35478834/1695289158611-edeae84b-0d4c-4d31-9edc-2ce7711efb56.png)

这里不把@autowired写到Application中是因为写了也不会注入，SpringBoot会把Application的注入忽略，

所以把@Autowired写到了Real类中，new去使用它。

<font style="background-color:#FBDE28;">以上图结构run,返回以下结果，很容易看出是在Bean产生了冲突：</font>

```java
2023-09-21 17:42:01.596  INFO 28712 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 1.8.0_333 on DESKTOP-7TNR5U5 with PID 28712 (D:\IdeaProjects\demo\demo2\target\classes started by hspcadmin in D:\IdeaProjects\demo)
2023-09-21 17:42:01.601  INFO 28712 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2023-09-21 17:42:01.847  WARN 28712 --- [           main] ConfigServletWebServerApplicationContext : Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanDefinitionStoreException: Failed to parse configuration class [com.example.demo.DemoApplication]; nested exception is org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name 'test' for bean class [com.example.demo.test2.Test] conflicts with existing, non-compatible bean definition of same name and class [com.example.demo.test.Test]
2023-09-21 17:42:06.009 ERROR 28712 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.BeanDefinitionStoreException: Failed to parse configuration class [com.example.demo.DemoApplication]; nested exception is org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name 'test' for bean class [com.example.demo.test2.Test] conflicts with existing, non-compatible bean definition of same name and class [com.example.demo.test.Test]
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:188) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.processConfigBeanDefinitions(ConfigurationClassPostProcessor.java:331) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(ConfigurationClassPostProcessor.java:247) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanDefinitionRegistryPostProcessors(PostProcessorRegistrationDelegate.java:311) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.support.PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(PostProcessorRegistrationDelegate.java:112) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.support.AbstractApplicationContext.invokeBeanFactoryPostProcessors(AbstractApplicationContext.java:746) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:564) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:147) ~[spring-boot-2.7.6.jar:2.7.6]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:731) [spring-boot-2.7.6.jar:2.7.6]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:408) [spring-boot-2.7.6.jar:2.7.6]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:307) [spring-boot-2.7.6.jar:2.7.6]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1303) [spring-boot-2.7.6.jar:2.7.6]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1292) [spring-boot-2.7.6.jar:2.7.6]
	at com.example.demo.DemoApplication.main(DemoApplication.java:11) [classes/:na]
Caused by: org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name 'test' for bean class [com.example.demo.test2.Test] conflicts with existing, non-compatible bean definition of same name and class [com.example.demo.test.Test]
	at org.springframework.context.annotation.ClassPathBeanDefinitionScanner.checkCandidate(ClassPathBeanDefinitionScanner.java:349) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ClassPathBeanDefinitionScanner.doScan(ClassPathBeanDefinitionScanner.java:287) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ComponentScanAnnotationParser.parse(ComponentScanAnnotationParser.java:128) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ConfigurationClassParser.doProcessConfigurationClass(ConfigurationClassParser.java:295) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ConfigurationClassParser.processConfigurationClass(ConfigurationClassParser.java:249) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:206) ~[spring-context-5.3.24.jar:5.3.24]
	at org.springframework.context.annotation.ConfigurationClassParser.parse(ConfigurationClassParser.java:174) ~[spring-context-5.3.24.jar:5.3.24]
	... 13 common frames omitted

与目标 VM 断开连接, 地址为: ''127.0.0.1:56835'，传输: '套接字''
```

<font style="color:#01B2BC;">如何判断是@Autowired注入时冲突还是容器都放不进去？</font>

<font style="background-color:#FBDE28;">注释掉@Autowired，发现仍然报该错，说明Factory不允许创建同类名Bean不加以区分</font>



现在创建一个子项目，作为demo的依赖模块的demo2，并将test2的类移动到demo2模块下，且添加两模块间的依赖关系，如下图：

![a](https://cdn.nlark.com/yuque/0/2023/png/35478834/1695289684290-a406a39a-a7ab-40c6-bad9-1904f1d5d29b.png)

<font style="color:#5C8D07;">不加</font><font style="color:#5C8D07;">执行结果：</font>

```java
2023-09-21 17:49:31.990  INFO 34544 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 1.8.0_333 on DESKTOP-7TNR5U5 with PID 34544 (D:\IdeaProjects\demo\demo2\target\classes started by hspcadmin in D:\IdeaProjects\demo)
2023-09-21 17:49:31.995  INFO 34544 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2023-09-21 17:49:32.652  INFO 34544 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-09-21 17:49:32.658  INFO 34544 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-09-21 17:49:32.658  INFO 34544 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.69]
2023-09-21 17:49:32.761  INFO 34544 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-09-21 17:49:32.761  INFO 34544 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 720 ms
2023-09-21 17:49:33.020  INFO 34544 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-09-21 17:49:33.027  INFO 34544 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.401 seconds (JVM running for 2.902)
```

<font style="color:#DF2A3F;">发现并没有报错！说明Factory的错没有报出，但不知道此时bean，添加打印BeanName</font>

```java
String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            System.out.println(beanDefinitionName);
        }
```

![](https://cdn.nlark.com/yuque/0/2023/png/35478834/1695290435435-c4556cf2-d2f7-4eb7-bb05-977ff9cc9323.png)

可以看到确实有一个test对象，但是不知道是test1的还是test2，换个方法ac.getBeanNamesForType(Test.class);注意Test导入的路径要不同

<font style="color:#DF2A3F;">结果发现：</font>

<font style="color:#DF2A3F;">test为applicaton本模块的Test类，而子模块没有注入</font>

<font style="background-color:#F1A2AB;">加@Autowired调用一下看能否正常用，发现报了空指针，说明没注入进来，Factory中有，但是却没注入，很奇怪：</font>

![](https://cdn.nlark.com/yuque/0/2023/png/35478834/1695290877248-fcdd1212-61f4-4397-be83-a80c1e422d5f.png)

**最后可以得出结论。**

**对于正常的同名Bean，会直接报错，但是如果存在依赖关系，那么可能会导致报错被吞，并且有时候可能会直接断开VM连接。**

**而如果已经出现了同名错误，但是没报出，@Autowired此时是失效的了，可以直接理解成多Component无法注入，虽然此时Factory中有对象，但是由于有错在先，仍不允许注入Bean**

**在出现同名Bean的情况下就不要考虑注入规则ByType/ByName了，因为有错在先，可以理解成Spring未考虑的问题。**