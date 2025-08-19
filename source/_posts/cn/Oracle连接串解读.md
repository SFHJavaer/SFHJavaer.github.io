---
layout: post
title: Oracle连接串解读
catalog: true
typora-root-url: Oracle连接串解读
author: Futari
date: 2024-07-02 22:25:42
---

我们在常常在SpringBoot配置文件中配置连接数据库，如图:

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724649447720-0031cb9c-616c-47c4-be0a-3c7799f0cb34.png)

首先：<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">Oracle JDBC连接一共有三种方式，分别是：SERVICE_NAME、SID和TNSName。</font>

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724649710075-76104795-58b8-4634-885b-dc7ba9e43c02.png)

<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">我们常常使用的是服务名，只有在旧版本使用SID，原因是一个SID对应一个一个Oracle实例，而服务名是在集群多实例的情况下诞生的，所以一个服务对应多个实例，我们连接串port后的默认就是服务名</font>

:::success
声明： @hostname:port/XXX  ，这个才是简易链接串，前面的jdbc:oracle:thin:为JDBC协议头

:::

对于简易连接串来说，不支持SID的连接方式，需要使用完整连接描述符：

```properties
spring:
  datasource:
    url: jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.100)(PORT=1521))(CONNECT_DATA=(SID=ORCLSID)))
```

@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.1.100)(PORT=1521))(CONNECT_DATA=(SID=ORCLSID)))这个串是完整的连接串而不是简易的，即`@hostname:port/XXX` 格式不能直接指定 SID  、



<h3 id="cKnWN">连接问题</h3>

既然<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">SERVICE_NAME、SID存在区别，为什么同样的用户密码和ip，使用navicat连接时选择Basic连接，不管选择SID还是服务名都能连接成功？</font>

<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">原因是不管什么方式，都连接上了同一实例，</font>

<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">比如在配置Oracle时，</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">SID</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);"> 和 </font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">Service Name</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);"> 可能设置成相同的值，导致不论选择哪种方式，连接都能成功。 ，一般在 在配置文件 </font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">tnsnames.ora</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);"> 中  。</font>

<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">或 Oracle 会为每个实例创建一个默认的服务，这个服务名通常和 </font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);">SID</font>`<font style="color:rgb(0, 0, 0);background-color:rgb(254, 254, 242);"> 一致  。</font>

