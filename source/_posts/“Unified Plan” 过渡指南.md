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
检查`addTransceiver()` 是否在默认构造的RTCPeerConnection上抛出异常是一种检查“unified-plan”是否为默认行为的特征检测方法。 
在M70中，添加了`getConfiguration()`，允许显式检查sdpSemantics的值，类似于chrome：// webrtc-internals /中已经显示的值。 


## Plan-b 和 Unified-Plan 的区别

### SDP 有何区别

**Plan B:**

- 一个`m= section`用于audio，一个`m= section`用于video，每个媒体流带有各自的媒体流识别属性，eg:mid。
- 如果offer里面包含同类型的tracks，对应的m行会有多个`a=ssrc`属性。

**Unified Plan**

- 一个m行对应一个 sending/ or receiving track，
- 每个m行通过mid来标识
- 如果有多个tracks，那么会创建多个m行。

下面分别是Plan B 和 Unified Plan 发送三个audio tracks所对应的offer:

Plan B offer:

    ...
    a=group:BUNDLE audio video
    a=msid-semantic: WMS HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm uQXvAPEH23ysg7WvwxKVmyzdZchzQpwGYTd9
    
    m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
    c=IN IP4 0.0.0.0
    a=rtcp:9 IN IP4 0.0.0.0
    a=ice-ufrag:suJZ
    a=ice-pwd:JLjmrwlB12Tr9MJb6B/WGQwy
    a=ice-options:trickle
    a=fingerprint:sha-256 80:DD:A7:A8:D4:16:CA:9B:1F:79:9A:0D:7B:05:EA:E7:35:FD:11:6F:B8:69:C0:57:0F:77:2B:D2:AE:0B:02:E3
    a=setup:actpass
    a=mid:audio
    a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
    a=sendrecv
    ...
    a=ssrc:3174753737 cname:u6uhaN/KTBYk6jwt
    a=ssrc:3174753737 msid:uQXvAPEH23ysg7WvwxKVmyzdZchzQpwGYTd9 f595523e-2fcd-400f-992e-0a127753952f
    a=ssrc:3174753737 mslabel:uQXvAPEH23ysg7WvwxKVmyzdZchzQpwGYTd9
    a=ssrc:3174753737 label:f595523e-2fcd-400f-992e-0a127753952f
    a=ssrc:1773101601 cname:u6uhaN/KTBYk6jwt
    a=ssrc:1773101601 msid:HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm bbf350aa-714c-40f2-a646-6d235d154086
    a=ssrc:1773101601 mslabel:HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm
    a=ssrc:1773101601 label:bbf350aa-714c-40f2-a646-6d235d154086
    
    m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 102 122 127 121 125 107 108 109 124 120 123 119 114
    c=IN IP4 0.0.0.0
    a=rtcp:9 IN IP4 0.0.0.0
    a=ice-ufrag:suJZ
    a=ice-pwd:JLjmrwlB12Tr9MJb6B/WGQwy
    a=ice-options:trickle
    a=fingerprint:sha-256 80:DD:A7:A8:D4:16:CA:9B:1F:79:9A:0D:7B:05:EA:E7:35:FD:11:6F:B8:69:C0:57:0F:77:2B:D2:AE:0B:02:E3
    a=setup:actpass
    a=mid:video
    ...
    a=sendrecv
    ...
    a=ssrc-group:FID 1133973908 3925917696
    a=ssrc:1133973908 cname:u6uhaN/KTBYk6jwt
    a=ssrc:1133973908 msid:HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm 27f35f0c-e1fa-4173-b9e1-ca1030987362
    a=ssrc:1133973908 mslabel:HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm
    a=ssrc:1133973908 label:27f35f0c-e1fa-4173-b9e1-ca1030987362
    a=ssrc:3925917696 cname:u6uhaN/KTBYk6jwt
    a=ssrc:3925917696 msid:HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm 27f35f0c-e1fa-4173-b9e1-ca1030987362
    a=ssrc:3925917696 mslabel:HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm
    a=ssrc:3925917696 label:27f35f0c-e1fa-4173-b9e1-ca1030987362

