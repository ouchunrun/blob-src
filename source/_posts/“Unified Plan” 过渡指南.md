---
title: “Unified Plan” 过渡指南
date: 2018-10-23
tags: [WebRTC, SDP] 
---

## Background

WebRTC规范多年来不断发展，在API等各方面都做了很多更改。本文主要概述“Plan B” 和 “Unified Plan”之间的差异，以帮助开发者在SDP的转换上做好准备。

如果应用程序执行以下操作之一，那就应该小心：

- 具有多个音轨或多个视频轨道。
- 依赖于本地和远程 track IDs 匹配的假设。
- Munges SDP，使用 MCUs 或 SFUs 或以其他方式修改SDP或使用非Chrome生成的SDP。


### SDP格式的控制

可以通过给RTCPeerConnection设置sdpSemantics参数来改变默认的SDP使用格式，此标志仅限Chrome; 如果浏览器无法识别参数，则会忽略它。如下代码：

      // Use Unified Plan or Plan B regardless of what the default browser behavior is.
      new RTCPeerConnection({sdpSemantics:'unified-plan'});
      new RTCPeerConnection({sdpSemantics:'plan-b'});

<!--more-->

除非你的应用程序为（unified-plan和plan-b）两种情况做好准备，否则建议明确地设置sdpSemantics来避免chrome的默认行为发生变化带来的影响。

### 使用chrome://flags控制SDP默认行为

