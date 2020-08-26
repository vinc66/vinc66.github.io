---
title: websockettttttt
date: 2017-11-25 11:40:21
tags:
    网路通信
---
    
### websocket是什么
    WebSocket是H5提供的一种基于TCP链接进行full-duplex（全双工）通信的协议。
    http协议需要三次握手，一次request对应一次reponse，只能通过客户端主动向服务端发送请求。
    Websocket是一次握手，然后建立一个通道，持续通信。客户端可以向服务端请求，而且服务端也可以主动向客户端发送信息。
    当然也可以通过类似ajax轮询之类的方式做到类似的功能，但是比较被动，和一条懒狗是的，得戳一下才能挪几步，很耗费精力（资源）。
    websocket可是真正的双向通信。
    
### 应用场景
    1.社交订阅
    2.多玩家游戏	 
    3.协同编辑/编程
    4.点击流数据	     
    5.股票基金报价
    6.体育实况更新
    7.多媒体聊天
    8.基于位置的应用
    9.在线教育
    
### 具体的
    我就写了一下， 基于spring-websockt的，当然直接用tomcat8自带的tomcat-websocket.jar也成 。这里不得不说下websocket当下最火的应用
    场景，弹幕，我就直接去b站了。
{% link 基于websockt聊天室 https://github.com/BinaryVinc/vserver/tree/chat-room [external] [title] %}

####    a.直播

{% img /img/20171125-websocket学习/1.png %}
#####   上图有这么几个参数需要搞一哈
    Request Headers
    Connection: Upgrade #表示当前的连接方式是升级版的，普通的http一般是keep-alive长连接方式
    Upgrade: websocket  #升级版的叫做websocket
    Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw== #这是浏览器随机生成的base64之后的值，作为一个有身份的链接
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13

    Response  Headers
    Sec-WebSocket-Accept    #这个是Sec-WebSocket-Key加密后的一串    
{% img /img/20171125-websocket学习/2.png %}
#####   上图WebSocket通信的Time一直是pending。
{% img /img/20171125-websocket学习/3.png %}
#####   frames就是具体通道

####    b.非直播
{% img  /img/20171125-websocket学习/4.png %}
    先把以前包括时间点，屏幕位置的弹幕信息拉下来，
    当然还有一个websocket通道来收集新的弹幕。