Unified Plan offer：
    
    ...
    m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
    c=IN IP4 0.0.0.0
    a=rtcp:9 IN IP4 0.0.0.0
    a=ice-ufrag:MMIj
    a=ice-pwd:DH/uiOzo7wM8bcW2JazSs7hl
    a=ice-options:trickle
    a=fingerprint:sha-256 28:95:9F:3C:11:1C:CE:BE:C5:9F:A3:36:E3:13:17:2E:E1:55:A5:87:BC:99:28:17:37:E0:24:C2:5F:80:D5:8D
    a=setup:actpass
    a=mid:0
    a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
    a=extmap:9 urn:ietf:params:rtp-hdrext:sdes:mid
    a=sendrecv
    a=msid:prn0ZG6Pu8puzEVWKGK6q4refADHXBhjOPlS 7984ce7a-1e5e-4f88-b367-53cb98e7c1ed
    ...
    a=ssrc:4288075558 cname:BrCO4vpAD8Iny2BS
    a=ssrc:4288075558 msid:prn0ZG6Pu8puzEVWKGK6q4refADHXBhjOPlS 7984ce7a-1e5e-4f88-b367-53cb98e7c1ed
    a=ssrc:4288075558 mslabel:prn0ZG6Pu8puzEVWKGK6q4refADHXBhjOPlS
    a=ssrc:4288075558 label:7984ce7a-1e5e-4f88-b367-53cb98e7c1ed
   
    m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126
    c=IN IP4 0.0.0.0
    a=rtcp:9 IN IP4 0.0.0.0
    a=ice-ufrag:MMIj
    a=ice-pwd:DH/uiOzo7wM8bcW2JazSs7hl
    a=ice-options:trickle
    a=fingerprint:sha-256 28:95:9F:3C:11:1C:CE:BE:C5:9F:A3:36:E3:13:17:2E:E1:55:A5:87:BC:99:28:17:37:E0:24:C2:5F:80:D5:8D
    a=setup:actpass
    ...
    a=mid:1
    a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
    a=extmap:9 urn:ietf:params:rtp-hdrext:sdes:mid
    a=sendrecv
    a=msid:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9 b15fc3d6-17a3-4056-85ab-56405a188d3e
    ...
    a=ssrc:4161345580 cname:BrCO4vpAD8Iny2BS
    a=ssrc:4161345580 msid:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9 b15fc3d6-17a3-4056-85ab-56405a188d3e
    a=ssrc:4161345580 mslabel:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9
    a=ssrc:4161345580 label:b15fc3d6-17a3-4056-85ab-56405a188d3e
    
    m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 102 122 127 121 125 107 108 109 124 120 123 119 114
    c=IN IP4 0.0.0.0
    a=rtcp:9 IN IP4 0.0.0.0
    a=ice-ufrag:MMIj
    a=ice-pwd:DH/uiOzo7wM8bcW2JazSs7hl
    a=ice-options:trickle
    a=fingerprint:sha-256 28:95:9F:3C:11:1C:CE:BE:C5:9F:A3:36:E3:13:17:2E:E1:55:A5:87:BC:99:28:17:37:E0:24:C2:5F:80:D5:8D
    a=setup:actpass
    a=mid:2
    ...
    a=sendrecv
    a=msid:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9 6c641805-210c-4bde-b634-67e1615c74ca
    ...
    a=ssrc-group:FID 1447610669 3533280561
    a=ssrc:1447610669 cname:BrCO4vpAD8Iny2BS
    a=ssrc:1447610669 msid:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9 6c641805-210c-4bde-b634-67e1615c74ca
    a=ssrc:1447610669 mslabel:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9
    a=ssrc:1447610669 label:6c641805-210c-4bde-b634-67e1615c74ca
    a=ssrc:3533280561 cname:BrCO4vpAD8Iny2BS
    a=ssrc:3533280561 msid:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9 6c641805-210c-4bde-b634-67e1615c74ca
    a=ssrc:3533280561 mslabel:dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9
    a=ssrc:3533280561 label:6c641805-210c-4bde-b634-67e1615c74ca

