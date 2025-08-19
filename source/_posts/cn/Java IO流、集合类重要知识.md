---
title: Java IO流
catalog: true
date: 2020-07-31 02:34:17
subtitle: Java IO流、集合类重要知识
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



### 集合类：

HashMap的key和value都可以是null，但是hashtable均不能为null

Arraylist底层是动态数组，arrayList的访问速度要比Linkedlist快（直接根据下标就能找到，并且对象是有序排列的，一个接一个就可以），所以在增删的时候为了要保证元素在内存上的连续性，需要进行重排

链表修改和删除的速度要快LinkedList底层是双向链表（在内存上不是连续的），根据Index大小判断从前往后还是从后往前遍历（顺着链条），所以时间复杂度是O(n)，增删只需要修改要操作位置的前后指针即可，不需要进行重排和其他操作。



```
java的数组之间是不能直接用强制类型转换的，想强转只能一个元素一个元素转

java直接使用toArray方法而不指定参数得到的是一个Object数组，可以指定它的参数为一个具体的数组对象，然后直接赋值给具体需要的数组对象即可，如list.toArray(new String[list.size()]);,对象转换为之后原本的引用不在适用，当然用一个新的引用来指向它

去重一种思路是用set集合来去除重复的元素，如果对顺序有一定要求的修改从前还是从后遍历即可
```

#### Java基础知识

final不能修饰接口，因为接口需要被实现，也不能修饰抽象类，因为抽象类需要被继承，final修饰的方法可以被重载，但是可以被重写，重载的关联比较小，而重写关联大，不可重写

在JDK8之前，接口中的方法都是没有方法体的方法，都需要实现类去实现所有的方法

在1.8之后，出现了default方法，就可以在接口中书写方法体，注意default方法在接口的实现类中可以不被（重写）实现，实现类当然会继承default方法，当然如果一个类实现了多个接口，其中有重复的default方法，那么此时就需要去重写，统一使用重写后的方法，不然不知道到底该使用哪一个接口的default方法

#### JDK8   Date新特性

SimpleDateFormatter被更新为DateTimeFormatter，采用了ISO 8061标准的日期格式化输出，可以使用of接收参数或者parse直接解析转化。

###### 同时引入了新的本地日期时间类，LocalDate或者LocalTime或者LocalDateTime

三者可以使用toxxx()转换，拼接成DateTime类底层是通过DateTime的构造方法new出来对象的。

日期类可以使用plusxxxx()和minusXXXX()进行年月日的加减操作，当然调整之后不符合时间规范会报错或者自动优化为符合时间。

**配合类：**           

###### Duiaton类代表日期之间的间隔，一般使用静态方法between取间隔。

###### Peroid类代表时间间隔，使用a.util(b)取间隔。