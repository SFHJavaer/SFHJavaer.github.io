---
layout: post
title: 'Git和Repo项目管理'
date: 2022-07-03
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Git

---

#### Git提交问题

######  git commit 时可以加 -f代表强制提交，因为对于某些设置的文件是可忽略的，如果要强行提交的话，可以使用上面的命令。

```bash
git commit -f #强制commit
```

###### 忽略提交的文件：比如某些重要的配置文件、或者是测试文件不想被提交的情况。

Ps：只有.git目录以及他的下层目录才能代表这是一个git的仓库，git目录以及他的下层目录才是可以被识别的，不然凭空的话只会报下面的错误（.git这是一个隐藏文件夹）

```bash
$ git log
fatal: not a git repository (or any of the parent directories): .git
```

忽略提交的文件在.idea的IDEA项目工程文件夹中或者是仓库文件夹中，名为<font color = 'red' >.gitignore</font>,格式如下;

（当然不同的地方管理范围不同，idea可以管理IDEA中的提交功能，而仓库中的忽略文件更精确）

### 关于常用命令的快速上手

![LIHOP.png](https://ask.qcloudimg.com/http-save/yehe-8900250/84aa2e5545580bd6e69f8daf5a83e407.png?imageView2/2/w/1620)

#### Git Status查看状态

###### 工作区没有提交到暂存区，会显示成未被跟踪untracked

![img](https://img-blog.csdnimg.cn/20190717105517927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNjc1MDYyMw==,size_16,color_FFFFFF,t_70)

###### 只是add没有commit，即在暂存区

![Alt](https://img-blog.csdnimg.cn/20190717105946400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNjc1MDYyMw==,size_16,color_FFFFFF,t_70)

###### commit之后会显示   working tree clean

![Alt](https://img-blog.csdnimg.cn/20190717110443617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNjc1MDYyMw==,size_16,color_FFFFFF,t_70)

### Repo快速理解

Repo的基本操作先不说了，Repo其实就是一个python写的Git命令脚本，去管理Git的多个库，通过拉取项目管理文件也就是mainfest.xml来获取项目，mainfest.xml维护了源代码结构

> 查看Repo（不同的Repo脚本支持的操作也不一样，可以自己从网上下载，或者自己项目组编写）源代码可以看到，通过获取命令字符，去匹配相对应的函数进行操作，本质上还是到某某仓库去拉取代码，只不过用Repo去建立了多个仓库之间的联系

##### Repo基于Gerrit的原理（我观察是通过Group组的权限管理来控制拉取的）

比如init会去相对应的{project}去同步代码，比如这里的**pub**

![Lg3Gw.png](https://s1.328888.xyz/2022/07/14/Lg3Gw.png)

