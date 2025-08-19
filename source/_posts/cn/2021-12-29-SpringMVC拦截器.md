---
layout: post
title: 'SpringMVC拦截器'
date: 2021-12-29
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringMVC
---

springmvc使用的是统一的全局的异常处理机制

把crontroller中的所有异常集中处理，采用aop的思想

---

使用两个注解：

@ExpextionHandler：使用在具体的方法上，作为异常处理crontroller，value属性用来指定异常的类型属于哪个类，如果不写则代表全局处理异常

@ControllerAdvice：控制器增强（即给控制器增加异常处理功能），在类上添加，当然要添加组件扫描器使程序扫描到异常处理类所在的包

异常处理发生逻辑：

1）将异常记录（到数据库），日志文件

记录日志发生的时间，发生异常的具体方法，异常错误内容

2）通过某种方式发送通知

3)给用户一定提示

异常处理方法和crontroller方法一样，同理可以有多个参数，ModelAndView，String等等

形参Expection表示crontroller中抛出的异常对象，由它可以获取异常信息

---

拦截器：

1）定义类实现HandlerInteceptor接口

2）在mvc配置文件中声明拦截器

拦截器一般用在用户登录处理、权限检查、记录日志等

拦截器是全局的，可以对多个crontroller进行拦截，过滤器是用来过滤请求参数，设置编码字符集等等

---

##### Controller和Handler有什么区别

一般来说，Controller是Handler，但Handler不一定是Controller。

例如，HttpRequestHandler，WebRequestHandler，MessageHandler都是可以使用DispatcherServlet的处理程序。 （（@）Controller是执行Web请求并返回视图的处理程序。）


简而言之，Handler只是一个术语，它不是一个类或接口。 它可以执行映射。

---

##### 方法上加@ResponseBody

加了代表返回数据按照返回值类型直接放到响应体中，和请求时放入请求体是相对的，然后方法从响应体中取，一般是Json，而不加的话就会返回ModelAndView给DispatcherServlet，最后处理完视图进行直接的展示，所以对于前后端分离的业务我们都是加@RestController而不是不加。