注意事项：

- Plan B里面使用了"a=mid:audio"，并且两个tracks在同一个m=audio通过srrc指明。
- Unified Plan，"a=mid:0"用于第一个音轨 ，"a=mid:1" 用于第二个音轨。**mid是身份的标识，必不可少。**
- Chrome 的Unified Plan使用时，"a=ssrc"伴随了msid、mslabel和lable等属性，这几个属性只是为了向后兼容，而不是unified-plan必须的。

    
    
### RTCRtpTransceiver API和行为的变化
    
- SDP中，一个Transceiver代表一个m行，因为m行既可以用于sending，也可以用于receiving,而transceiver由发送器和接收器组成。
  收发的方向可以在“inactive”, “sendonly”, “recvonly” 和 “sendrecv”，之间根据需求切换。


- 将 track 使用addTrack() 或addTransceiver() 添加到peer connection时，transceiver会被创建（或重复使用），并将track附加到sender.。
因为transceiver可能是用于接收，receiving track始终被创建并连接到receiver.。不像发送方，receiver’s track始终存在且无法替换，但默认为muted状态。

- setLocalDescription()和setRemoteDescription()永久地将transceivers连接到m行，通过将transceiver.mid的值设置为m = section的mid。
这是一个字符串标识符，两端可以用来关联transceivers，以便知道哪个是哪个。


### Local 和 Remote 的 Track IDs 不匹配

**对于 Plan B：**

- 为每一个被添加的local track 创建了sender，为每一个协商的remote track 都创建了receiver。本地端和远程端的track id相互匹配。
    例如前面发送两个audio track的例子，通过webrtc-internal查看各个接口的状态，可以发现，sender和receiver是单独创建的。
offer端：
    
    	
    senderAdded
    Caused by: addTrack
    getSenders()[0]:{
      track:'default',
      streams:['uQXvAPEH23ysg7WvwxKVmyzdZchzQpwGYTd9'],
    }

    senderAdded
    Caused by: addTrack
    getSenders()[1]:{
      track:'default',
      streams:['HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm'],
    }

    senderAdded
    Caused by: addTrack
    getSenders()[2]:{
      track:'72f4efdadc3bc5a4c5f2ab4baa1f0a56fdf70b46a4b4ea83924afe95e8c1c4b4',
      streams:['HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm'],
    }

answer端：

	
    receiverAdded
    Caused by: setRemoteDescription
    getReceivers()[0]:{
      track:'f595523e-2fcd-400f-992e-0a127753952f',
      streams:['uQXvAPEH23ysg7WvwxKVmyzdZchzQpwGYTd9'],
    }

    receiverAdded
    Caused by: setRemoteDescription
    getReceivers()[1]:{
      track:'bbf350aa-714c-40f2-a646-6d235d154086',
      streams:['HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm'],
    }
	
    receiverAdded
    Caused by: setRemoteDescription
    getReceivers()[2]:{
      track:'27f35f0c-e1fa-4173-b9e1-ca1030987362',
      streams:['HNWGnpqStY44w8vHQZ3BE1VsRXmuajjk1pXm'],
    }

sender和receiver都只触发了一次。

**对于 Unified Plan：**

