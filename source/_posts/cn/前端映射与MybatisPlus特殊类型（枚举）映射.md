---
layout: post
title: 前端映射与MybatisPlus特殊类型（枚举）映射
catalog: true
typora-root-url: 前端映射与MybatisPlus特殊类型（枚举）映射
author: Futari
date: 2024-04-11 22:26:48
---

我们处理后端Crud时，如果直接将前端的报文应用到数据库动态SQL拼接时，如果传入的RequestDTO某些属性使用枚举接收，那么往往容易报错。

我们知道，这样是可以映射的：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724650982907-c1604268-5e52-47b9-97ac-2df9b919db3b.png)

那么比如前端传入交易状态这个字段，后端是有可以使用的枚举的，直接使用DealStatusEnum接收，会发生什么？

<h4 id="KRXnn">首先，能不能接收到？</h4>

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724650731372-27f7e7ce-268e-418c-8925-e0f6514143ec.png)

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724650752574-7584053f-f401-4b6d-b247-60e470bd7a97.png)

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724650760561-01300a55-2b67-4ba6-b016-075d3728d944.png)

如果枚举如上图所示，invType是可以映射到枚举对象上的，原因是枚举在默认情况下的反序列化，就是直接将前端的字符直接匹配常量名进行映射的，也可以使用如下方式：

`InvestTypeEnum` 枚举也可以通过 `key` 和 `value` 进行映射 ，可以改为

```json
"invType": [
    { "key": "I", "value": "ISSUE" },
    { "key": "T", "value": "FVTPL" }
]

```

但是如果你传入的参数为key-value对应的字符串，那么就不能正常反序列化，接收不到，所以需要使用 @JsonCreator  自定义反序列化器。





<h3 id="Xk3mN">使用</h3>

注意，枚举就是一种特殊的类，所以在XML中使用起来，获取其属性用.，调用方法也要.XXX()

如果我们直接传入枚举到xml，这里的status是一个枚举，那么必然报错，OGNL表达式左边为enum，右边为String，当然类型不一致，所以需要使用 <font style="background-color:#DF2A3F;"> status.name()方法</font>

```xml
 <when test="status == 'OutStanding'">
        <!-- 执行逻辑 -->
</when>
```

:::success
但是 注意status.name()方法，既然调用了方法，就必须先判断其是否为null

:::

然后，我使用以下方式，仍然报错：

```xml
 <when test="invType.name() == 'I'">
        <!-- 执行逻辑 -->
</when>
```

原因是取出的值默认是value，也就是String类型的，但是对于单字符，OGNL会将单引号包括的单字符解析为Char类型（单引号多字符转String不受影响），所以类型不同无法比较。必须改为以下形式，将后方转为字符串

```xml
 <when test="invType.name() == 'I'.toString()">
        <!-- 执行逻辑 -->
</when>
```

或

```xml
 <when test='invType.name() == "I"'>
        <!-- 执行逻辑 -->
</when>
```

```xml
<when test="a != null and a.equals('s')">
    <!-- 执行逻辑 -->
</when>

```

可以看到，实例方法是可以直接.使用的，如果是静态方法

`@类全名@方法名()` 用于调用静态方法。  

```xml
<if test="@java.lang.Math@random() > 0.5">
  <!-- 逻辑 -->
</if>

```









<h3 id="IV69G">扩展</h3>

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724652734495-d2fbbd60-d353-4af7-a49b-b55b8fa79178.png)

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1724652757708-2759a253-778f-410d-8f1c-6eae18add741.png)

