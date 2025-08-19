---
layout: post
title: IDEA之Bean报红问题的分析与解决
catalog: true
typora-root-url: IDEA之Bean报红问题的分析与解决
author: Futari
date: 2023-02-12 21:57:41
---

##### 这是一个很细节的问题，一般情况不会注意，其实要理解这种现象为什么会产生。

现象如下图：

![报错](a068de340a224a43a42b113a0782e01b.png)



比如作为Dao的Bean爆红线但是可以正常注入运行，我们之前知道了使用@Resource解决，原因就是@Autowired是Spring提供的，他只能检测到Spring本身的类，如果你使用Mybatis而不对Mapper类加上@Dao注解

###### 那么其实就是没有将类加入到容器中，所以使用JDK提供的@Resource，就相当于检查不出来了，尤其不推荐在Setting关闭检查，也可以使用@RequiredAutowired忽略参数检查，当然这样如果Bean为空就会在运行时报错了。

这里涉及到一个注解@RequiedArgsConstructor，该注解最主要的的作用是将加final修饰的属性作为Bean进行IOC注入，注意这是一个Lombok提供的注解，使用该注解默认使用构造器注入，这也是Spring推荐的注入方式，直接不需要手写有参构造方法。

> @RequiedArgsConstructor适合一个类中调用很多的bean，每一个bean都使用@Autowired修饰很麻烦。



#### 那么为什么4.0之后推荐使用构造器注入的方式？而不是Autowired注解注入或者setter注入？

原因是@Autowired可能会产生循环依赖并且多Bean注入不美观。

构造器注入可以保证完全初始化，保证这个注入的Bean一定不是空的，因为调用有参构造不传入参数会报错，Bean加final修饰保证Bean的不可变，只需要写一个有参构造就可以了，注意被注入的属性也就是Bean，是把Bean注入到了当前类。

Setter同样在很多属性的时候，会显得非常的冗余写很多的构造器。

