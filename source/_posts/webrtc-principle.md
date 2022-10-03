---
title: WebRTC学习中篇：通话原理
date: 2022-10-03 11:46:49
tags:
- 浏览器
categories:
- 前端
top_img: /img/post/webrtc.png
cover: /img/post/webrtc.png
---

# 连接的建立离不开Offer与Answer的交换流程
先来看个场景，两个端通信，和我们平时和服务器通信类似（http三次握手），都需要进行一些信息的交换。对WebRTC来说，至少需要如下步骤：
+ 创建Offer，发送Offer
+ 创建Answer，发送Answer
+ 发送 / 接收媒体信息和网络信息
+ 创建P2P链接，传输数据

![](/img/post/webrtc/offer-answer.png)

WebRTC一旦建立，端与端之间可传输内容：
+ 媒体流：navigator.getUserMedia，传递音频，视频
+ 任意数据：dataChannel（可读写的全双工通道），传递文件

# 信令服务器（Signal Server）
要建立P2P的媒体传输，就必须要有媒体协商和网络协商，那么如何让AB两端对于媒体信息和网络信息做交换，就需要通过一个信令服务器作为中间层来进行。

![](/img/post/webrtc/signal-server.png)

## 概念
简单来说，信令服务器也是一种web服务器，只是传输的不是普通的数据，而是信令。就好像一个中间人，在A与B两端穿针引线，让双方尽可能以安全和简洁的信息传输，使两端能建立通话。
信令就是为了使两端能建立通信链接，交换信息协调通信的过程。信令交换的信息大概有：
+ 回话控制消息用于open/close通信
+ error message
+ Media Meta数据：如编解码器，带宽，媒体类型
+ 秘钥数据：建立安全的链接
+ 网络数据：IP和端口

## 作用
信令服务器一般搭建在公网或两端都能访问的局域网，实现SDP媒体信息和Candidate网络信息的交换服务：
+ A端通过websocket发送信息（SDP，Candidate）给到信令服务器
+ 信令服务器通过websocket把A的信息发送给B
+ 反正亦然，通过信令服务器的交换，AB两端即可P2P链接
此外也可以充当部分传统服务的功能：
+ A端发起会话请求，给到信令服务器
+ 信令服务器再转发到B端
+ B端采取接受或拒绝，反馈给信令服务器
+ 信令服务器把结果再给到A端

## 方式类型
WebRTC并没有强制信令交换机制的方式，所以WebSocket或XHR都是可以的，作为Web服务器，自然也可以用java，go，nodejs等。

# 媒体协商（SDP）

## 概念
两个客户端（A和B）想创建链接，一般来说需要有一个双方都能访问的服务器来帮忙交换链接所需要的信息。首先两端要交换的就是SDP（Session Description Protocol），了解对方支持的媒体格式，再建立双方想要的链接。

![](/img/post/webrtc/sdp-1.png)

> 比如A支持VP8，H264，B支持VP9，H264，那么经过协商后，取交集H264

SDP作为一个专门的协议，就是用于描述上类信息，WebRTC用用中，音视频通讯的两端必须先交换SDP信息，这样才能链接通信。这个交换SDP的过程，即为媒体协商。

## 过程
+ 在建立链接前，双方需要通过API指定要传输的数据类型（Audio，Video，DataChannel）和想要接受的数据。
+ 一端通过CreateOffer()方法，获取offer类型的SessionDescription，再通过公共服务器传递给另一端。
+ 另一端再通过CreateAnswer()，获取answer类型的SessionDescription，再通过公共服务器传递给发起方。
+ 哪一端是创建Offer或Answer都是可以的，只要保证双方创建的SessionDescription类型是相互对应的。
+ 信令服务器是用来交换双方SDP信息，一般是通过Socket链接交换处理，在Nodejs中可以使用WebSocket。

## SDP协议
会话描述协议（Session Description Protocol），描述多媒体链接内容太的协议，让两端在数据传输时能识传输的数据。

![](/img/post/webrtc/sdp-2.png)

+ 分辨率，格式，编码，加密算法等
+ 描述设备间媒体链接的一种数据格式
+ 一行或多行UTF-8文本组成，每行以一个字符的类型开头，等号后是值或描述的结构化文本（格式取决于类型）
  