Chrome M71：[chrome://flags](chrome://flags/) contain the experiment `WebRTC: Use Unified Plan SDP Semantics by default`。如果启用该选项或者在启动Chrome的时候传递如下命令行参数：

    --enable-features=RTCUnifiedPlanByDefault

除非另行指定SDP格式参数，否则将以 unified-plan 的方式构建RTCPeerConnections。

也就是说，在Chrome上面，你可以通过不指定sdpSemantics的方式来运行应用程序。

### 特征检测

M69 增加了 `transceivers` ，但只有在使用“unified-plan”时才支持它们。
检查`addTransceiver()` 是否在默认构造的RTCPeerConnection上抛出异常是一种检查“unified-plan”是否为默认的特征检测方法。 
在M70中，添加了`getConfiguration()`，允许您显式检查sdpSemantics的值，类似于chrome：// webrtc-internals /中已经显示的值。 


## Plan-b 和 Unified-Plan 的区别

### SDP 有何区别

**Plan B:**

- 一个`m= section`用于audio，一个`m= section`用于video，每个媒体流带有各自的媒体流识别属性，eg:mid。
- 如果offer里面包含同类型的tracks，对应的m行会有多个`a=ssrc`属性。

**Unified Plan**

- 一个m行对应一个 sending/ or receiving track，
- 每个m行通过mid来标识
- 如果有多个tracks，那么会创建多个m行。

下面分别是Plan B 和 Unified Plan 发送两个audio tracks所对应的offer:

Plan B offer:

    ...
    a=group:BUNDLE audio
    a=msid-semantic: WMS stream-id-2 stream-id-1
    m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
    ...
    a=mid:audio
    ...
    a=rtpmap:103 ISAC/16000
    ...
    a=ssrc:10 cname:cname
    a=ssrc:10 msid:stream-id-1 track-id-1
    a=ssrc:10 mslabel:stream-id-1
    a=ssrc:10 label:track-id-1
    a=ssrc:11 cname:cname
    a=ssrc:11 msid:stream-id-2 track-id-2
    a=ssrc:11 mslabel:stream-id-2
    a=ssrc:11 label:track-id-2

Unified Plan offer：
    
    ...
    a=group:BUNDLE 0 1
    a=msid-semantic: WMS
    m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
    ...
    a=mid:0
    ...
    a=sendrecv
    a=msid:- <track-id-1>
    ...
    a=rtpmap:103 ISAC/16000
    ...
    a=ssrc:10 cname:cname
    a=ssrc:10 msid: track-id-1
    a=ssrc:10 mslabel:
    a=ssrc:10 label:track-id-1
    m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
    ...
    a=mid:1
    ...
    a=sendrecv
    a=msid:- track-id-2
    ...
    a=rtpmap:103 ISAC/16000
    ...
    a=ssrc:11 cname:cname
    a=ssrc:11 msid: track-id-2
    a=ssrc:11 mslabel:
    a=ssrc:11 label:track-id-2

注意事项：

- Plan B里面使用了"a=mid:audio"，并且两个tracks在同一个m=audio通过srrc指明。
- Unified Plan，"a=mid:0"用于第一个音轨 ，"a=mid:1" 用于第二个音轨。
- Chrome 的Unified Plan使用时，"a=ssrc"伴随了msid、mslabel和lable等属性，这几个属性只是为了向后兼容，而不是unified-plan必须的。
    
    
### RTCRtpTransceiver API和行为的变化
    
- SDP中，一个Transceiver代表一个m行，因为m行既可以用于sending，也可以用于receiving,而transceiver由发送器和接收器组成。
  收发的方向可以在“inactive”, “sendonly”, “recvonly” 和 “sendrecv”，之间根据需求切换。


- 将 track 使用addTrack() 或addTransceiver() 的对等连接时，收发器创建（或重复使用）并将轨道附加到发件人。
因为收发器可能是用于接收，始终创建接收轨道并将其连接到接收器。不像发送方，接收方的跟踪始终存在且无法替换，但默认情况下静音。

- setLocalDescription()和setRemoteDescription()久地将transceivers连接到m行，通过将transceiver.mid的值设置为m = section的mid。
这是一个可以的字符串标识符，两个endpoints用来关联transceivers，以便知道哪个是哪个。


### Local 和 Remote 的 Track IDs 不匹配

**对于 Plan B：**

- 为每一个被添加的local track 创建了sender，为每一个协商的remote track 都创建了receiver。本地端和远程端的track id相互匹配。

**对于 Unified Plan：**

- senders 和 receivers 是成对产生的。`transceiver.receiver.track`可能在remote SDP提供track之前就创建了。所以，在RTCPeerConnection.ontrack触发的时候，并不能保证已经有一个和发送端所匹配的ID。
- 此外，因为addTransceiver() 和 replaceTrack()，同一个track可能被发送多次。Track IDs具有误导性。取而代之的，transceiver.mid可以用来关联local 和 remote tracks。你不得不在知道mid之前设置setLocalDescription(answer)。
    
    
    
### Remote Tracks Are Never Truly “Removed”

Plan B 里面：

- RTCPeerConnection.removeTrack()删除了sender and track，在协商的时候，对端也会删除对应的receiver and track。

Unified Plan 里面：

- removeTrack()改变了transceiver的收发方向，取消sender’s track。协商期间，transceiver’s改变收发方向，同时mute the track，但是never removed。并且改track还可能再次复用。

### MediaStreamTrack.onended No Longer Fires

因为remote tracks可能被muted，但是永远不会被删除，MediaStreamTrack.onended 不会再被触发。相应的，使用 MediaStreamTrack.onmute来检查track是否被删除。


### Legacy APIs

RTCPeerConnection.addStream() 和 removeStream() 不是标准的 APIs。由于向后兼容的原因，它们在RTCPeerConnection.addTrack()和removeTrack()之上填充。
在Unified Plan中会继续使用，但是使用标准的APIs会更好。

同样地，在RTCPeerConnection上触发的事件"addstream"和"removestream"(使用事件处理程序"onaddstream"和"onremovestream")是标准中没有的遗留api。
由于向后兼容的原因，在M71的Unified Plan中支持它们，但是，RTCPeerConnection.ontrack, MediaStreamTrack.onmute 和 MediaStream.onaddtrack/onremovetrack更好。







