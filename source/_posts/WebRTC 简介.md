---
title: WebRTC 简介
date: 2019-3-30
tags: [webrtc] 
---


<!--more-->

# 一、introduction
---
**WbRTC**是Web Real-Time Communication的缩写，顾名思义，也就是web实时通讯技术，它允许网络应用或站点，**在不借助中间媒介、不安装任何插件或者第三方软件的情况下，建立浏览器之间P2P链接，实现视频流或者其他任意数据的传输。**而Web开发者也无需关注多媒体的数字信号处理过程，只需编写简单的Javascript程序调用相应的接口即可实现。


WebRTC 是一组开放的 API，用于实现实时音视频通话并且已成为 W3C 标准草案，目前是WebRTC 1.0版本，**Draft** 状态。

<!--more-->

## 相关概念 
- **SDP(Session Description Protocol)**
SDP是一种会话描述协议，用来描述双方的IP地址和端口号，通信所使用的带宽，会话的名称、标识符、激活时间，双方所要传输的媒体类型（视频、音频、文本）、媒体格式等等。该协议仅包含所要传递的媒体的描述信息，而不直接传递媒体内容。 
- **ICE（Interactive Connectivity Establishment）** 
ICE是一种以UDP为基础用于实现穿越NAT网管或者防火墙的协议。 
- **TURN&&STUN** 
两种协议都是用来明确自己的外网地址的，差别是如果要服务器辅助进行数据交换则设置TURN服务器，不需要则设置STUN服务。

# 二、兼容性
---
目前支持 webrtc 的浏览器有 Chrome Firefox Opera，IE、safari 等不支持。
*这些不支持的浏览器可以使用插件，尽管这似乎违背了 webrtc 的初衷。*

