---
title: WebRTC学习笔记：基础知识
date: 2022-10-03 10:13:58
tags:
- 浏览器
categories:
- 前端
top_img: /img/post/webrtc.png
cover: /img/post/webrtc.png
---

# WebRTC简介
## 什么是WebRTC
网页即时通信（Web Real-Time Communication），是一个支持浏览器进行实时语音对话或视频对话的API。能通过简单的API为Browser和App提供实时通信（RTC）的功能。
WebRTC 是一种 HTML5规范，是无需在浏览器中安装任何插件就可以在网页内进行实时通信工作的开源技术，直接在浏览器和设备之间添加实时媒体通信。
2011年6月开源，并在Google，Mozilla，Opera的支持下被纳入万维网联盟W3C推荐标准。 

## WebRTC前景
并不仅仅受限于Browser环境，而是无论Browser，桌面应用，移动应用或loT设备，只要IP链接可到达且符合WebRTC规范即可互相连接，大大释放了各种终端的实时通信能力。
+ 音视频会议
+ 在线教育
+ 音乐播放器
+ 照相机
+ 共享桌面
+ 录制
+ 即时通讯工具
+ 云端游戏
+ 实时人脸识别
+ P2P网络加速

## 应用WebRTC和传统客户端与服务器通信的区别
![传统模式](/img/post/webrtc/no-webrtc.png)
![应用WebRTC](/img/post/webrtc/use-webrtc.png)

## WebRTC优点
+ Google开源维护
+ 跨平台：Web，Android，iOS，Windows，MacOS，Linux
+ 实时传输：传输速度快，低延时，适合实时性要求较高的场景
+ 强大的音视频处理引擎，能力强大
+ 免插件：无需额外安装插件
+ 免费
+ 强大的打洞能力：WebRTC技术包含了使用STUN、ICE、TURN、RTP-over-TCP的关键NAT和防火墙穿透技术，并支持代理
+ 在Web端，主流浏览器均支持：Chrome，Safari，FireFox

# WebRTC核心
+ 音视频引擎：OPUS、VP8 / VP9、H264
+ 传输层协议：底层传输协议为UDP
+ 媒体协议：SRTP / SRTCP（对媒体数据的封装与传输控制协议）
+ 数据协议：DTLS / SCTP（流控制传输协议，提供类似TCP的特性，SCTP可以基于UDP上构建，在WebRTC里是在DTLS协议之上）
+ P2P内网穿透：STUN / TURN / ICE / Trickle ICE
+ 信令（signaling）与SDP协商：HTTP / WebSocket / SIP、 Offer Answer模型

# WebRTC架构
![](/img/post/webrtc/webrtc-frame.png)

## 第一层：应用层
Web应用：视频会议，语音电话，远程教育等