```bash
//------------------------------- 会话层 -------------------------------
// version <sdp版本号，不包括次版本号>
v=0
// 过程中有改变编码之类的操作,重新生成sdp时,session id不变,session version加1
// owner <user name> <session id> <session version> <net type> <addr type> <addr>
o=- 4571619080104764276 2 IN IP4 127.0.0.1
// session <session name>
s=-
// webrtc始终是0
// time <开始时间> <结束时间>
t=0 0
// attribute,0 1绑定一个传输通道传输,与下方的a=mid相匹配
a=group:BUNDLE 0 1
// media stream id缩写msid,webrtc media stream缩写WMS
a=msid-semantic: WMS 9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU

//------------------------------- 媒体层:以下为音频描述信息 -------------------------------
// 端口9表示不接收数据;secret audio video protocol family缩写SAVPF
// media <媒体类型> <port> <传输协议> <payload type集>
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
// 该属性webrtc并没有使用
// connection
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
// 以下两行ice协商过程中的安全验证信息
a=ice-ufrag:FvYc
a=ice-pwd:st4MHVcrFq+pnZgFNV2qSmW0
// trickle,即sdp里面描述媒体信息和ice候选项的信息可以分开传输
a=ice-options:trickle
// dtls协商过程中需要的认证信息,sha-256加密算法
a=fingerprint:sha-256 94:85:06:47:AD:89:09:7D:AF:E0:48:B7:58:55:1D:44:48:34:8F:4A:39:98:62:44:B0:19:BA:03:00:B9:94:66
// actpass既可以是客户端,也可以是服务器;active客户端;passive服务器
a=setup:actpass
// 前面a=group:BUNDLE中用到的媒体标识
a=mid:0
// 要在rtp头部中加入音量信息
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
a=extmap:2 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
a=extmap:3 urn:ietf:params:rtp-hdrext:sdes:mid
a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id
a=extmap:5 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id
// 仅发送,其他类型sendrecv,recvonly,sendonly,inactive
a=sendonly
// 与前面的msid相同,第二个为track id
a=msid:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU a1987158-6a67-4b19-9931-5cd2a8d2f6f8
// rtp,rtcp使用同一个端口来传输
a=rtcp-mux
// payload type的描述
a=rtpmap:111 opus/48000/2
a=rtcp-fb:111 transport-cc
a=fmtp:111 minptime=10;useinbandfec=1
a=rtpmap:103 ISAC/16000
a=rtpmap:104 ISAC/32000
a=rtpmap:9 G722/8000
a=rtpmap:0 PCMU/8000
a=rtpmap:8 PCMA/8000
a=rtpmap:106 CN/32000
a=rtpmap:105 CN/16000
a=rtpmap:13 CN/8000
a=rtpmap:110 telephone-event/48000
a=rtpmap:112 telephone-event/32000
a=rtpmap:113 telephone-event/16000
a=rtpmap:126 telephone-event/8000
a=ssrc:1196321802 cname:kDl3rfnZtJEKUJsa
a=ssrc:1196321802 msid:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU a1987158-6a67-4b19-9931-5cd2a8d2f6f8
a=ssrc:1196321802 mslabel:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU
a=ssrc:1196321802 label:a1987158-6a67-4b19-9931-5cd2a8d2f6f8

//------------------------------- 媒体层:以下为视频描述信息 -------------------------------
m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 102 122 127 121 125 107 108 109 124 120 123 119 114 115 116
c=IN IP4 0.0.0.0
// CT是设置整个会议的带宽，AS是设置单个会话的带宽，它们的单位都是kbit/s。setRemoteDescription之前修改
// bandwidth <类型>:<带宽>
b=AS:100

a=rtcp:9 IN IP4 0.0.0.0
a=ice-ufrag:FvYc
a=ice-pwd:st4MHVcrFq+pnZgFNV2qSmW0
a=ice-options:trickle
a=fingerprint:sha-256 94:85:06:47:AD:89:09:7D:AF:E0:48:B7:58:55:1D:44:48:34:8F:4A:39:98:62:44:B0:19:BA:03:00:B9:94:66
a=setup:actpass
a=mid:1
a=extmap:14 urn:ietf:params:rtp-hdrext:toffset
a=extmap:13 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
a=extmap:12 urn:3gpp:video-orientation
a=extmap:2 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01
a=extmap:11 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay
a=extmap:6 http://www.webrtc.org/experiments/rtp-hdrext/video-content-type
a=extmap:7 http://www.webrtc.org/experiments/rtp-hdrext/video-timing
a=extmap:8 http://tools.ietf.org/html/draft-ietf-avtext-framemarking-07
a=extmap:9 http://www.webrtc.org/experiments/rtp-hdrext/color-space
a=extmap:3 urn:ietf:params:rtp-hdrext:sdes:mid
a=extmap:4 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id
a=extmap:5 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id
a=sendonly
a=msid:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU f2b07fae-01be-4c8e-ac69-895c1d44ff87
a=rtcp-mux
// 尽可能的减少rtcp包的发送，只发丢包
a=rtcp-rsize
// payload type的描述,编码器VP8,时钟采样率90000
a=rtpmap:96 VP8/90000
// 对rtpmap 96的描述,google标准的接收端带宽评估
a=rtcp-fb:96 goog-remb
// 对rtpmap 96的描述,传输端的带宽评估
a=rtcp-fb:96 transport-cc
// 对rtpmap 96的描述,支持客户端请求i帧,codec control using RTCP feedback message缩写ccm,Full Intra Request缩写fir
a=rtcp-fb:96 ccm fir
// 支持丢包重传
a=rtcp-fb:96 nack
// 支持i帧重传
a=rtcp-fb:96 nack pli
// rtx丢包重传
a=rtpmap:97 rtx/90000
// apt关联
a=fmtp:97 apt=96
a=rtpmap:98 VP9/90000
a=rtcp-fb:98 goog-remb
a=rtcp-fb:98 transport-cc
a=rtcp-fb:98 ccm fir
a=rtcp-fb:98 nack
a=rtcp-fb:98 nack pli
a=fmtp:98 profile-id=0
a=rtpmap:99 rtx/90000
a=fmtp:99 apt=98
a=rtpmap:100 VP9/90000
a=rtcp-fb:100 goog-remb
a=rtcp-fb:100 transport-cc
a=rtcp-fb:100 ccm fir
a=rtcp-fb:100 nack
a=rtcp-fb:100 nack pli
a=fmtp:100 profile-id=2
a=rtpmap:101 rtx/90000
a=fmtp:101 apt=100
a=rtpmap:102 H264/90000
a=rtcp-fb:102 goog-remb
a=rtcp-fb:102 transport-cc
a=rtcp-fb:102 ccm fir
a=rtcp-fb:102 nack
a=rtcp-fb:102 nack pli
a=fmtp:102 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42001f
a=rtpmap:122 rtx/90000
a=fmtp:122 apt=102
a=rtpmap:127 H264/90000
a=rtcp-fb:127 goog-remb
a=rtcp-fb:127 transport-cc
a=rtcp-fb:127 ccm fir
a=rtcp-fb:127 nack
a=rtcp-fb:127 nack pli
a=fmtp:127 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=42001f
a=rtpmap:121 rtx/90000
a=fmtp:121 apt=127
a=rtpmap:125 H264/90000
a=rtcp-fb:125 goog-remb
a=rtcp-fb:125 transport-cc
a=rtcp-fb:125 ccm fir
a=rtcp-fb:125 nack
a=rtcp-fb:125 nack pli
a=fmtp:125 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e01f
a=rtpmap:107 rtx/90000
a=fmtp:107 apt=125
a=rtpmap:108 H264/90000
a=rtcp-fb:108 goog-remb
a=rtcp-fb:108 transport-cc
a=rtcp-fb:108 ccm fir
a=rtcp-fb:108 nack
a=rtcp-fb:108 nack pli
a=fmtp:108 level-asymmetry-allowed=1;packetization-mode=0;profile-level-id=42e01f
a=rtpmap:109 rtx/90000
a=fmtp:109 apt=108
a=rtpmap:124 H264/90000
a=rtcp-fb:124 goog-remb
a=rtcp-fb:124 transport-cc
a=rtcp-fb:124 ccm fir
a=rtcp-fb:124 nack
a=rtcp-fb:124 nack pli
a=fmtp:124 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=4d0032
a=rtpmap:120 rtx/90000
a=fmtp:120 apt=124
a=rtpmap:123 H264/90000
a=rtcp-fb:123 goog-remb
a=rtcp-fb:123 transport-cc
a=rtcp-fb:123 ccm fir
a=rtcp-fb:123 nack
a=rtcp-fb:123 nack pli
a=fmtp:123 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=640032
a=rtpmap:119 rtx/90000
a=fmtp:119 apt=123
// fec冗余编码，rtp头部负载类型116，否则就是各编码原生负责类型
a=rtpmap:114 red/90000
a=rtpmap:115 rtx/90000
a=fmtp:115 apt=114
//支持ULP FEC
a=rtpmap:116 ulpfec/90000
// 一个cname可以对应多个ssrc
a=ssrc-group:FID 790880351 2784909276
a=ssrc:790880351 cname:kDl3rfnZtJEKUJsa
a=ssrc:790880351 msid:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU f2b07fae-01be-4c8e-ac69-895c1d44ff87
a=ssrc:790880351 mslabel:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU
a=ssrc:790880351 label:f2b07fae-01be-4c8e-ac69-895c1d44ff87
a=ssrc:2784909276 cname:kDl3rfnZtJEKUJsa
a=ssrc:2784909276 msid:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU f2b07fae-01be-4c8e-ac69-895c1d44ff87
a=ssrc:2784909276 mslabel:9TH6yxQRgyF6Yna81tiidOMebGqKms24SCEU
a=ssrc:2784909276 label:f2b07fae-01be-4c8e-ac69-895c1d44ff87
```
> 注：虽然SDP中有IP和端口的信息，但WebRTC并没有使用这些信息，而是通过ICE框架来建立链接的。

