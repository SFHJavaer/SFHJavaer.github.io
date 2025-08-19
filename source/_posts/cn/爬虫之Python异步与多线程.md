---
layout: post
title: 爬虫之Python异步与多线程
catalog: true
typora-root-url: 爬虫之Python异步与多线程
author: Futari
date: 2023-09-12 22:04:43
---

:::info
💡  根据 [遗忘曲线](https://baike.baidu.com/item/%E9%81%97%E5%BF%98%E6%9B%B2%E7%BA%BF/7278665?fr=aladdin)：如果没有记录和回顾，6天后便会忘记75%的内容

:::

XPath获取结果是一个列表，原因是最终定位到某个标签，但是这个标签可能有多个，所以形成了单层列表，但是如果为/td/tr类型，td和tr都能定位到多个，所以就会依次类推形成多层list。（<font style="color:#DF2A3F;">结果是多层的，为了返回出来当然只能返回list啦</font>）



Python创建多线程的方式和Java类似，但是手动需要导一下包

**协程：一定是基于单线程的协程，在一个线程阻塞时执行下一位，提高CPU的运行效率，因为是单线程，所以不需要线程切换。被阻塞的线程由loop函数循环查看是否恢复到运行态**

<font style="color:#DF2A3F;">在python中使用协程：创建async函数，即异步函数</font>

然后使用asyncio的event_loop对象调用run_util_complete，或者直接使用async.run运行异步函数。



<font style="color:#DF2A3F;">async加在哪？</font>

加在所有的异步框架的对象前面，因为异步对象会进行异步操作

<font style="color:#DF2A3F;">await加在哪？</font>

加在阻塞操作上，即需要进行协程调整的地方，不管是网络阻塞还是IO阻塞，await相当于让当前任务到一边去，然后loop轮询着，先让别的任务在此线程进行异步。

如果阻塞操作不加await，那么就会导致系统认为该操作不需要进行异步，也不会将该任务放到一边等待，一定会报错

（当然这里我理解成不符合异步规则报错，而不是因为该语句没有得到结果就接着往下执行了，因为即使不协程了那也是单线程顺序执行吧

不过可以理解成语句得不到结果，系统可能将不加await的语句都当作非阻塞语句，然后也不会进行切换操作，结果却产生了阻塞效果，但是又没有给予切换指令，故而报错）

