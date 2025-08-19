---
layout: post
title: 前后端Controller--接收特殊类型参数问题
catalog: true
typora-root-url: 前后端Controller--接收特殊类型参数问题
author: Futari
date: 2024-01-21 22:20:23
---

<h3 id="C4SpR">数组</h3>

今天在前端发送某个属性值为[]时，后端接收到的为空的ArrayList对象。

首先清楚，在MVC框架下，后端接收到什么参数，完全取决于序列化框架，原因是参数对象一定是反序列化变为对象的，而对于大多数序列化框架，规则基本是一致的，所以建议记忆。

这里Jackson的序列化规则就是[],最后被反序列化为ArrayList而不是null;





![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1721096702451-9d356ac3-64e3-42c7-b735-14353996908e.png)

这些接收到的都是ArayList



<h3 id="TeNry">枚举</h3>

对于前端传入一个字符串，后台可以直接使用enum接收吗

**答案是分条件**

**如果如下图所示是可以的，如第二张图是不可以的**

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724291831239-fce4b4c0-ed86-4d42-9cc9-2a22c390d0b5.png)

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724291861460-8b61fef1-1d48-4768-ae0b-95a567b1ffcc.png)

