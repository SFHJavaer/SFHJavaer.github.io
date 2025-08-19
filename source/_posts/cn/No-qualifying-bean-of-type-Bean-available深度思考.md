---
layout: post
title: No qualifying bean of type 'Bean' available深度思考
catalog: true
typora-root-url: No qualifying bean of type 'Bean' available深度思考
author: Futari
date: 2024-11-18 22:25:03
---



在多工程项目中，我们常常遇到这个问题。

 <font style="color:#DF2A3F;">No qualifying bean of type 'XXXXBean' available  ，其实首先要理解错误的意思，这里的XXXXBean一般是具体某个接口或抽象类，如IXXXService，翻译过来就是    没有可用的   满足IOpicsService接口类型的Bean</font>



:::info
解读：也就是说Bean确实是没有，但是并不是IOpicsService接口没有，而是没有对应的真正实现类即Bean

:::

**那么没有对应的实现类了，就一定是没写实现类或者实现类迁移了没引吗？**

> 有可能，但不一定，为什么说不一定，看下面的例子

<h3 id="rwR6q">例子</h3>

今天在测试一个接口，详情：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723444572052-68517660-ac98-4c7d-83b5-16ab00e9c125.png)

可以看到，是Sub4这个类没有找到，：后面可以看到就是该类加载的时候初始化失败了，初始化失败了那么类当然就没被加载了，没被加载那JVM上就没有，当然就是NoClassDefFoundException了

:::info
碰到类加载失败了，当然直接锁定到Sub4的类的静态属性

:::

从IDEA控制台也可以看到报错信息：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723444916885-3b7666c5-6c9c-4b5d-ac0b-999e64d61a8b.png)

有时候错误也会直接展示：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723447875675-883db1f9-ea44-47eb-926a-a2cc67756e6e.png)

如果没具体显示（有的时候显示的异常都不一样，可能这次请求是noClassDef，下次异常抛的就是noSuchBean），那么点开该类后发现属性大多为通过SpringUtils.getBean获取的，所以极有可能是内部getBean失败，抛出了NoSuchBeanException，所以也就导致了类加载初始化失败![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723445290778-4ab159d0-585c-4144-a0f8-e5438feff541.png)

我们找一个本工程的Bean在其静态块中调用Sub4来触发Sub4的加载，启动工程，断点到Sub4类静态代码块中，调用查看具体哪个Bean没有get到导致初始化失败

> （在@Autowired(require = false)的情况下可以查看是否为null，这个时候可以通过哪个service为null来判断谁注入失败）

经过在静态工程调试，我们发现时NoSuchBeanException：ITermopicssService，再看一下工程，发现类也是正常继承的，实现类应该也可以找到的,那为什么呢？

:::color3
PS: SpringUtils.getBean没有获取到Bean，那么将会直接抛出NoSuchBeanException

另外如果在启动时就报出了NoSuchBeanException或者初始化失败导致启动失败，说明断点打的往后了，打在static块的开头即可

:::

<details class="lake-collapse"><summary id="uf943381c"><span class="ne-text">只启动不排查</span></summary><p id="u8b7d22af" class="ne-p"><span class="ne-text">这时候找到了哪个接口的Bean创建失败了，如果我们仅仅想要项目正常启动，那么这种情况我们随便找一个Component，使用Autowired注入ITermopicssService，来让容器中有一个Bean，项目就能启动起来继续排查</span></p></details>

随便注入一下：

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723447024206-c9a5b222-ca9e-41b2-a94e-6896647645a3.png)

:::success
因为异常内部的异常是看不到的，所以有可能排查不到根本原因，所以启动之后多次断点

:::

启动起来之后，随便找地方断点

对比一下，发现Autowired就可以注入，但是SpringUtils如果get不到不也会创建Bean吗，为什么前面没Autowired的时候就是getBean会Exception呢？原因只有一个，getBean的时候没创建，那为什么没创建？![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723447226993-75869645-1221-41c2-9596-0bb78e25a659.png)

（Autowired能注入说明类结构和Component是没问题，那问题就在getBean）

<details class="lake-collapse"><summary id="u8a433c2d"><span class="ne-text">其中一种解释</span></summary><p id="u9ea8c5ca" class="ne-p"><span class="ne-text" style="color: rgb(51, 51, 51); font-size: 14px">一些特殊实例对象是存放在</span><code class="ne-code"><span class="ne-text" style="color: rgb(192, 52, 29); background-color: rgb(251, 229, 225); font-size: 12px">DefaultListableBeanFactory#resolvableDependencies</span></code><span class="ne-text" style="color: rgb(51, 51, 51); font-size: 14px">变量中的，在容器启动时，如果发现需要注入这些特定的实例对象，就直接在该变量中获取，自然也就不能通过</span><code class="ne-code"><span class="ne-text" style="color: rgb(192, 52, 29); background-color: rgb(251, 229, 225); font-size: 12px">BeanFactory#getBean</span></code><span class="ne-text" style="color: rgb(51, 51, 51); font-size: 14px">方法来获取了。</span></p><p id="uf8d36400" class="ne-p"><span class="ne-text" style="color: rgb(51, 51, 51); font-size: 14px">所以SpringUtils可能会拿不到。</span></p></details>

分别启动Sub4工程和调用工程，发现Sub4工程可以get到，而调用工程有时NoSuchBean，有时提示类未找到，总之就是Sub4工程正常，而调用工程创建不了Bean![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723448925111-c21cccb5-485a-4d42-a23e-b975444107c7.png)

对比一下，毫无区别

重新打包，本地调一下，还是这样，为什么一个工程换一个就不行了呢？

查资料了解到可能和@CompontentScan有关，查看两个Starter的父类MgBxpMainStarter

![](https://cdn.nlark.com/yuque/0/2024/png/35478834/1723455828021-0ede1ea5-e798-4b51-b6ac-989465132375.png)

可以注意两个地方，虽然最近改动过，但是现在没有这个配置的，返回true代表该类被排除，不在Spring扫描范围内

可以看到persistence其实就是ItermOpicsService的包路径，所以ItermOpicsService就被排除掉了，在启动的时候就不会被加载了

但是为什么static和固收一个能获取到一个不能呢？？？

:::success
原因是架构对BeanProcessor进行了重写，排除范围内的类，不会直接排除，而是如果产生了autowired行为就会注入，如果没有autowired行为说明未使用就不会创建，而SpringUtils.getBean原本应该会创建，但是重写的代码只处理了@Autowired的注入流程，让被排除的类又创建了Bean，但是SpringUtils肯定是没有类似处理的，所以get不到

:::

所以正是在static中某个地方autowired了，所以容器中有了，所以get才能get到，而固收没有DI，所以一直get一直nosuchbeanexception



