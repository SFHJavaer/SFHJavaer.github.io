---
layout: post
title: Serializable接口的实际意义
catalog: true
typora-root-url: Serializable接口的实际意义
author: Futari
date: 2024-03-22 22:27:34
---

在项目中我们看到实体类都实现了Serializable接口，为什么有的时候不实现也可以正常执行、返回报文呢？



原因在于该对象需不需要进行序列化（或者更准确的来说是     ：       非本地传输）

> 注意：这里的序列化 ≠ 转Json，如果通过MVC的@RequestBody，那么会先自动把对象转为Json（String），再通过网络发送（此时是JDK自带序列化），先转成的本地Json其实还是字符串
>
> 而String默认是实现序列化接口的，所以它能够返回到前端



+ **writeObject这些才是真正序列化操作，被操作对象一定要实现Serializable接口**
+ **<font style="color:rgb(77, 77, 77);">ObjectMapper#writeValueAsString只是对象转换成json，与Serializable无关。</font>**

> 引用文章：[https://blog.csdn.net/qq_25135655/article/details/94602975](https://blog.csdn.net/qq_25135655/article/details/94602975)

不管是内部序列化还是进行外部传输（比如RPC远调），即使serializable是一个空接口，实现它代表该类可以<font style="color:#DF2A3F;">以JDK自带的序列化方式进行解析并进行序列化</font>

如果项目Controller统一返回MVC Json，那么既不实现Serializable也不使用@JonGenerator自定义序列化器只有项目存在对本对象手动序列化操作时才会报错



<font style="color:#DF2A3F;">（所以不管对象多么复杂，如果不在内部手动序列化，直接通过MVC返回Json，那么都可以将其看成一个String，都无需实现什么序列化接口）</font>

<h3 id="Pf2GP">为什么入库不需要实现？</h3>

基本entity都不需要实现，原因是Entity在规范上不需要进行外部传输，入库本质上是将数据通过SQL的方式插入到数据库中，并不是通过将原对象进行序列化来实现持久化的







今天就碰到了这个问题，交易大对象中的一个小引用对象没有实现Serializable接口

![](https://cdn.nlark.com/yuque/0/2025/png/35478834/1736500162480-c4339a3c-554b-4f05-9457-3f787b9f15e1.png)

可以看到这个其实还是内部基架处理的专门针对交易对象的序列化，所以不实现Serializable此时抛出：

NoSerializableException异常--------> :没实现序列化接口

