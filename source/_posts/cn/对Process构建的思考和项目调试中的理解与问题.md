---
title: 对Process构建的思考和项目调试中的理解与问题
catalog: true
date: 2022-09-16 02:34:17
subtitle: 
lang: cn
header-img: /img/header_img/lml_bg.jpg
tags:
- 项目
categories:
- 项目
---

### 对Process构建的思考和项目调试中的理解与问题

###### 在项目接口测试中，请求体中的数据不能在param添加，因为不是@RequestParam的，要根据接口进行提交，选择的请求体类型一般是x-www-form-urlencoded或者是Json，前者一般是from表单的请求体，后者是一般的json通用请求类型

##### 一个联调安卓端的小问题：

安装完adb和aapt之后，配置环境变量，在项目中可以通过JDK的native方法去执行系统的cmd命令

![Lof30.png](https://s1.328888.xyz/2022/07/13/Lof30.png)

```java
我这里出了一个小问题，原因是配置完环境变量之后只是重启了项目，并没有重启idea，导致系统的环境没有配置到工程上，通过错误信息很明显地可以看到是因为系统找不到adb这个program，重启idea之后这个问题得到解决
```

通过研究项目源码以及错误信息定位：

发现执行cmd原理是通过创建一个进程来执行，通过抽象类Process实现，如下图

> Process类是一个抽象类，方法都是抽象的，它封装了一个进程，也就是一个可执行的程序 该类提供进程的输入、执行输出到进程、等待进程的完成和检查进程的退出状态及销毁进程的方法

![LotS5.png](https://s1.328888.xyz/2022/07/13/LotS5.png)

获取Runtime()对象来exec执行系统方法，exec源码如下图

![LoQJC.png](https://s1.328888.xyz/2022/07/13/LoQJC.png)

返回一个ProcessBuilder，即<font color = 'blue'>进程构建器</font>,绑定所需要的参数之后start()

![LoSid.png](https://s1.328888.xyz/2022/07/13/LoSid.png)

经过重载方法的调用，进入到其中，发现创建了Process的实现类ProcessImpl

![LopXR.png](https://s1.328888.xyz/2022/07/13/LopXR.png)

得到这个进程子类对象之后，执行其static方法start进行执行

![LoXuP.png](https://s1.328888.xyz/2022/07/13/LoXuP.png)

cmd进程被启动并执行

#### 项目中一些知识的普及

##### Monkey

Monkey是移动端app的自动化测试工具，我们系统在底层其实就是借助这些工具来完成对软件的测试，Monkey原理就是类似于猴子一样，通过向系统发送伪随机的用户事件流（如按键输入、触摸屏输入、滑动Trackball、手势输入等操作），来对设备上的程序进行测试，检测程序长时间的稳定性，多久的时间会发生异常。（**像猴子一样胡乱的输入，看程序是否会出错**）

> monkey需要通过adb来唤醒，即通过在cmd窗口中执行: adb shell monkey ｛+命令参数｝来进行Monkey测试;

###### 所以在我们项目中其实时通过adb来调用Monkey的

##### adb是什么？

> 在刷机领域中：adb是一个泛指，其实adb是包括两个工具的：Fastboot和ADB

adb的全称是安卓调试桥，也就是建立了安卓和开发环境之间的交流桥梁，是用来操作Android的一套指令集，通过shell来操作Android操作系统，注意只能调试安卓，所以对IOS的调试要另外开发。

##### 那aapt又是什么？

aapt是Android资源打包工具，通过复杂的流程将项目资源构建成为一个APK。

#### java日志LoggerFactory.getLogger

```java
private final Logger logger = LoggerFactory.getLogger(RequestFilter.class);
```

用来获取指定类的日志对象（Logger类型的），获取到日志对象之后调他的一些方法,如;

```java
logger.debug、logger.info、logger.warn、logger.error、logger.fatal
```

> 相同处：
> 它们的作用都是把错误信息写到文本日志里
>
> 不同的是它们表示的日志级别不同：
> 日志级别由高到底是：fatal -> error -> warn -> info -> debug,低级别的会输出高级别的信息，高级别的不会输出低级别的
>
> 信息，如等级设为Error的话，warn,info,debug的信息不会输出
>
> 修改日志输出的级别要在log4j文件中进行配置
> 项目正式发布后，一般会把日志级别设置为fatal或者error

##### 测试过程中的一个bug

当创建测试批次时会在两个map中分别put进两个kv，分别是（deviceId,执行器）（deviceId,deviceSuject）信息，第二个map中的值deviceSuject被先赋成null，然后到map中获取，如果系统重启，那么执行器和设备信息的map都会清空，所以在执行器ma.getp找不到value的时候就会创建一个新的Executor，但是设备信息deviceSubject没有重新生成，数据库中也没有保存这个type字段，所以会报空指针异常。

##### 关于应用测试完卸载不掉的bug

通过指定设备名和包名来执行卸载命令

>  adb -s 5ENDU19528001279 uninstall cc.baidu.app

```bash
Failure [DELETE_FAILED_INTERNAL_ERROR]
```

###### 解决方法有两种：

```txt
第一种：adb root
adb -s 5ENDU19528001279 uninstall cc.baidu.app
获取adb的root权限，但是没root的设备不能执行
```

```txt
第二种：
adb shell pm list packages -s 找到要删除的包名
//获取包名地址
adb shell pm path com.xxx.xxx
package:/data/app/包名/base.apk
//挂载系统读写权限
adb remount
remount succeeded
//删除包
adb shell rm /data/app/com.xxx.xxx-1/base.apk
//重启后ok
adb reboot
```

发现第二种其实还是i要root，没root的手机是没有卸载权限的，在adb shell进入到命令行模式的时候

```shell
HWPCT:/ $ adb root
/system/bin/sh: adb: inaccessible or not found
```

没有root的权限是获取不到的，对于设备的一些关键位置的访问也是access denied

1.禅道数据库不知道用不用的上，事实上会保存错误信息

2.配置文件路径（夹）是自己建的，不知道有没有需要的文件

3.缺少application.yml所需要的本地沙箱包

