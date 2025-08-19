---
layout: post
title: 'SpringBoot表现层消息一致性处理'
date: 2021-12-19
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringBoot
---

对表现层返回的数据封装为json格式,前后端数据传送统一

统一定义为两个字段：data和flag（不管是CORD功能还是查询单挑或多条的功能）

flag代表请求是否成功：比如区分数据不存在还是查询过程抛异常

data代表传送的数据（data中多条数据即为列表json）



解决方案为设置一个前后端数据协议（处理类）

```java
@Data
public class R{
    private Boolean flag;
    private Object data;
}

```

将Controller中所有的mapping方法返回值统一使用自定义的R类，返回R对象

统一后的Controller如下所示：

~~~ java
    @PostMapping
    public R save(@RequestBody Book book){
        return new R(bookService.save(book));
    }
    @PutMapping
    public R update(@RequestBody Book book){
        return new R(bookService.modify(book));
    }
    @DeleteMapping("{id}")
    public R delete(@PathVariable Integer id){
        return new R(bookService.delete(id));
    }
    @GetMapping("{id}")//主路径是books，下级目录为{id}，即http://localhost:8080/books/{id},默认取下级目录的值作为id传入路径变量中
    public R getById(@PathVariable Integer id){
        return new R(true,bookService.getById(id));
    }
~~~

