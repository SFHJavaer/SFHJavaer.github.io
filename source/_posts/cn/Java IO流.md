---
title: Java IO流
catalog: true
date: 2020-07-31 02:34:17
subtitle: JavaIO流学习笔记
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- IO
categories:
- IO
---

## Java IO流

英文字母在windows中占1个字节   比如：'a'字符

中文汉字在windows中占2个字节   比如：“中”字符

字节流一次读取一个字节

字符流一次读取一个字符



IO流四大家族的首领：

##### java.io.InputStream				字节输入流

##### java.io.OutputStream				字节输出流

##### java.io.Reader 				字符输入流

##### java.io.Writer					字符输出流



**注意：在Java中只要类名以Stream结尾的都是字节流，以Reader或者Writer结尾的都是字符流**

**所有的流都实现了Java.io.Closeable接口，都是可关闭的，都有close方法，流是一个管道，用完之后一定要关闭**

**flush（）是刷新方法，输出流实现了它（java.io.flush接口），输入流没有实现（注意硬盘上的文件读取到内存到读（内存读）），所以用完输出流之后也要刷新一下，刷新表示将未输出的数据强行输出完（清空管道）**



需要掌握的几个主要的流：文件专属流		转换流		数据流		标准输出流		对象专属流

下面的展示了FileInputStream读取文件的基本代码：

```java
package com.java.io;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

//读和写是把内存作为主体
//路径：/ 代表的路径是根目录；. 代表的路径是当前Java项目的根目录,idea默认相对路径也是从工程的根目录开始。所以开头直接可以指定到src
public class FileInputStreamText {
    public static void main(String[] args) {
        FileInputStream file = null;
        //创建文件字节输入流,main方法异常一般直接try catch
        try {
            file = new FileInputStream("src/com/java/io/File.txt");





//方法一：下面的read方法只能读取单字节，可以利用自建一个byte数组来读取多个字节
            //            int readData = 0;
//            while((readData = file.read()) != -1){
//                System.out.println(readData);
//            }
            //方法二最终版
            byte[] bytes = new byte[4];
            int readCount = 0;
            while((readCount = file.read(bytes)) != -1){
                System.out.print(new String(bytes,0,readCount));
            }
//下面是方法二
//            while(true){
//                //读取一个数组返回的是一次读取字节的数量，而不是字节值
//                int readCount = file.read(bytes);
//                if(readCount == -1){
//                    break;
//                }
//                //把读到的readCount个数据打印出来，因为用数组存储字节，对第上一轮数据没有覆盖的话，数组中未被覆盖的字节还会保留。
//                System.out.print(new String(bytes,0,readCount));
//            }





        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //在finally中关闭流,关闭流的前提是 流不是空，避免空指针异常
            if (file != null){
                try {
                    file.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

arraylist里具体元素处没有元素是不能set的，会报数组越界异常，先要用add添加元素

set是覆盖原来索引处的数据，而add的话，如果是已经有数据的地方add，那么会将原来存在的数据往后移，联系Arraylist的线性结构就能理解

#### 多线程

java线程的级别是内核级的，其实详细的说是用户-内核级的，是由内核线程直接调度用户线程，在jdk1.1时，内核线程指将资源分配给主线程，然后用户线程进行调用，之后就修改了资源分配的机制，因为每一个线程从底层都是一个函数，都会有一个函数入口，所以所以一个函数的栈是不共享的，而堆和代码区和数据区是共享的（在JVM里才是方法区），线程操作系统分配资源的最小单位

CAS是乐观锁，有三个参数，（地址参数，旧值，新值），可以保证线程修改完，下一个线程修改时如果当前内存的旧值和自己的旧值不相同，就会拒绝当前线程，线程重新进行执行，相当于一个while循环，循环参数为cas，但是可能会产生ABA问题

set和get是一条语句，是一个原子的操作，但是自增不是原子的，自增包含了三个原子操作



使用AtomicInteger等类可以进行原子操作,源码使用了CAS，CAS是实现自旋锁的基础，而且CAS是无锁操作，减少性能损耗，cas的原子操作性能比线程加一个锁要高

解决ABA问题的类：使用了增加版本的方式AtomicStampedRefernence，在进行比较时增加一个mark版本号的比较

锁升级：一个线程获取了锁，对象上markword上标记上线程id，如果出现竞争，升级为轻量级锁，如果自旋失败，升级为重量级锁，这里指的是锁的状态，自旋指的是锁的类型

synchronized的本质是monitor，调用内核的资源，monitor可以执行两个操作monitorenter（monitor中的线程数为0，线程进入并将进入数加一，如果不为0则该线程会被阻塞，等待其他线程出monitor）和monitorexit底层指令执行，entryset和waitset的调度，如果线程调用wait就会进入到waitset中

wait/notify方法必须在同步代码块中，因为必须先持有锁才能进行锁的释放，而且容易出现wakeup问题。因为这两个方法本身就不是原子操作