## 第二层：网络API层，为上层提供服务
面向第三方开发者的WebRTC标准API（比如javascript），能让开发者更容易地开发WebRTC的网络应用。最新的标准化进程可见[W3C的WebRTC文档](https://www.w3.org/TR/webrtc/)
+ MediaStream：媒体数据流，音视频流
+ RTCPeerConnection：应用层调用接口的类
+ RTCDataChannel：非音视频数据传输，文字图片等
+ 丰富的[API](https://developer.mozilla.org/zh-CN/docs/Web/Api/webrtc_api)

## 第三层： C++编写的底层API
使浏览器厂商容易实现WebRTC标准的Web API，抽象的对数字信号过程进行处理。如RTCPeerConnection API是每个浏览器之间点对点连接的核心，RTCPeerConnection是WebRTC组件，勇于处理对等体之间流数据的稳定和有效通信。

## 第四层：会话管理层
一个抽象的session层，提供session建立和管理的功能。
+ 主要是开发者自定义实现 
+ 对于web应用，建议用WebSocket来管理信令（signaling）
+ 信令主要是用来转发会话双方的媒体信息和网络信息（比如端口，编码解码）

## 第五层：音视频引擎/传输层
此处有WebRTC的三个主要模块：

### Transport
WebRTC的传输层，涉及音视频的数据发送、接受、网络打洞等。
+ 打通网络
+ 多路复用
+ SRTP（安全的实时传输协议，用以音视频流传输）
+ STUN+TURN+ICE建立P2P的链接
+ 整个WebRTC通信是基于UDP的

### VoiceEngine
包含一系列多媒体处理的框架，包括从视频采集卡到网络传输端的整体解决方案
+ iSAC：Internet Speech Audio Codec
  - 针对VoIP和音频流的宽带和超宽带音频编解码器
  - WebRTC音频引擎默认编解码器
  - 采样频率：16khz（默认），24khz，32khz
  - 自适应速率：10kbit/s~52kbit/s
  - 自适应包大小：30~60ms
  - 算法延时：frame+3ms
+ ILBC：Internet Low Bitrate Codec
  - VoIP音频流的窄带语音编解码器
  - 采样频率：8khz
  - 20ms帧比特率为15.2kbps
  - 30ms帧比特率为13.33kbps
+ NetEQ For Voice
  - 针对音频软件实现的语音信号处理原件，能够快速且高解析度的适应不断变化的网络环境，确保音质优美且缓冲延迟最小。
  - 能够有效处理网络抖动和语音包丢失时对于因质量的影响
  - 核心技术，对提高VoIP质量有显著效果
+ AEC：Acoustic Echo Canceler
  - 回声消除器
  - 基于软件的信号处理元件，实时去除mic采集的回声
+ NR：Noise Reduction
  - 降噪消除
  - 基于软件的信号处理元件，消除VoIP某些背景噪音
### VideoEngine
视频处理引擎，包含一系列视频处理的整体框架，从摄像头采集视频到视频信息网络传输，再到显示视频的整个过程解决方案
+ VP8
  - 视频图像编解码器，WebRTC默认的视频引擎编解码器
  - 适合实时通信，是针对低延时设计的编解码器
+ Video Jitter Buffer
  - 视频抖动缓冲器
  - 降低由于视频抖动和视频信息报丢失带来的不良效果
+ Image  Enhancements
  - 图像质量增强
  - 对摄像头采集的图像进行处理（敏感度检测，颜色增强，降噪处理等），提升视频质量

## 第六层：自定义层
由各大浏览器开发商自定义实现。

# WebRTC两大功能

## MediaStream（媒体捕获）
允许访问输入设备（摄像头，麦克风），用以捕获媒体数据流。
![](/img/post/webrtc/media-capture.png)

+ 最底层是硬件设备，网上是音频和视频的捕获模块
+ 中间为音视频引擎。
  - 音频引擎负责音频采集和传输，具有降噪，回声消除等功能
  - 视频引擎负责网络抖动优化，传输编码优化
+ 再往上是C++的API，在这个API往上是浏览器提供的Javascript API

## RTCPeerConnection（媒体传输）
允许在两个浏览器之间直接通讯（将捕获的音视频流实时发送到另一个WebRTC点），建立两个对等点之间的连接。
![](/img/post/webrtc/media-transport.png)

+ 基于UDP基础上搭建
+ ICE、STUN、TURN 用于内网穿透, 解决了获取与绑定外网映射地址，以及 keep alive 机制
+ DTLS 用于对传输内容进行加密，可以看做是 UDP 版的 TLS，这一层对于看中安全的WebRTC很重要，是必须的。对于前端开发，用到的Javascript API必须是https或本地主机。
+ 信令不属于WebRTC标准
+ SRTP 与 SRTCP 是对媒体数据的封装与传输控制协议
+ SCTP 是流控制传输协议，提供类似 TCP 的特性，SCTP 可以基于 UDP 上构建，在 WebRTC 里是在 DTLS 协议之上
+ RTCPeerConnection 用来建立和维护端到端连接，并提供高效的音视频流传输
+ RTCDataChannel 用来支持端到端的任意二进制数据传输

# WebSocket与WebRTC的区别
说了那么多，最后我们再来对比下与WebSocket技术的一些不同之处：
## 场景不同
+ WebSocket是终端与服务端之间的全双工通信
+ WebRTC是终端与终端之间的全双工通信。
## 协议不同
+ WebSocket是TCP协议
+ WebRTC是UDP协议
## 实时性不同
+ WebSocket延迟高
+ WebRTC延迟低（直连）
## 传输路径不同
+ WebSocket需要经过服务器，占用服务器带宽
+ WebRTC是P2P方式，（媒体数据）不经服务器，不占服务器带宽