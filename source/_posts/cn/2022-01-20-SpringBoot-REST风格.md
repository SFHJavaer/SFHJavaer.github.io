---
layout: post
title: 'SpringBoot RESTful风格'
date: 2022-01-20
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringBoot RESTFUL
---

## Springboot RESTFUL风格

RESTful风格的url的优点：

1）隐藏资源的访问行为，无法通过链接地址判断对资源的操作

2）书写简化

REST是一种风格不是规范，或是现在已经约定俗成的规范

描述模块的名称通常使用复数加s，表示此类资源而不是单个资源



按照REST访问资源时使用行为动作来区分进行何种操作（一般是8种请求方式），这种访问方式叫RESTful

实现步骤：1）设定http请求动作（RequestMapping中指定value和method）

2）设置请求参数（路径变量），value的路径上占位符参数要和方法的形参一致，形参上加上@PathVariable，将路径中的占位符的值当作形参变量传入方法中



### 方法形参的注解：

#### 重点：

@RequestParam:接收url地址传参或表单传参

@RequestBoby：接收Json数据

@PathVariable：接收路径参数

后期应用：当发送参数不超过一个时使用@PathVariable，超过一个使用Json格式，即使用@RequestBoby，少数情况非json的数据使用@RequestParam

##### 所以原来的注解是写在方法上或方法内的，可以直接移动到类的上面来使所有的controller生效

所以在类上的@Controller和@Response可合并成为@RestController

方法动作的指定:将RequestMapping(method = .....)简化为形如@PostMapping



Springboot配置文件加载顺序：

properties > yml >yaml

常用的配置文件是yml格式，当多个配置文件共存时，相同的配置按优先级，不同的配置都会生效

配置文件自动提示消失:说明idea没有把该文件识别为配置文件，需要在工程结构中Facets手动添加配置文件（可能会添加的bug），注意Idea图标代表的文件种类



yml的语法规则不用记忆，记住核心规则：数据前面要加空格与冒号隔开，yml以数据为核心

@Value注解将外在值作为参数传入

将-后面的整体当作一个独立的对象，可直接使用字段.对象来取，同样可使用字段.对象.属性来取得对象的属性，即${一级属性名.二级属性名}（引用其他变量（属性））

yml是支持转义字符的，不过需要对属性加引号将它括起来才会生效

