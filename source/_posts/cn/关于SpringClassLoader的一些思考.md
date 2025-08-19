---
title: 关于SpringClassLoader的一些思考
catalog: true
date: 2022-09-03 02:34:17
subtitle: 关于SpringClassLoader的一些思考
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- Spring
categories:
- Spring
---

### 关于SpringClassLoader的一些思考

###### 看了Tomcat我以为只要没打破双亲委派机制的就是jvm的类加载机制appclassLoader，其实是没关系的

###### 看了spring之后我觉得springboot肯定是自定义加载器，刚才试了几个bean类获取了下发现是AppClassLoader

###### 看springboot自动装配源码发现在扫factory文件的时候使用的是SpringFactoriesLoader这个自定义加载器，里面是对后面自动装配的properties类的的加载和解析，仔细一看却不是自定义的classloader（没继承）

> 严格按照继承体系来说只能算是appclassloader的引用类加载器，从网上找的那种说法，说明在项目以jar的方式运行的时候，springboot内部会使用JarLauncher入口的LaunchedURLClassLoader，说明是有自定义类加载器
>
> 内置tomcat加载的时候也是自定义的，所以就说主要的成员类都是Appclassloader加载的，对于特殊场景下才用springboot自定义的类加载器

![img](https://s1.328888.xyz/2022/10/11/gZgvy.png)