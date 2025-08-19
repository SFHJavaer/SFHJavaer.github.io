---
layout: post
title: AOP使用经验和基础知识总结
catalog: true
typora-root-url: AOP使用经验和基础知识总结
author: Futari
date: 2023-12-18 22:15:13
---

很简单的，五种通知类型，主要注意的是环绕通知，也就是符合切入点表达式的切入点，会根据通知内容即自定义方法自定义执行。

切面类中写着通知方法，通知方法上用注解标明其类型，当被执行的切点符合注解的表达式范围时，执行通知。

不光是记录前后日志，在执行动作前后校验权限，执行数据库操作（触发器）等等都可以使用AOP。

执行条件的限定主要取决于切入点表达式

---

1. **<font style="color:rgb(51, 51, 51);">execution：一般用于指定方法的执行，用的最多。</font>**
2. <font style="color:rgb(51, 51, 51);">within：指定某些类型的全部方法执行，也可用来指定一个包。</font>
3. <font style="color:rgb(51, 51, 51);">this：Spring Aop是基于动态代理的，生成的bean也是一个代理对象，this就是这个代理对象，当这个对象可以转换为指定的类型时，对应的切入点就是它了，Spring Aop将生效。</font>
4. <font style="color:rgb(51, 51, 51);">target：当被代理的对象可以转换为指定的类型时，对应的切入点就是它了，Spring Aop将生效。</font>
5. <font style="color:rgb(51, 51, 51);">args：当执行的方法的参数是指定类型时生效。</font>
6. <font style="color:rgb(51, 51, 51);">@target：当代理的目标对象上拥有指定的注解时生效。</font>
7. <font style="color:rgb(51, 51, 51);">@args：当执行的方法参数类型上拥有指定的注解时生效。</font>
8. <font style="color:rgb(51, 51, 51);">@within：与@target类似，看官方文档和网上的说法都是@within只需要目标对象的类或者父类上有指定的注解，则@within会生效，而@target则是必须是目标对象的类上有指定的注解。而根据笔者的测试这两者都是只要目标类或父类上有指定的注解即可。</font>
9. <font style="color:rgb(51, 51, 51);">@annotation：当执行的方法上拥有指定的注解时生效。</font>
10. <font style="color:rgb(10, 191, 91);background-color:rgb(243, 245, 249);">reference pointcut</font><font style="color:rgb(51, 51, 51);">：(经常使用)表示引用其他命名切入点，只有@ApectJ风格支持，Schema风格不支持</font>
11. <font style="color:rgb(10, 191, 91);background-color:rgb(243, 245, 249);">bean：当调用的方法是指定的bean的方法时生效。(Spring AOP自己扩展支持的)</font>

---

AOP切点的接口叫JoinPoint，主要参数是

+ name: 方法名
+ args：方法参数
+ signature：方法签名

---



注意方法签名：是方法的唯一标识，尤其是对于重载方法的区分，一个方法的方法签名如下

**<font style="color:rgb(77, 77, 77);">全类名.方法名(形参数据类型列表)返回值数据类型</font>**

**<font style="color:rgb(77, 77, 77);">举例(javap -s 生成方法签名)：</font>**

```java
$ javap -s java.lang.String  
Compiled from "String.java"  
public final class java.lang.String extends java.lang.Object implements java.io.Serializable,java.lang.Comparable,java.lang.CharSequence{  
public static final java.util.Comparator CASE_INSENSITIVE_ORDER;  
  Signature: Ljava/util/Comparator;  
public java.lang.String();  
  Signature: ()V  
public java.lang.String(java.lang.String);  
  Signature: (Ljava/lang/String;)V  
public java.lang.String(char[]);  
  Signature: ([C)V  
public java.lang.String(char[], int, int);  
  Signature: ([CII)V  
public java.lang.String(int[], int, int);  
  Signature: ([III)V  

```



方法参数：

```java
//获取方法参数(返回值为数组)
Object[] args = joinPoint.getArgs();
```





异常通知举例：

```java
 @AfterThrowing(value = "execution( public int com.aop.service.Mycalculator.*(int,int))",throwing = "e")
  public static void logException(JoinPoint joinPoint,Exception e) {
      // 获取方法签名
      Signature signature = joinPoint.getSignature();
      // 获取方法名
      String name = signature.getName();
    System.out.println(name+"方法出现异常--AfterThrowing,异常信息"+e);
  }


```

表达式的 修饰符和返回值不重要，原因是方法签名的唯一性不取决于其，只取决于方法参数（重载要素）



