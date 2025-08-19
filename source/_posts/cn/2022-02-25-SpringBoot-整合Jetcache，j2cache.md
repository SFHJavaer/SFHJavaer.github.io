---
layout: post
title: 'SpringBoot-整合Jetcache，j2cache'
date: 2022-02-25
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringBoot 
---

## Springboot 整合三方（Jetcache，j2cache）的具体演示

#### 缓存：提高永久存储介质的读取效率，并且提供临时的数据存储空间

可以用HashMap存储数据，不借用数据库，实现类似的缓存机制

SpringBoot提供了多种缓存,常用的有Ehcache，Redis，memcached，不指定默认为simple

###### 在引导类中使用@EnableCaching来启用缓存

###### 在业务层方法上使用@Cacheable(value，key)将方法的return返回值存入到缓存中

> 其中value代表缓存空间的名称，而key则代表具体的存入数据的键，比如方法参数为id，则key可指定为key = "#id"

PS：注意异或运算，比如5和6异或，结果不是一，在计算机中异或运算，是每一个二进制进行异或，而不是整个数，计算机是以二进制进行编码的

~~~
常采用的集中数据淘汰策略：

LRU:挑选最少使用的淘汰

LFU：挑选最近使用最少的数据淘汰

TTL：挑选即将过期的淘汰

RANDOM：随机挑选淘汰


~~~



变更缓存提供商：Redis

~~~
1.加入依赖坐标
2.在xml配置文件中添加redis，因为redis被维护了，所以依赖不需要去mvn仓库加，而且配置文件直接修改配置就可以，所以如果采用xmemcached就需要手动加依赖。并且采用硬编码的方式去实现操作和配置
~~~

###### 加载自定义配置文件参数的方式：

先在xml中编写好具体配置，在配置实体类上加入类注解@Component（交给Spring容器扫描）,@ConfigurationProperties(指定前缀)，@Data,（如果没有set方法的话不能进行配置的注入）,之后直接在其它类中引用实体类的参数就可以间接调用xml中自己设定的参数，而不是每一次都在程序中修改参数。



##### jetcache缓存整合

1.配置远程或者本地缓存的必要属性（在yml中），不然没法初始化，因为SpringBoot没有维护，配置中没有输入提示，格式需要按照jetcache规定的格式手动写入

2.在引导类中开启jetcache注解支持，在Service实现类中声明（自动注入）缓存对象，在方法上添加@CreateCache()注解，括号中为开启缓存的配置

3.操作缓存：使用上面声明的缓存对象操作缓存（对象调用get,put等方法）

> ###### 上面的方法是操作缓存对象的方法，如果要对某一个非缓存对象方法启用具体的缓存调用，使用下面的方法

引导类开启@EnableMethodCache（指定BasePackages包），之后在具体方法上用@Cache（指定属性参数）来配置缓存

###### PS：可以配置方法缓存的刷新，使用@CacheRefresh进行配置，注意这里的注解都可以在括号中携带参数，不然布恩那个进行缓存的参数配置



缓存框架



112.5
