---
title: WebRTC学习终章：建立连接
date: 2022-10-03 13:55:30
tags:
- 浏览器
categories:
- 前端
top_img: /img/post/webrtc.png
cover: /img/post/webrtc.png
---

通过前两章的学习，已经对WebRTC有了大致的理解，那么作为前端开发，最关键的就是要通过WebRTC提供sdk来建立P2P的连接。

# 两端建立连接
+ AB两端通过信令服务器交换各自的SDP信息
+ AB两端通过实现STUN协议的STUN Server，获取各自的NAT结构，子网IP，公网IP和端口（Candiate信息），完成各自的打洞
+ AB两端通过信令服务器交换各自的Candiate信息
  - 如果AB在同一个NAT下，通过内网Candiate，交换后，即可建立连接
  - 如果AB都在非对称型NAT下，就依赖STUN Server获取公网Candiate，交换后，再建立连接
  - 如果AB有一端（或全部）在对称型NAT下，仅通过STUN Server获取公网Candiate还是会无法建立连接，此时就需要处于对称型NAT下的端去通过TURN Server进行服务转发，然后将转发形式的Candiate信息发送给另一端，交换后，建立连接
+ AB两端想目标IP端口发送报文，通过SDP中的信息，建立起加密长连接

# 两端操作详细流程
+ A创建Offer
+ A保存Offer（set local description）
+ A通过信令服务器发送Offer给B
+ B保存Offer（set remote description）
+ B创建Answer
+ B保存Answer（set local descripton）
+ B通过信令服务器发送Answer给A
+ A保存Answer（set remote description）
+ A通过信令服务器发送ICE Candidate给B
+ B通过信令服务器发送ICE Candidate给A
+ A和B就可以接收到对方的媒体数据

![WebRTC连接时序图](/img/post/webrtc/connection.png)

# demo实践
附上利用WebSocket和PeerJS实现的[视频白板会议室demo](https://github.com/renwangyu/lagou-demo/tree/master/my-video-room)