---
layout: post
title: Optional基础使用
catalog: true
typora-root-url: Optional基础使用
author: Futari
date: 2023-10-04 22:11:19
---

Optional专治俄罗斯套娃的if判断，比如链式获取某个对象的属性的属性，由于对象的属性可能为空，所以会进行套娃判断null

```java
public String getStreetName( Province province ) {
    return Optional.ofNullable( province )
            .map( i -> i.getCity() )
            .map( i -> i.getDistrict() )
            .map( i -> i.getStreet() )
            .map( i -> i.getName() )
            .orElse( "未找到该道路名" );
}

PS：如果前面的map映射失败，或者最后一个map完没有return，也就是最后的orElse就会执行，同义如下：

public String getStreetName( Province province ) {
    if( province != null ) {
        City city = province.getCity();
        if( city != null ) {
            District district = city.getDistrict();
            if( district != null ) {
                Street street = district.getStreet();
                if( street != null ) {
                    return street.getName();
                }
            }
        }
    }
    return "未找到该道路名";
}
PS：orElse或者orElseGet都是针对于ofNullable是否为空来进行的，为空执行orElse
区别是orElseGet可以传入一个接口，来定制逻辑。orElseThrow可以扔lambda异常表达式
```

**重要：isPresent也支持lambda，如果nullable里不为空再执行isPresent中的apply操作。**

所以可以把if判断和后续操作都放到一行代码中，外层if正好用<font style="color:rgb(221, 74, 104);background-color:rgb(250, 250, 250);">orElseGet扔入处理逻辑lambda</font>解决掉。



<font style="background-color:#FBDE28;">orElse和orElseGet这些都是链式调用的，所以是一定会执行的，关键是在于是不是满足这个条件，不满足else当然就返回里面的lambda了，如果不满足else就相当于这个lambda没生效了，直接返回原来的T，并且注意返回的是T，这里直接返回的T当然也可能是null的。</font>

