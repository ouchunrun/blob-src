---
title: RTCPeerconnections 流监听事件
date: 2018-11-16
tags: [WebRTC, stream, track] 
---

### 流或track增加时的事件监听
1、onaddstream

- type： MediaStreamEvent 
- remote peer 在当前连接添加新的MediaStream时，会触发local peer的该事件。

<!--more-->

2、onaddtrack

- 当任何类型的track添加到媒体流时会触发该事件。
- 当浏览器向stream添加一个新的track。比如 RTCPeerConnection renegotiated，
  或a stream being captured using HTMLMediaElement.captureStream() gets a new set of tracks because the media element being captured loaded a new source.

3、ontrack

- type: RTCTrackEvent
- 当一个新的MediaStreamTrack被创建，并且RTCRtpReceiver相关联时会触发该事件。

> ps: 都是对接受到的stream或track进行监听，本端添加不会触发自己的上述事件




### onremovestream

- type： MediaStreamEvent 
- 当 MediaStream 删除时触发。


### onremovetrack

- 当任何类型的track删除时会触发该事件。
- 比如 RTCPeerConnection renegotiated，或a stream being captured using HTMLMediaElement.captureStream() 
  gets a new set of tracks because the media element being captured loaded a new source.

### onended

- 当track不提供任何数据流时触发：包括到达的媒体输入结束、用户撤销所需权限、删除源设备或远程对等端终止连接。

ps: 
1、onaddstream和onremovestream接口的替换，和unified-plan没有关系，因为无法追溯ontrack最低在那个版本能够支持，
所以选择在这个（解决unified-plan问题）节点替换掉。

2、onaddstream使用ontrack进行替换。但是onremovestream在peerConnetion上暂时没有可替换的方法，只能使用onremovetrack对每一个track进行绑定。





