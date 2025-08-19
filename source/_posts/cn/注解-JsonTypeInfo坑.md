---
layout: post
title: 注解@JsonTypeInfo坑
catalog: true
typora-root-url: 注解@JsonTypeInfo坑
author: Futari
date: 2023-08-09 18:38:48
---

## 注解@JsonTypeInfo坑

对于多抽象类的多态序列化，一般使用<font style="color:#DF2A3F;">@JsonTypeInfo</font>和<font style="color:#DF2A3F;">@JsonSubTypes</font>配合解决

---

根据传入报文的属性进行反序列化，使用的use = JsonTypeInfo.Id.Name，指定property = "属性名"

@JsonSubTypes中指定name为属性的具体值，对应了其value属性的值为Class类

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME,property = "typeName")  
@JsonSubTypes({@JsonSubTypes.Type(value=Sub1.class,name = "sub1"),@JsonSubTypes.Type(value=Sub2.class,name = "sub2")})  

```

---

注意：

1、这里的@JsonTypeInfo的责任是让Controller知道需要根据哪个属性进行序列化

2、@JsonSubTypes责任是根据name值（一定是报文的，因为是由请求决定的）

来反序列化为value类

3、反序列化根据属性必须在父抽象类和子类中共有，并且子类该属性一定赋上值

4、赋值不是为了给反序列化指定目标类，而是如果不赋值，那么在反序列化后该属性将会被丢弃变为null

---

<font style="color:#DF2A3F;">5、在static项目中，如果“根据属性“含有大写字母，那么在Controller会反序列化成功，但是在通过基架调CloudService时属性丢失的问题还是会产生</font>

<font style="color:#117CEE;">猜测：基架调用jackson存在一定错误，或Jackson本身在对同一对象进行两次反序列化时会产生该bug。</font>

<font style="color:#117CEE;"></font>

<font style="color:#117CEE;"></font>

后期补充：

再调用@RequestBody接收对象时，如果报文里字段为驼峰，如typeTemplate，后端直接同名接还是null，所以会把之前的结论否定掉，和JsonType的序列化无关

jackson的反序列化规则：根据后端来，变量字母一般均为小写，如果出现了大写字母驼峰命名的，那么前端就应当以下划线来间隔单词，如果前端不遵守这个规则，那么后端无法接收到

如果后端也不遵守基本的驼峰命名，那么同样也接受不到

目前猜测Jackson官方是根据规则进行反序列化设计的，<font style="color:#AD1A2B;">后期有待商榷</font>

