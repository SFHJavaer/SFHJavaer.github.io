---
layout: post
title: uni-app
catalog: true
typora-root-url: uni-app
author: Futari
date: 2023-03-07 17:27:29
---

### uni-app

##### 项目结构：

```js
┌─cloudfunctions        云函数目录（阿里云为aliyun，腾讯云为tcb，详见uniCloud）
│─components            符合vue组件规范的uni-app组件目录
│  └─comp-a.vue         可复用的a组件
├─hybrid                存放本地网页的目录，详见
├─platforms             存放各平台专用页面的目录，详见
├─pages                 业务页面文件存放的目录
│  ├─index
│  │  └─index.vue       index页面
│  └─list
│     └─list.vue        list页面
├─static                存放应用引用静态资源（如图片、视频等）的目录，注意：静态资源只能存放于此
├─wxcomponents          存放小程序组件的目录，详见
├─main.js               Vue初始化入口文件
├─App.vue               应用配置，用来配置App全局样式以及监听 应用生命周期
├─manifest.json         配置应用名称、appid、logo、版本等打包信息，详见
└─pages.json            配置页面路由、导航条、选项卡等页面类信息，详见
    
```

重点是pages、static、main.js、App.vue、manifest.json、pages.json这几个目录或者文件。

#### uniapp路由页面配置(pages.json)

pages是项目中用到的所有的页面配置，第一项是登录app的启动页，一般放登录页或者首页，其他的可以参考[官方文档](https://link.juejin.cn?target=https%3A%2F%2Funiapp.dcloud.io%2Fcollocation%2Fpages%3Fid%3Dpages)看看说明并手动设置下参数，看下效果就明白了。subPackages是分包加载配置，这个是为小程序准备的，小程序在发布上传代码时对总包大小(目前是16M)有要求，还要示对总包进行分包，每个小分包不能超过2M，这个就要求在这里进行配置分包加载。

globalStyle这个是设置全局样式的，一般都会有一个统一风格的样式，可以在这里设置，包括导航栏、标题、窗口背景色这些。tabBar这个也非常重要，这是app的标配，相当于是一级导航栏，拿我们这个hello uniapp的项目举例来说，框住部分就是tabBar的内容。

