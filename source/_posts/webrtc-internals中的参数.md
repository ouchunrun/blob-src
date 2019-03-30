---
title: webrtc-internals中的参数
date: 2019-3-30
tags: [webrtc, webrtc-internals] 
---


当你想要找到你WebRTC产品中的问题时，webrtc-internals是一个非常棒的工具，因为你需要用它测试WebRTC以及debug，或者你需要对你的配置进行微调。

<!--more-->



![webrtc-internals](https://webrtc.org.cn/wp-content/uploads/2017/01/internals1-300x158.jpg)


If you open chrome://webrtc-internals while in an active WebRTC session, you will immediately see the API trace:

![43118ddde8a0a3a55e8f7c2c24bfb832.png](https://i.loli.net/2019/03/30/5c9f15340eb53.png)

可以点击任何一个，展开看看它里面的参数。

- **addStream**：如果调用此方法，则Javascript代码已将MediaStream添加到peerconnection。您可以看到流的ID以及音频和视频轨道。

- **onAddStream**：显示正在添加的远程流，包括音频和视频轨道ID，它在setRemoteDescription调用和setRemoteDescriptionOnSuccess回调之间调用

- **createOffer**：显示对此API的任何调用，包括offerToReceiveAudio，offerToReceiveVideo或iceRestart等选项。

- **createOfferOnSuccess**：显示createOffer调用的结果，包括类型（显然应该是'offer'）和由此产生的SDP。也可以调用createOfferOnFailure来指示错误，但这种情况非常罕见

- **createAnswer和createAnswerOnSuccess和createAnswerOnFailure类似，但没有其他选项**

- **setLocalDescription**：显示setLocalDescription调用中使用的类型和SDP。如果您在createOffer和setLocaldescription之间进行任何SDP调整，您将在此处看到此信息。这会导致setLocalDescriptionOnSuccess或setLocalDescriptionOnFailure回调，显示任何错误。这同样适用于setRemoteDescription及其回调，setRemoteDescriptionOnSuccess和setRemoteDescriptionOnFailure


- **onRenegotiationNeeded**：是onnegotiationneeded事件的旧chrome内部名称。如果您的应用使用此功能，您可能需要查找它


- **onSignalingStateChange**：显示调用setLocalDescription和setRemoteDescription后信令状态的变化。

- **iceGatheringStateChange**：它会告诉你候选地址ICE采集的状态。如果有ICE候选人要收集，在setLocalDescription之后，它将改为gathering状态。

- **onnicecandidate**：事件显示所有候选人收集在一起，其中包含m-line和MID的信息。同样，addIceCandidate方法显示来自另一方的信息。通常，您应该看到两种事件类型。

- **oniceconnectionstate**：是最重要的事件处理程序之一。它告诉您对等连接是否成功。

---

![0465be422d36482511f9e5540c5511e2.png](https://i.loli.net/2019/03/30/5c9f153410d0f.png)

 上图WebRTC状态说明： 
**最上面一排依次是：三个peerconnection API（分别代表音频、视频、演示）、MediaStream API。**



二、PeerConnection状态说明：

**1、最上面的一行：表示peerconnection，大括号中的内容是peerconnection的配置参数**

**2、左上角的Event：peerconnection的内部接口，对应的是JSEP协议** 

- createOfferOnSuccess：浏览器生成的本地sdp
- signalingstatechange：peerconnection的状态
- icegatheringstatechange：本地ice后续地址收集的状态
- icecandidate：候选地址的收集接口
- iceconnectionstatechange：ICE连通性检查的状态


**3、右上角的EventStats Tables：是媒体通道的具体信息，数据实时更新** 

- bweforvideo (VideoBwe)：带宽相关信息
- Conn-video-1-0 (googCandidatePair)：RTP相关信息
- Conn-video-1-1 (googCandidatePair)：RTCP相关
- ssrc_689353421_recv (ssrc)：解码相关信息
- ssrc_495012973_send (ssrc)：编码相关信息


**4、左下角的部分：也是媒体通道的具体信息，以表格的形式表示，与右上角的EventStats Tables一一对应** 

- Stats graphs for bweforvideo：带宽相关信息
- Stats graphs for Conn-video-1-0：RTP相关信息
- Stats graphs for Conn-video-1-1：RTCP相关
- Stats graphs for ssrc_689353421_recv (video)：解码相关信息
- Stats graphs for ssrc_495012973_send (video)：编码相关信息



---

参考

[1、【webrtc-internals中的参数的真正含义是什么】](http://webrtc.org.cn/parameters-webrtc-internals/)
