# 网络协商（Candidate）
## 概念
通过两端的网络情况，找到一条能P2P通信的链路：
+ 获取外网IP地址映射
+ 通过信令服务器（Signal Server）交换网络信息
+ 理想状况：每个浏览器都是唯一的公网IP，很简单直接P2P进行链接，爽

![理想情况](/img/post/webrtc/candidate-1.png)

+ 实际情况：电脑都是在某个局域网中，且都有防火墙，访问公网需要NAT，难搞

![实际情况](/img/post/webrtc/candidate-2.png)

## NAT技术
网络地址转换（Network Address Translation），就是为了解决IPV4下IP地址匮乏而出现的技术。比如一个公网路由器下，会有n个内网IP地址。

![NAT架构](/img/post/webrtc/nat-frame.png)

其中NAT核心的映射表实际存储关系，随着NAT的类型不同，可能会包含：
+ 内网IP和端口
+ NAT外网IP与端口
+ 目标IP和端口
  
![](/img/post/webrtc/nat-type.png)

再看看NAT的类型，主要分为四种：
+ 完全锥形NAT
  - 内网主机有自己的IP和端口，通过路由器防火墙NAT后，会得到一个外网IP。那么外网只要有一台主机曾经被内网主机请求过，就会在NAT上打洞，形成一个外网IP和端口。那么外网主机只要获取这个外网的IP和端口，就都可以顺利通过防火墙，向内网主机发送数据，形成通信。
  - 容易穿越，相应的，安全性差。

