---
layout: post
title: 关于对Java语法糖的理解
catalog: true
typora-root-url: 关于对Java语法糖的理解
author: Futari
date: 2025-02-05 22:24:13
---

+ switch 在1.8支持case String类型，<font style="color:#ED740C;">class中其实是根据hashCode和equals判断的</font>，为什么需要equals，<font style="color:#ED740C;">因为仅凭hashCode可能发生哈希碰撞</font>。
+ 泛型，没有泛型对应的class类型，java阶段的泛型都会在class阶段擦除，如HashMap<String,String>会被擦除为Hash Map.Class，而<A extends Comparator<A>>会被擦除为Comparator类型，<font style="color:#ED740C;">也就是都会擦除为最顶级的父类类型</font>   

> 避免编译报错，所以可以理解为什么泛型重载是不能编译的了吧，因为多实例映射到同一Class中，所以泛型，类里的静态变量对多个实例来说是共享的

+ 自动装箱/拆箱，<font style="color:rgb(255, 104, 39);">调用包装器的valueOf方法实现的</font>，都是隐式的
+ 可变入参![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1721962488858-48fd5bfc-8037-455b-a819-fda6bcef7161.png)------>隐式转数组![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1721962500786-16226646-87b7-47f4-8fab-48f15aee0bd4.png)
+ <font style="color:#ED740C;">枚举 ：</font>当我们使用enmu来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类，所以枚举类型不能被继承，还能创建独有的特性，<font style="color:#ED740C;">语法糖就是封装，仅仅是编译时的概念</font>
+ 内部类 ：也就是分别编译和源代码合并（如果把两个class反编译得到的还是一个源文件）
+ 条件编译，也就是有的源代码不会编译，但是Java的条件编译特性很弱，原因是只支持显式的常量条件优化

举例子：其实也可以理解成Java编译器优化

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1721962968148-d39bd8db-0409-4f79-87ae-3fbff0e005ff.png)-![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1721962987573-f053dcb7-ca69-4375-b7e5-912a38af3933.png)

+ 断言 默认Assert是不在编译阶段生效的，可以配置<font style="color:rgba(0, 0, 0, 0.9);">-enableassertions会启用断言检查，断言其实就是if else，满足条件则true，false则会抛出AssertError异常</font>
+ <font style="color:rgba(0, 0, 0, 0.9);">字面值，JDK1.7的时候，数字字面值就可以在数字中任意假的下划线_，编译时会自动去除下划线，目的是为了数字在源代码阶段更直观  10_000 = 10000</font>
+ <font style="color:rgba(0, 0, 0, 0.9);">for-each ：for和迭代器</font>
+ <font style="color:rgba(0, 0, 0, 0.9);">try-with-resources语句，其实就是封装了对try catch finally的资源处理封装</font>
+ <font style="color:#ED740C;">最经典的，Lambda表达式：</font>

<font style="color:rgb(255, 104, 39);">所以，lambda表达式的实现其实是依赖了一些底层的api，在编译阶段，编译器会把lambda表达式进行解糖，转换成调用内部api的方式，</font><font style="color:rgba(0, 0, 0, 0.9);">lambda$main$1和lambda$main$0等等</font>

