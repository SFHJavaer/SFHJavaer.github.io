---
layout: post
title: '简单实现ActiveMQ、RabbitMQ'
date: 2022-03-07
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Spring RabbitMQ
---
#### JMS：一个规范的接口，相当于JDBC

JMS消息模型有两种：p2p（点对点模式）和pub-sub（发布者-订阅者模式）

JMS常用的消息种类我们主要使用BytesMessage

规范的实现主要有：ActiveMQ，Redis，HornetMQ，RabbitMQ，RocketMQ（没完全遵守JMS规范）



AMQP：协议，对定了网络交换的数据格式

##### ActiveMQ

MQ都是在配置文件中加配置，依赖是spring-boot-starter-xxx系列

配置最重要的是ip地址和端口（broker-url），说清楚MQ服务在哪，分清楚管理端口和服务端口

配置的是管理端口，自己编程用的是服务端口

注入JmsMessagingTemplate，ActiveMQ的API接口,调用convertAndSend方法（先转码再发送），同理在消息处理端使用receiveAndConvert方法接收

> 发布订阅的消息再管理端的topics中呈现

设置ActiveMQ的监听器，一旦消息队列中进入了消息，Listener直接调用他所属的处理方法，不会等待，将类交给Spring管理的同时加入@JmsListener注解（参数为destination指定要处理的消息名）

> @SendTo注解可以将下面方法的返回值保留，返回给参数中指定的消息，所以可以使用@SendTo做消息队列的链式处理
>
> 生产与消费消息

#### RabbitMQ：

###### direct：直连模式

在Service中注入AmqpTemplate而不是RabbitTemplate，命名遵守的是具体实现类而不是接口了



