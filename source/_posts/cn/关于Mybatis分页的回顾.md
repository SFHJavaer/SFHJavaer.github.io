---
title: 关于Mybatis分页的详细回顾
catalog: true
date: 2022-04-02 02:34:17
subtitle: 关于Mybatis分页和PageHelper
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- Mybatis
categories:
- Mybatis
---

Mybatis分页有几种方法：

（1）可以直接使用limit语句，在sql语句中使用#{}进行占位，传入的参数调用DAO进行在数据库层面的分页操作

（2）使用mybatis提供的PageHelper第三方插件，通过调用PageHelper的静态方法startPage（current，size），两个参数前面是具体展示第几页，第二个是展示的页码，在Springboot导入依赖并在yml文件中配置，如果是Spring导入的插件则需要在spring配置文件或者mybatis配置文件中书写插件参数，在进行方法调用时先调用静态方法，底层会使用Threadlocal和拦截器，然后执行自己在dao层定义好的selectAll方法，将selectAll查询结果传入PageInfo类的对象中（作为形参），最后返回PageInfo对象即分好页的对象，注意PageInfo对象中有很多的属性，是由分页插件内部计算出来的各种参数

```java
@Override
public PageBean selectPageByExample(E example) throws BaseException {
	PageHelper.startPage(PaginationContext.getPageNum(),
                         PaginationContext.getPageSize());
    List<M> list = getDao().selectByExample(example);
    return new PageBean<M>(list);
}
```



（3）使用mp，直接实现Basemapper接口，调用mp的APi：selectPage（查询页面，有两个形参，这里是IPage对象和queryWapper查询条件），Basemapper提供了众多的基本方法，不需要像原来一样进行自定义，比如selectbyid，updatebyid等等

使用方法：直接调用Dao接口（该接口继承了BaseMapper）注入对象的selectPage方法，该方法传入两个形参，而IPage有是一个接口，所以使用他的实现类对象Page类，Page直接new出来，构造方法传入current和size，接着将page和querywapper传入即可

```java
 /**
     * 利用MP的方式实现分页查询
     * API说明: selectPage(arg1,arg2)
     * arg1:   MP中的分页对象 固定的
     * arg2:   MP分页中的条件构造器
     * @param pageResult
     * @return
     * 动态Sql: select * from user where username like "%admin%"
     * 条件: 如果用户传递query则拼接where条件 否则不拼接where条件
     */
    @Override
    public PageResult getUserList(PageResult pageResult) {//原始3个参数
        //1.定义MP的分页对象 arg1:页面   arg2:行数
        IPage iPage = new Page(pageResult.getPageNum(), pageResult.getPageSize());
        //2.构建查询条件构造器
        QueryWrapper queryWrapper = new QueryWrapper();
        //判断用户数据是否有效  有效 true   无效 false
        boolean flag = StringUtils.hasLength(pageResult.getQuery());
        queryWrapper.like(flag, "username", pageResult.getQuery());
        //经过MP分页查询将所有的分页(total/结果/页面/条数/xxx)数据封装到iPage对象
        iPage = userMapper.selectPage(iPage,queryWrapper);
        //从分页对象中获取总记录数
        long total = iPage.getTotal();
        //从分页对象中获取分页后的结果
        List<User> rows = iPage.getRecords();
        return pageResult.setTotal(total).setRows(rows);    //需要返回的是5个参数
    }


```



然后需要将分页拦截器加入到Mybatis拦截器中才能使分页生效，代码如下：

先创建一个MybatisPlusInterceptor，再将PaginationInnerInterceptor加到其中

别忘了加@Configuration，或者在spring启动类上加入Import制定具体的配置类，不然配置类扫描不到

```java
//命名规则: 类似于配置文件 则把这个类称之为"配置类"一般Config结尾
@Configuration //标识我是一个配置类(代替之前的xml文件)
public class MybatisPlusConfig {
    //铺垫: xml中通过标签管理对象,将对象交给Spring容器管理. <bean>
    //配置类: 将方法的返回值交给Spring容器管理  @Bean注解.

    /**
     * 关于MP分页规则说明
     *   规则: 需要设定一个拦截器.将分页的Sql进行动态的拼接.
     *  Sql: 规则现在的Sql都支持Sql92标准!!!! 设计理念不同
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new 	  PaginationInnerInterceptor(DbType.MARIADB));
        return interceptor;
    }
}

```