其实相当于注解方式的AOP，XML方式配置并不更麻烦，而且更加直观，可读性增加。



<h2 id="rGz4d">代理与AOP</h2>

使用到底层使用切面或拦截实现的工具时，比如事务（数据库），其底层都是使用的AOP

实际上简化了手动创建与关闭、管理事务的过程，用到了AOP其实就是用到了代理，很容易理解，因为代理的本质就是完成了前后置的处理。

**当然牢记：代理的中间调用一定是调用被代理对象的方法。**

这就是方法内部进行调用时事务为什么失效，

比如@Transactional放到方法a上，不管是a调用b（比如insert操作）方法还是b调a方法，都会失效

原因是：如果是第一种，a没有持久化操作，其实就相当于事务前置处理---->invokeA()，在原实例的a方法中调this.b()------->事务后置处理

所以ProxyA方法根本不可能直接调到b方法，此时的在生成的代理类的字节码中，ProxyB才有super.b()，代理类生成的ProxyB是直接调的被代理类的b，但是没有调到Proxy的B，就不会有事务控制了！！！（super指的就是原对象）所以属于b的事务失效。

**<font style="color:#DF2A3F;">所以关键是调用方是不是Proxy！！！会不会失效，看自调用方法的的调用方是不是Proxy！！！</font>**

**第二种：**

b方法本身就没加入事务控制，所以调b直接调的被代理类的this.a，被代理类无法执行事务（实例调用，相当于单纯方法），所以事务失效。

:::success
所以如果想要不失效，只需要在原逻辑中别使用this自调用，使用代理自调用（直接在@autowired注入本Bean（因为单例，所以该类一定是代理对象）或从AopContext获取代理对象），这样就相当于让自调用纳入Spring事务管理，事务管理就会生效。（即使ProxyB直接调的super.b()，什么操作都没做，但是他还是纳入到了事务管理的流程，经过了事务拦截器的处理）

:::





例子

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class MyService {

    @Transactional
    public void a() {
        System.out.println("Executing method a");
        b();
    }

    public void b() {
        System.out.println("Executing method b");
    }
}

```

```java
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

public class MyService$$EnhancerByCGLIB$\$123456 extends MyService {
    
    private MethodInterceptor methodInterceptor;

    public MyService$$EnhancerByCGLIB$\$123456(MethodInterceptor methodInterceptor) {
        this.methodInterceptor = methodInterceptor;
    }

    @Override
    public void a() {
        try {
            // 调用MethodInterceptor来处理事务逻辑
            methodInterceptor.intercept(this, MyService.class.getMethod("a"), new Object[0], MethodProxy.create(MyService.class, this.getClass(), "void a()", "a", "a"));
        } catch (Throwable t) {
            throw new RuntimeException(t);
        }
    }

    @Override
    public void b() {
        // 调用父类的方法，且b没有事务增强是最关键的！
        super.b();
    }
}
```

不管是通过继承还是实现来代理的，代理类的a方法除了invokeA之外，和b没有任何关系！！！

即使a方法调了b，那也是原方法调的，代理方法只专注于a，代理代的就是invoke原方法，不是将原方法加几个$符号逻辑一模一样生成的就是代理，代理不是复制逻辑，而是invoke整个原逻辑！！！！



:::success
<font style="color:rgb(55, 83, 117);background-color:rgb(247, 250, 255);">通过Spring的代理对象进行调用的，才能触发事务管理，才能生效，离开了Spring就是纯Java！！！</font>

:::

<h3 id="hBJf0">Bean配置</h3>

那么对于@Configuration注解，我们已知配置类会生成Bean（代理）

那么如果里面写两个@Bean，a方法中自调用b()后再return new A()对象，那么此时脱离范围，会不会使得b的@bean失效，这样也许不准确，因为两个都加了注解，浅层理解也是可以猜对的

<h5 id="BDY3K">但是深入思考，你说b这个 @Bean方法会被创建（执行）几次呢？执行过程呢？</h5>

configuration代理类顺序创建Bean，先构造a时，前置处理完，invoke实例的a方法，这里接下来调的是this.b(),所以第一条打印的是实例执行的b方法，invoke完之后，proxya的后置处理完成，打印a语句。

**接下来就结束了。。。。为什么**

涉及到启动时Bean的注册流程，虽然是在Configuration实例的this.b(),但是创建调用b方法时，b的方法信息表明其是一个Bean，应当被Spring所管理，所以会去先检测Bean是否注册存在，不存在就注册，所以后续再调用发现已注册就都不会再创建Bean了，所以也保证单例