- senders 和 receivers 是成对产生的。`transceiver.receiver.track`可能在remote SDP提供track之前就创建了。所以，在RTCPeerConnection.ontrack触发的时候，并不能保证已经有一个和发送端所匹配的ID。
 例如前面发送两个audio track的例子，通过webrtc-internal查看各个接口的状态，可以发现，在ontrack触发的时候，mid为null，此时并不存在。senders 和 receivers 是成对产生的。
    
    
    transceiverAdded
    Caused by: addTrack
    getTransceivers()[0]:{
      mid:null,
      sender:{
        track:'default',
        streams:['prn0ZG6Pu8puzEVWKGK6q4refADHXBhjOPlS'],
      },
      receiver:{
        track:'0d78d6e0-4e4e-4038-9ba6-79104c31fd4b',
        streams:[],
      },
      stopped:false,
      direction:'sendrecv',
      currentDirection:null,
    }

    transceiverAdded
    Caused by: addTrack
    getTransceivers()[1]:{
      mid:null,
      sender:{
        track:'default',
        streams:['dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9'],
      },
      receiver:{
        track:'adeca0f1-498c-45f3-9a89-0e44c1b73b35',
        streams:[],
      },
      stopped:false,
      direction:'sendrecv',
      currentDirection:null,
    }

    transceiverAdded
    Caused by: addTrack
    getTransceivers()[2]:{
      mid:null,
      sender:{
        track:'72f4efdadc3bc5a4c5f2ab4baa1f0a56fdf70b46a4b4ea83924afe95e8c1c4b4',
        streams:['dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9'],
      },
      receiver:{
        track:'b9ddfabf-d391-465a-af59-2d0a142c5493',
        streams:[],
      },
      stopped:false,
      direction:'sendrecv',
      currentDirection:null,
    }
    
    
- 此外，因为addTransceiver() 和 replaceTrack()，同一个track可能被发送多次，Track IDs具有误导性。取而代之的，transceiver.mid可以用来关联local 和 remote tracks。
    你可能不得不在知道mid之前设置setLocalDescription(answer)。
    
transceiver会多次被触发，例如setLocalDescription，setRemoteDescription，addTrack，removeTrack：
        	
    transceiverModified
    Caused by: setLocalDescription
    getTransceivers()[0]:{
      mid:'0',
      sender:{
        track:'default',
        streams:['prn0ZG6Pu8puzEVWKGK6q4refADHXBhjOPlS'],
      },
      receiver:{
        track:'0d78d6e0-4e4e-4038-9ba6-79104c31fd4b',
        streams:[],
      },
      stopped:false,
      direction:'sendrecv',
      currentDirection:null,

    transceiverModified
    Caused by: setLocalDescription
    getTransceivers()[1]:{
      mid:'1',
      sender:{
        track:'default',
        streams:['dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9'],
      },
      receiver:{
        track:'adeca0f1-498c-45f3-9a89-0e44c1b73b35',
        streams:[],
      },
      stopped:false,
      direction:'sendrecv',
      currentDirection:null,
    }

    transceiverModified
    Caused by: setLocalDescription
    getTransceivers()[2]:{
      mid:'2',
      sender:{
        track:'72f4efdadc3bc5a4c5f2ab4baa1f0a56fdf70b46a4b4ea83924afe95e8c1c4b4',
        streams:['dqiheO0fWCOZSzt9gr183bdOzxrQZml8vHL9'],
      },
      receiver:{
        track:'b9ddfabf-d391-465a-af59-2d0a142c5493',
        streams:[],
      },
      stopped:false,
      direction:'sendrecv',
      currentDirection:null,
    }
    
    
### Remote Tracks Are Never Truly “Removed”

Plan B 里面：

- RTCPeerConnection.removeTrack()删除了sender and track，在协商的时候，对端也会删除对应的receiver and track。

Unified Plan 里面：

- removeTrack()改变了transceiver的收发方向，取消sender’s track。协商期间，transceiver’s改变收发方向，同时mute the track，但是从不 removed。并且这个被删除得track还可能再次复用。

### MediaStreamTrack.onended No Longer Fires

因为remote tracks可能被muted，但是永远不会被删除，MediaStreamTrack.onended 不会再被触发。相应的，使用 MediaStreamTrack.onmute来检查track是否被删除。


### Legacy APIs

RTCPeerConnection.addStream() 和 removeStream() 不是标准的 APIs。由于向后兼容的原因，它们在RTCPeerConnection.addTrack()和removeTrack()之上填充。
在Unified Plan中会继续使用，但是使用标准的APIs会更好。

同样地，在RTCPeerConnection上触发的事件"addstream"和"removestream"(使用事件处理程序"onaddstream"和"onremovestream")是标准中没有的遗留api。
由于向后兼容的原因，在M71的Unified Plan中支持它们，但是，RTCPeerConnection.ontrack, MediaStreamTrack.onmute 和 MediaStream.onaddtrack/onremovetrack更好。







