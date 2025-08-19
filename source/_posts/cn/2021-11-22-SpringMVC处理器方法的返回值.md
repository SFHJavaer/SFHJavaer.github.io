---
layout: post
title: 'SpringMVC处理器方法的返回值'
date: 2021-11-21
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Java SpringMVC
---
### SpringMVC处理器方法的返回值

1）ModelAndView：传递数据并且跳转到其他资源（视图），对视图进行forward，如果有多余的部分则该该返回值不合适
2）String：表示视图，可以逻辑名称，也可以是完整视图路径，逻辑名称需要配置视图解析器，可以直接在方法中return“视图名称”
3）void：多用于响应ajax请求，不需要返回对象，直接前后端分离动态改变



> 处理ajax，使用json作为数据的格式，使用ObjectMapper把结果对象转换为json格式的数据如下：
> （这里使用框架的jackson驱动去完成）
``` java
String json = "";
if (object !=null){
	ObjectMapper om = new ObjectMapper();
	json = om.writeValueAsString(object);
}
```
> 在respnse响应作用域中设置Content-Type
``` java
response.setContentType = "application/json;charset = utf-8"
```
> 最后输出ajax响应( java.io.PrintWriter 具有自动行刷新的缓冲字符输出流)
``` java
PrintWriter pw = response.getwriter();//获取响应的字符流对象，因为我们要在响应作用域中添加数据返回给用户
pw.println(json);//将json数据打印到response的流对象中
pw.flush();
pw.close();//关闭流通道
```
###### 注意：上面的实现步骤中有很多是固定流程，所以交给框架进行处理

4)对象Object，返回Object的属性就是数据，所以返回Object和视图无关，也可以返回一个List的对象集合，去进行遍历等等操作
做ajax主要使用json格式的数据,基本的实现步骤：

1）加入json工具库依赖，springMvc默认使用jackson
2）spring配置文件加入<mvc:annotation-driven>注解驱动(完成java对象到指定对象数据格式的转换)
HttpMessageConveter接口：消息转换器，返回值是一个ArrayList
在注解驱动加入配置文件中时，会自动导入该接口的7个实现类对象，
3）处理器方法上加入@ResponseBoby（代替Print）

处理流程：框架返回对象类型，调用接口中的canWrite()检查是否能进行对象到数据的转换
如果可以则调用write()进行转换
最后使用@ResponseBoby把数据结果输出到浏览器上

###### PS：String作为处理器方法的返回值可以返回数据，区分返回值是数据还是视图，看有没有@ResponseBoby
有注解就是数据，反之默认不加为视图

