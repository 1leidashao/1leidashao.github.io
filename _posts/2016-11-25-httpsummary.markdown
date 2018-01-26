---
layout:     post
title:      "何为Web"
subtitle:   "web的今生前世"
date:       2016-11-29 14:00:00
author:     "Fengzhao"
header-img: "img/post-bg-2015.jpg"
tags:
     
    -TCP/IP
---

# Web概述 #
W3C的[官方文档](http://www.w3.org/WWW/)的第一句话是这样的："The World Wide Web(known as"WWW","Web"or"W3")is the universe of network-accessible information,the embodiment of human konwledge".如果将这句话翻译成中文，就是"Web是网络信息的来源，知识的化身"。Web为我们提供了一种获取和操作网络资源的方式，这些资源不仅仅包含像文字和图片这些传统的信息载体，还包含音频和视频这些多媒体信息。Web的核心主要体现在三个方面，即HTTP、超文本和超媒体。超文本和超媒体规范了网络信息的表现形式，他们利用“链接”在相关资源节点之间连接形成一张非线性的网(Web)，HTTP提供了网络访问的标准协议。

## 1、HTTP报文 ##
  客户端和Web服务器在一次http事务中交换的消息被称为HTTP报文，客户端发送给服务器的请求消息被称为请求报文，服务器返回给客户端的响应消息被称为响应报文。请求报文和响应报文采用纯文本编码，由一行行简单的字符串组成。一个完成的HTTP报文由如下三个部分组成：
  *   起始行：代表HTTP报文的第一行文字，请求报文利用起始行表示采用的HTTP方法、请求URL、和采用的HTTP版本，响应报文的起始行内容是HTTP版本和响应状态码。
  *   报头集合：起始行后面可以包含零个或多个报头字段，每个报头表现为一个键值对。键和值分别表示报头名称和报头的值，两者通过：号来分割。HTTP报文采用一个空行来作为报头集合结束的标志。
  *   主体内容：报头集合的空行结束之后就是报文的主体内容。客户端提交给服务器的数据一般至于请求报文的主体中，响应报文的主体也承载着服务器返回给客户端的数据。无论是请求报文和响应报文，其主体内容均是可以忽略的。

  下面我们来看一个请求报文的实例，这个片段反应了我访问微软官网的请求报文。
        GET http://www.microsoft.com/zh-cn HTTP/1.0
        Host: www.microsoft.com
        Connection:keep-alive
        Cache-Control:max-age=0
        User-Agent: Fiddler
    这行报文的内容，
