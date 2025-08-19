---
layout: post
title: 'Spring AOP笔记'
date: 2021-11-08
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Java Spring
---

### AOP（面向切面编程）

## 1.动态代理

实现方式：jdk动态代理。使用jdk中的Proxy，Method，InvocationHander创建代理对象

此种要求目标类必须实现接口

cglib动态代理：第三方工具库，通过继承来创建代理对象，继承目标类，创建子类（代理对象），要求目标类不能是final，方法也不能是final，因为final不能被继承或修改

```
动态代理的作用：

1）在源代码不改变的情况下增加功能

2）减少代码重复

3）专注业务逻辑代码

4）解耦合。业务功能和日志，事务和非事务分离
```

<br>

> Aspect：给目标类增加的功能就是切面，比如日志，事务，统计信息，权限验证。

###### 切面的特点：一般都是独立使用的非业务方法。

##### Orient：面向

##### Programming：编程

##### OOP：面向对象编程（同理面向切面以切面为核心）

#### 实现AOP：

``` 
1）分析项目功能时找出切面

2）合理安排切面的执行时间（方法前还是方法后）

3）合理安排切面的执行位置，在哪个类哪个方法中增加增强功能
```



#### 术语：

**Aspect**：切面

**JoinPoint**：连接点，连接业务方法和切面的位置，某类中的业务方法

**Pointcut**：切入点，指多个连接点的集合
**目标对象**：给哪个类的方法增加工功能，这个类是目标对象

**目标方法**：添加功能的方法

**Advice:**通知，通知表示切面功能执行的时间

##### 切面的三个关键要素：

1）切面的具体功能代码，切面干什么

2）切面的执行位置，使用Pointcut表示切面的执行位置

3）切面的执行时间，使用Advice表示时间，在目标方法的前后



> AOP是一个规范，是动态的一个规范化，一个标准

AOP的技术实现框架：

1.spring在内部实现了aop规范。主要在进行事务处理时进行使用aop，项目中很少用

<br>2.aspectJ：开源的aop框架，在spring中集成了aspect



##### aspectJ实现aop的方法有两种：

1）使用xml配置文件配置全局事务

2）使用注解，常用五个注解，第一个是切面的执行时间，在规范中叫做Advice（通知，增强）

``` java
@Before		前置通知

@AfterReturning

@Around

@AfterThrowing

@After
```



aspectJ的切入点表达式（表达式可以加入通配符）

execution(访问权限	方法返回值	方法声明(参数)	异常类型) 				其中的访问权限和异常可以省略

例：execution(public void com.Service.execution(String name))			com包中Service类中的exexution方法

可以使用符号*任意多个字符

​	..方法的任意多个参数			或者当前包和子包

​	+当前类和子类			或接口和实现类



1. `@Component` 注解作用于类，而`@Bean`注解作用于方法
2. `@Component` 注解是通过类路径扫描来自动装配到IOC容器中的，而`@Bean`注解是将方法返回值作为bean装载到IOC容器中
3. `@Bean`注解比`@Component`注解的自定义性更强，而且很多地方我们只能通过`@Bean`注解来注册bean。比如当我们引用第三方库中的类需要装配到`Spring`容器时，则只能通过`@Bean`来实现而不能通过`@Component`注解来实现（其实很好理解：你总不能在第三方类上加`@Component`注解吧）。当然，还有一个注解`@Import`也可以用来把第三方类导入到IOC容器中