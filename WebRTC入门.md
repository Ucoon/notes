# WebRTC是什么

>WebRTC，名称源自网页即时通信（Web Real-Time Communication）的缩写，是一个支持网页浏览器进行实时语音对话或者视频对话的API，它允许网络应用或者站点，在不借助中间媒介的情况下，建议浏览器之间点对点（Peer-to-Peer）的连接，实现视频流音频流或者其他任意数据的传输。WebRTC既是API也是协议，它是两个WebRTC Agen协商双向安全实时通信的一组规则，开发人员可以通过WebRTC API使用WebRTC协议。

解释：可以用HTTP和Fetch API之间的关系作为类比。WebRTC协议就是HTTP，而WebRTC API就是Fetch API。

# 为什么学习

- 开放标准
- 多种实现
- 在浏览器中使用
- 强制加密
- NAT穿透
- 复用现有技术
- 拥塞控制
- 亚秒级延迟

# WebRTC协议是一组其他技术的集合体

1. 信令(Signaling)：peer如何在WebRTC中找到彼此


2. 使用STUN/TURN进行连接和NAT穿透
3. 使用DTLS和SRTP加密传输层
4. 通过RTP和SCTP进行点对点通信

![WebRTC是一系列协议的集合](https://webrtcforthecurious.com/zh/images/01-webrtc-agent.png)

# WebRTC(API)如何工作

1. `new RTCPeerConnection`

   `RTCPeerConnection`是最顶层的“WebRTC会话”。它包含上述所有协议。

2. `addTrack`

   `addTrack`创建一个新的RTP流，并将为这个流生成一个随机的SSRC（Synchronization Source/同步源）。然后`createOffer`将生成会话描述符，这个流将被加入其中的媒体部分，每次调用`addTrack`都会创建一个新的SSRC和对应的媒体部分

3. `createDataChannel`

   如果没有SCTP关联存在，`createDataChannel`将创建一个新的SCTP流，默认情况下，SCTP是不启用的，只有一方请求数据通道时才启动。

   在DTLS会话建议之后，SCTP关联将立即通过ICE发送数据包，并使用DTLS加密

4. `createOffer`

   `createOffer`生成本地状态的会话描述，以与远端Peer共享

5. `setLocalDescription`

   `setLocalDescription`提交所有请求的更改。在此调用之前，`addTrack`，`createDataChannel`和类似调用都是临时的，调用`setLocalDescription`时，使用由`createOffer`生成的值。

   通常，在此调用之后，你会将offer发送给远端Peer，他们将调用`setRemoteDescription`，将此offer设入。

6. `setRemoteDescription`

   收到远端Peer发来的offer之后，我们将通过`setRemoteDescription`通知本地Agent。这就是使用JavaScript API传递“信令”的方式。双方都调用过`setRemoteDescription`后，WebRTC Agent现在拥有足够的信息来开始进行点对点（P2P）通信！

7. `addIceCandidate`

   `addIceCandidate`允许WebRTC Agent随时添加更多

8. `ontrack`

9. `oniceconnectionstatechange`

10. `onstatechange`

# 信令

## 什么是WebRTC信令

信令是使呼叫成为可能的初始引导程序。交换信令消息后， WebRTC Agent才可以直接相互通信。信令通常发生在“out-of-band”，也就是说，应用通常不使用WebRTC本身来交换信令消息。在连接的peer中，任何适用发送消息的架构（比如`WebSocket`）均可被用于传递SDP信息。

## WebRTC信令如何工作

信令使用现有的协议SDP（Session Description Protocol，会话描述协议），两个WebRTC Agent会将建立连接所需的所有状态通过此协议来分享。

## 会话描述协议（SDP）

>SDP（Session Description Protocol）是一种通用的会话描述协议，主要用来描述多媒体会话，用途包括会话声明、会话邀请、会话初始化等。
>
>WebRTC主要在连接建议阶段用到SDP，连接双方通过信令服务交换会话信息，包括音视频编解码器(codec)，主机候选地址、网络传输协议等。

每个SDP消息均由key/value对组成，并包含“media sections（媒体部分）”列表。两个WebRTC Agent交换的SDP所包含一些详细信息，如：

- Agent可供外部访问的（候选的）IP和端口
- Agent希望发送多少路音频和视频流
- Agent支持哪些音频和视频编解码器
- 连接时需要用到的值（`uFrag`/`uPwd`）
- 加密传输时需要用到的值（证书指纹）

## SDP和WebRTC如何协同工作

### 什么是Offer和Answer

WebRTC使用Offer和Answer模型，即一个WebRTC Agent发出“Offer”以开始呼叫，如果另一个WebRTC Agent愿意接受“Offer”的内容，它会响应“Answer”。

### 用于发送和接收的收发器（Transceivers）

收发器是WebRTC中特有的概念，它的作用是将“媒体描述”暴露给JavaScript API。每个媒体描述都将成为一个收发器。每次创建收发器时，都会将新的媒体描述添加到本地会话描述中。

# 连接



# Get Started

参考**[Firebase + WebRTC代码实验室](https://webrtc.org/getting-started/firebase-rtc-codelab)**