![](/img/post/webrtc/nat-1.png)

+ 受限圆锥型NAT
  - 映射会保存内网主机IP和端口，映射的外网IP和端口，目标IP
  - 当内网主机向外网目标发送请求，就会在NAT上生成一个映射表，那么目标就可以通过不同的端口（IP受限，端口不受限）来返回给内网主机，建立通信。相应的，其他的外网主机就无法通过NAT的刚才那个目标IP限制（因为此时映射表里没有外网其他主机IP），所以无法通信。
  - 所以如果内网主机要和外网目标通信，就都需要发送请求，映射表上记录，方能建立通信。

![](/img/post/webrtc/nat-2.png)

+ 端口受限圆锥型NAT
  - 映射会保存内网主机IP和端口，映射的外网IP和端口，目标IP和端口。
  - 和受限圆锥型NAT类似，只是在IP限制的基础上加了端口限制，即使同一台外网主机，如果端口不同，也无法建立连接。

![](/img/post/webrtc/nat-3.png)

+ 对称型NAT
  - 上述三种在形成映射后的外网IP和端口是不变的。外网机器想找还是能找到这个外网的IP和端口，只是不一定能建立通信。
  - 对称型NAT除了IP受限，端口受限之外，对每一台外网主机都会形成一个不同IP和端口映射，上一次的外网IP和端口，对这一次就没用了。

![](/img/post/webrtc/nat-4.png)

基于上述原理，路由器防火墙这一类设备的NAT会保护内网地址的安全，所以当想采用P2P链接方式时候，外网地址的访问就会被阻止，自然无法建立连接。
此时就需要某种技术（打洞）去穿透NAT，达到连接的目的。
针对NAT的打洞，在建立P2P连接时，针对上面说的NAT的四种不同类型，会分别进行不同处理：
+ 完全圆锥型：双方中任何一方都可以发起通信连接。
+ 受限圆锥型 / 端口受限圆锥型：必须一开始两端就向对方发起请求，保证双方的NAT都有NAT映射，就能建立连接。
+ 对称NAT：终端向STUN Server发送请求，映射的公网IP和端口，与向其他终端发送，映射的公网IP和端口是不同的，一个连接只能创建一个公网映射，其他终端无法使用之前通过STUN Server打好的洞，因此双方无法连接成功，此时就需要使用TURN做中转。