实时查看WebRTC在浏览器中的支持情况： [http://caniuse.com/#search=webRTC](http://caniuse.com/#search=webRTC)


# 三、WebRTC 系统包含的元素
---
WebRTC系统所包含的典型元素：

- Web服务器
- 运行于各种设备和操作系统之上的浏览器
- 台式机
- 平板电脑
- 手机和其他服务器
- PSTN(公用交换电话网)门户
- 以及其他互联网通信终端
WebRTC支持上述所有设备之间的通信。


# 四、WebRTC三个API
---
##  1、MediaStream（又称getUserMedia）
通过MediaStream的API能够通过设备的摄像头及话筒获得视频、音频的同步流，以及其他可能大轨道。

### 用法
```
navigator.mediaDevices.getUserMedia(constraints);
```
一般来说，`navigator.mediaDevices.getUserMedia`用法：

```
navigator.mediaDevices.getUserMedia(constraints)
.then(function(stream) {
  /* use the stream */
  /* handleSuccess  成功回调函数*/
})
.catch(function(err) {
  /* handle the error */
 /* handleError   失败处理函数*/
});
```

`getUserMedia`接受三个参数（约束条件、成功回调函数、失败回调函数），`mediaDevice.getUserMedia`只接受一个参数，其他处理函数是以.then和.catch来实现的。



**注意：~~navigator.getUserMedia~~此方法已经被`navigator.mediaDevices.getUserMedia`取代。**

---
### 三个参数
---
- **(1)constraints**：约束项
    - **属性**：约束项可以具有下面这两个属性中的一个或两个：
        - 1.audio—表示是否需要音频轨道
        - 2.video—表示是否需要视频轨道
            - **分辨率**：width、height
            - **关键字**：min、max、exact、ideal
            - **获取移动设备的前置或者后置摄像头**
        使用视频轨道约束的facingMode属性。可接受的值有：user（前置摄像头），environment（后置摄像头），left和right。
            - **帧速率**：frameRate
            - **使用特定的网络摄像头或者麦克风。**

- **(2)handleSuccess**：成功回调函数
如果调用成功，传递给它一个流对象

- **(3)handleError**：失败回调函数
如果调用失败，传递给它一个错误对象

---


### 兼容浏览器的getUserMedai的写法
---
```
var getUserMedia = (navigator.getUserMedia ||
                    navigator.webkitGetUserMedia ||
                    navigator.mozGetUserMedia || 
                    msGetUserMedia);
```

### 检测webRTC的可行性
---
主要从getUserMedia和webRTC本身来入手，示例代码如下：

```
function detectWebRTC() {
  const WEBRTC_CONSTANTS = ['RTCPeerConnection', 'webkitRTCPeerConnection', 'mozRTCPeerConnection', 'RTCIceGatherer'];

  const isWebRTCSupported = WEBRTC_CONSTANTS.find((item) => {
    return item in window;
  });

  const isGetUserMediaSupported = navigator && navigator.mediaDevices && navigator.mediaDevices.getUserMedia;

  if (!isWebRTCSupported || typeof isGetUserMediaSupported === 'undefined' ) {
    return false;
  }

  return true;
}
```
>但这些兼容写法在实际开发的时候是不需要的，Adapter.js库里封装好了兼容性检测方法及其他方法。Adapter.js主要就是用来解决兼容性的。[Adapter.js](https://webrtc.github.io/adapter/adapter-latest.js)
Google维护的函数库[adapter.js](https://apprtc.appspot.com/js/adapter.js)，就是用来抽象掉浏览器之间的差异，这个一直在更新。



---

### HTMLMediaElement.srcObject
---
接口的`srcObject`属性`HTMLMediaElement`设置或返回作为媒体源的对象`HTMLMediaElement`，该对象可以是a mediaStream、a mediaSource、a blog 、a file。

#### 用法
```
var mediaStream = HTMLMediaElement .srcObject
 HTMLMediaElement .srcObject = mediaStream
```



### URL.createObjectURL()

注意：使用 一个~~`MediaStream`~~对象作为此方法的输入正在被弃用。这个方法正在被讨论是否应该被移除. 所以，你应当在你使用*`MediaStream`时*避免使用这个方法，而用*`HTMLMediaElement.srcObject()` 替代*.



---
## 2、 RTCPeerConnection
---

**RTCPeerConnection**的作用是在浏览器之间建立数据的“点对点”**（peer to peer）**通信，也就是将浏览器获取的麦克风或摄像头数据，传播给另一个浏览器。这里面包含了很多复杂的工作，比如**信号处理、多媒体编码/解码、点对点通信、数据安全、带宽管理**等等。

不同客户端之间的音频/视频传递，是不用通过服务器的。但是，两个客户端之间建立联系，需要通过服务器。服务器主要转递两种数据。

- 通信内容的元数据：打开/关闭对话（session）的命令、媒体文件的元数据（编码格式、媒体类型和带宽）等。
- 网络通信的元数据：IP地址、NAT网络地址翻译和防火墙等。

>现WebRTC 没有指定具体的信令协议，媒体协商采用了 SDP 协议。（ps：在我们的 webrtc client 中使用 sip 信令）



---

### UDP和TCP的区别
---
**（Transmisson Control Protocol，TCP）传输控制协议**，它保证如下几点：

- 任何送出的数据都有送达的确认
- 任何未送达接收端的数据会被重传并停止发送更多的数据。
- 数据是唯一的，接收端不会有重复的数据


**（User Datagram Protocol， UDP）用户数据报协议**，以下是他不保证的事情：

- 不保证数据发送或接收扥先后顺序。
- 不保证每一个数据包都能都传送到接收端吗；一些数据可能在半路丢失。
- 不跟踪每一个数据包的状态，即使接收端有数据丢失也会继续传输。

**TCP的这些限制使得WEBRTC开发者选择UDP作为传输协议**，webRTC的音频、视频和数据在浏览器间的传输不需要最可靠但需要最快。这意味着允许丢失帧，也就是说这类应用来说UDP是一个更好的选择。但不意味着webRTC就不使用TCP协议！

如果UDP传输失败， ICE 会尝试TCP: 首先是 HTTP, 然后才会选择 HTTPS。如果直接连接失败，通常因为企业的NAT转发和防火墙，也就是ICE使用了第三方中转。这样一来，ICE首先使用STUN和UDP和直接连接端点，失败之后返回TURN服务器，表达式‘finding cadidates’指向找到的网络接口和端口。


### 没有服务器的RTCPeerConnection
请求和被请求都在同一个页面，它们可以直接通过RTCPeerConnection对象在页面上交换信息，而不需要使用中介的信号机制。WebRTC samples用的是这种方式，但一般来说，实际开发的时候，这种方式不会用到。只是在学习和测试的时候用的比较多【个人而言】。

### 有服务器的RTCPeerConnection

在现在的世界里，WebRTC需要服务器，但是服务器配置非常简单，步骤如下：

*   用户找到对方并交换双信息，比如名字。
*   WebRTC客户端应用交换网络信息。
*   两个端交换多媒体数据信息。
*   WebRTC客户端遍历[NAT 网关](http://en.wikipedia.org/wiki/NAT_traversal)和防火墙。

其他方面，WebRTC需要四个类型的服务器端功能：

*   用户连接和通信
*   信号量
*   NAT/防火墙转发
*   如果通信失败再次发送

#### 服务器主要转递两种数据。

- **通信内容的元数据**：打开/关闭对话（session）的命令、媒体文件的元数据（编码格式、媒体类型和带宽）等。
- **网络通信的元数据**：IP地址、NAT网络地址翻译和防火墙等。

WebRTC协议没有规定与服务器的信令通信方式，因此可以采用各种方式，比如`WebSocket（HTTP、数据通道`等）。通过服务器，两个客户端按照Session Description Protocol（SDP协议）交换双方的元数据。

 



---

### 信号: session控制，网络和多媒体信息
---
WebRTC使用`RTCPeerConnection`进行两个浏览器之间的数据流的通信，但是也需要一种机制来协调沟通控制信息，这一个过程叫做信号。信号的方法和协议不是WebRTC指定的，而是RTCPeerConnection API的一部分。

信号用来交换以下三个类型的信息：

- **Session控制信息**：用来初始化或是关闭通信，并报告错误。
- **网络配置**：本机的IP和端口等配置。
- **媒体功能**：编码解码器配置。 


---
### 建立webRTC的基本会话流程
---

singing server信令服务器：用来开始和结束通话，即开始视频、结束视频这些操作指令和通信数据的交换等。

注意：使用turn服务器的时候，客户端AB不直接通信，而是分别与turn通信从而建立链接。
使用stun服务器的时候，因为客户端B无法访问客户端A的内网IP，所以A通过stun服务器获得自己的外网IP再发送给B，从而建立链接。

![webRTC基本会话流程](http://upload-images.jianshu.io/upload_images/8154321-d6a351a8b0cc7259.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




1. 首先双方都建立一个RTCPeerConnection的实例，其中一方（称为offer方）用RTCPeerConnection.createOffer()创建一个会话描述sessionDescription，该会话描述包含SDP报文信息和该sessionDescription的类型(type) 

2. 接下来调用RTCPeerConnection.setLocalDescription()方法将本地的localDescription设置为刚才创建的sessionDescription。之后将创建的sessionDescription发送给对方（称为answer方），发送方式没有规定，可以通过服务器中转，可以通过IM软件发送(这里使用WebSocket信令服务器)。 

3. answer端接收到sessionDescription后调用RTCPeerConnection. setRemoteDescription方法设置,然后调用RTCPeerConnection. createAnswer方法产生自己的sessionDescription。 

4. 再将创建的sessionDescription发送给offfer方，同样发送方式没有规定。offer方接收到sessionDescrip后调用RTCPeerConnection. setRemoteDescription方法设置，这样双方的SDP信息就交换完成了。
 
5. 在完成SDP的交换后双方还要交换ICE candidate信息。双方首先设置RTCPeerConnection.onicecandidate回调函数，当candidate可用时，双方中的一方将所有icecandidate发送给对方，发送方式同样没有规定，接收方调用RTCPeerConnection.addIceCandidate方法接收candidate信息。经过这些步骤后双方连接就建立完成了。

注意：~~RTCPeerConnection.addStream~~ 被 `RTCPeerConnection.addTrack`取代;








---
## 3、RTCDataChannel 
---
RTCDataChannel 使得浏览器之间（点对点）建立一个高吞吐量、低延时的信道，用于传输任意数据。表示连接的两个对等端之间的 **双向数据通道**。

我们可以使用`channel = pc.createDataCHannel("someLabel");`来在`PeerConnection`的实例上创建Data Channel，并给与它一个标签

---

### 最后，需要知道的内容

---

- WebRTC已经纳入HTML5标准
- 目前支持webrtc的浏览器有 Chrome Firefox Opera，IE不支持~
- WebRTC没有指定具体的信令协议，具体的信令协议留给应用程序实现。
- webRTC使用JSEP协议建立会话，什么是JSEP后面说
- WebRTC采用ICE实现NAT穿越
- WebRTC客户端之间可以进行点对点的媒体传输。



---
### 加密与安全
---

webRTC运行时，对于所有协议的实现，都会强烈执行加密功能。这意味着浏览器间的每一个对等连接，都自动处于高的安全级别中。所使用的加密技术都应该满足对等应用的以下几个要求：

- 信息在传输过程中被窃取，无法进行读取
- 第三方不能够伪造信息，是消息看上去像是链接者发送的
- 消息在传输给另一方时不能进行编辑
- 加密算法的快速性，一支持客户端之间的最大宽带

**为了满足各方面的需求，我们的协议使用的是DTLS**

-  **安全传输层协议（TLS）**用于在两个通信应用程序之间提供保密性和数据完整性。
该协议由两层组成： TLS 记录协议（TLS Record）和 TLS 握手协议。
TLS的流行是因为直接嵌入到应用层和传输层，是的很少甚至不需要改变应用本身的逻辑二使应用变得安全。使用TLS的唯一缺点就是它必须基于TCP的应用，但我们的webRTC协议并不是在其基础上工作的，

- **DTLS(Datagram Transport Layer Security)**即数据包传输层安全性协议。
DTLS源于希望能有一个像TLS一样简单易用的协议，但拥有UDP传输层的功能。它吸取了TLS协议许多相同的概念，并增加了对UDP的支持。

到这里最大的收获，你需要知道数据通道和webRTC应用中，数据是安全的。



---
参考资料：
[1][通过WebRTC实现实时视频通信（三）](http://gbtags.com/gb/share/3929.htm)
[2][webRTC实战总结](https://www.ctolib.com/topics-129397.html)
[3][WebRTC API](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)
[4][使用WebRTC搭建前端视频聊天室——入门篇](https://segmentfault.com/a/1190000000436544)
[5][webrtc进阶-信令篇-之三：信令、stun、turn、ice](https://blog.csdn.net/fireroll/article/details/50780863)
[6][getUserMedai()视频约束](http://webrtc.org.cn/getusermedia-video-constraints/)
[7][WebRTC1-原理探究](https://blog.csdn.net/future_todo/article/details/52689420)

...还有很多