## ICE框架
首先，如果有这么一个公网IP服务器，AB两端都可以向其发送发包，公网服务器就可以获知AB双方的IP和端口，又因为是AB双发主动向公网IP服务器发包，所以公网服务器就可以穿透双方的NAT并发包给对应的A/B端。所以只要公网IP将A/B的IP和端口发给B/A，那么就AB就可以进行消息传输了。
互动式链接建立（Interactive Connectivity Establishment）提供的是一种框架，整合了各种NAT穿透技术（STUN，TURN），实现了统一。主要作用于让客户端成功穿透远程用户与网络之间可能存在的各类防火墙。

## STUN协议
![STUN架构](/img/post/webrtc/stun.png)

NAT会话穿越应用程序（Session Traversal Utilities for NAT）
+ 一种网络协议，该协议由RFC 5389定义。
+ 根据此协议实现的服务器称为STUN服务器。
+ STUN是用来探测终端NAT类型，IP和端口的服务。允许位于NAT（或多重NAT）后的客户端找出自己的公网地址和端口，并查出自己位于哪种类型的NAT之后，以及NAT为某一个本地端口所绑定的Internet端端口。
+ 在获取到NAT类型，IP和端口后，就会触发WebRTC的candidate事件，然后连接双方交换IP与端口，开始打洞。
+ 这些信息被用来在两个同时处于NAT路由器之后的主机之间创建UDP通信。
+ 简单说来，STUN只是帮助终端找到自己在公网网关对应的IP和端口（不会分配），不做其他事了。
> 注：之后的媒体信息交换还是信令服务器的作用。

+ 传输媒体流的STUN服务器也是采用P2P方式搭建。
+  STUN即使取得了公网IP地址，也不一定能建立链接。因为不同的NAT类型处理传入的UDP分组方式是不同的。
> 注：完全圆锥型NAT，受限圆锥型NAT，端口受限圆锥型NAT可以使用STUN穿透；对称型NAT（双向NAT），STUN无法打通，就需要使用TURN技术。

+ WebRTC使用ICE框架（整合了STUN与TURN），只要两端交换了IP和端口后，就会自动打洞，如果打洞失败，就会使用TURN Server转发流量。

## STUN实现的NAT探测流程
注意，先记着，STUN Server的IP有两个（IP1，IP2）

### step1：探测终端是否可用UDP，是否在NAT的内网中
+ 终端向STUN Server的IP1的端口1发送一个UDP包。
+ STUN Server收到包后，把包的源IP和源端口写入UDP包中，再通过STUN Server的IP1和端口1返回给终端。
  - 如果没有收到STUN的返回，说明STUN Server不存在，或配置错误。NAT设备有问题。
+ 终端收到UDP包后，将之前写入的源IP，和自己的IP作比较。
  - 如果一致，就说明终端在公网。
  - 如果不一致，说明在内网，有NAT的存在，进行step2。

### step2：探测NAT是否是完全圆锥型
+ 终端向STUN Server的IP1发送一个UDP包，但要求STUN Server通过IP2向终端返回UDP包。
  - 如果终端收到UDP包，说明NAT是来者不拒的，所以是完全圆锥型NAT。
  - 如果终端未收到UDP包，继续进行step3。

### step3：探测NAT是否为对称型
+ 终端向STUN Server的IP2的端口2发送一个数据包
+ STUN Server收到包后，把包的源IP和源端口写到UDP包中，在通过自己的IP2和端口2把包返回给终端。
+ 终端收到UDP包后，对比源端口信息。
  - 如果包里的源端口和step1的端口不一样，则说明是对称型NAT。
  - 如果包里的源端口和step1的端口一样，继续进行step4。
> 注：主要是因为对称型NAT对每次IP或端口改变的连接，都会重新分配一个端口使用，如果step1和step3的端口不一致，就可以确定是对称型NAT了。

### step4：探测NAT是受限圆锥型还是端口受限圆锥型
+ 终端向STUN Server的IP2的某个端口（比如端口X）发送一个数据包，要求STUN Server通过IP2但不是端口X的端口向终端返回数据包。
  - 如果终端收到返回数据包，表示只要IP相同，但是端口不同，NAT允许数据包返回，就是受限圆锥型NAT。
  - 如果终端未收到，表示端口受限NAT。

## TURN协议
![TURN架构](/img/post/webrtc/turn.png)

有些场景（参考上述），STUN技术无法穿透（找不到对方的IP和端口）的时候，就需要公网的服务器作为一个中继， 对来往的数据进行转发。这个转发的协议就被定义为TURN（Traversal Using Relays around NAT）：通过Relay方式穿越NAT。
+ 是RFC 5389的一个拓展。
+ 添加了Relay功能。
+ STUN分配公网IP失败，通过TURN服务器请求公网IP地址作为中继地址。媒体数据由TURN服务器中转。

> 注：此时的媒体传输带宽由服务器承担，相应的本地带宽压力就小了。