---
title: WebRTC演示分屏实现思路
date: 2018-10-25
tags: [WebRTC, SDP] 
---

 <h3 style="background:#cbc9c9;">思路</h3>
 
 对于分屏，实际就是在本地建立一个 P2P 连接，把演示流发送给新窗口，类似于WEBRTC demo 上面的 简单的对等连接的例子[点击查看示例 》 Basic peer connection demo](https://webrtc.github.io/samples/src/content/peerconnection/pc1/)。

---

 <h3 style="background:#cbc9c9;">目前支持情况</h3>
 

目前基本的主流浏览器都可以实现分屏，如：Firefox、chrome、vivaldi、Opera 。


<!--more-->

 
---

 <h3 style="background:#cbc9c9;">接下来考虑的问题</h3>
 
 4. 在分屏的时候，尽量使用与当前所建立的收演示流的编码一致，一般为VP8或H.264
 5. 在Chrome上需要添加x-google-start-bitrate 及 SDP中b行的配置
    - 因为在分屏建立对等连接的时候，本端不知道对端的能力，所以一开始会发低一点。可以在answer中告诉offer我的能力，一开始就发高清晰度的演示流。


---

 <h3 style="background:#cbc9c9;">参考点</h3>
 
**（1）关于x-google-start-bitrate的协商的参考SDP：**

```
v=0
o=root 367665105 367665106 IN IP4 192.168.121.152
s=GrandStream X-Server 0.0.5.3 (M)
c=IN IP4 192.168.121.152
t=0 0
m=video 63104 RTP/SAVPF 126 96
b=AS:1024
a=ice-ufrag:79871092667f8d7234cbbba237b2b537
a=ice-pwd:7de7d05f74d8b5aa4a7ac6ae000361e0
a=candidate:Hc0a87998 1 UDP 2130706431 192.168.121.152 63104 typ host
a=connection:existing
a=setup:passive
a=fingerprint:sha-256 67:04:9D:03:6D:64:FA:31:2C:83:B1:4F:90:8F:E4:33:5D:F1:A4:F8:6C:59:8D:38:B7:DA:94:7F:92:73:52:BE
a=ssrc:197102408 cname:983a8c0
a=ssrc:197102408 msid:0bwdE607of2lH5GWuLSmBG689JEZcyiQsERm 3933dd34-b9ea-47f9-bb50-8a63066d1040
a=rtcp-fb:* nack pli
a=rtpmap:126 H264/90000
a=fmtp:126 max-fs=8100;packetization-mode=1;level-asymmetry-allowed=1;profile-level-id=42E028;max-fs=8100;max-mbps=40500;x-google-min-bitrate=1024;`x-google-start-bitrate=1536;x-google-max-bitrate=2048`
a=rtpmap:96 VP8/90000
a=fmtp:96 max-fr=5
a=fmtp:96 max-fs=8100
a=rtcp-fb:* ccm fir
a=content:`slides`
a=rtcp-mux
a=recvonly
```
 
这个参数仅供Chrome解析用，建议仅对演示流进行设置。位置放于某种编码的fmtp a行下。大致说明如下：


|字段             |含义          |
|:--------------: |:----------------------: |
|x-google-min-bitrate    |字面理解是设置最小码率，由于Google自家的Hangout也没用这个参数，我们也不需要设置码率的下限|
|x-google-start-bitrate |起始码率，其作用等于是b行的作用，告知内部编码器可以使用的**起始带宽值**是多少，由此控制初始码率。默认值为300|
|x-google-max-bitrate|最大码率，其作用用于限制码率。




查阅Chrome代码，以下几个字段含义如下：
|字段            |含义            |
|:--:            |:--             |
|target_media_bitrate_bps|程序中自己计算出来的目标码率，受x-google-start-bitrate控制 |
|media_bitrate_bps| 编码器实际编出来的码率 |
|preferred_media_bitrate_bps| 来自SDP b行AS中的带宽限制，受x-google-max-bitrate控制 |


---

**（2）关于bandwidth参数协商的流程的介绍：**

 1. Bandwidth的定义：  
    SDP中的bandwidth参数用于在OfferSDP中告知对方本设备的解码器可以接受的最大会话流或媒体流的bit率
    
 2. Bandwidth的格式：

 在SDP中的m行之前（关于会话的）或m行之后（关于对应媒体流的）都可以加bandwidth参数，具体格式为：b=<bwtype>:<bandwidth>，其中不同的bwtype，对应不同的带宽限制的计算方法，

|bwtype|bandwidth|
|:--:|:--:|
|CT  |本地解码器接受的最大会议会话组IP层信息所占带宽的总和（包含多个会话的多个媒体流），单位是kbits/s|
|AS  |本地解码器接受的最大单路会话IP层信息所占带宽，单位是kbits/s|
|TIAS   |本地解码器接受的最大单路会话的媒体流应用层信息所占带宽，单位是bits/s


**关于视频AS值的计算方法：**

视频流的带宽TIAS值采用话机的VideoBitRate，

> 带宽AS值 = （带宽TIAS值/1000） + （RTPHeader+UDPHeader+IPHeader）的传输比特率

目前抓包统计的1080P的RTP发包率为531pkts/s，所以我们将预估了一个最大发包率为600pkts/s（600这个数字之前也和kpgao确认过）

> （RTPHeader+UDPHeader+IPHeader）的传输比特率 = 600 * （12+8+20）* 8 = 192000（bits/s）

所以

> 视频带宽AS值 = VideoBitRate + 192


**关于bandwidth的协商的参考SDP:**

```
v=0
o=Example_SERVER 3413526809 0 IN IP4 server.example.com
s=Example of TIAS and maxprate in use
c=IN IP4 0.0.0.0
`b=AS:60`
`b=TIAS:50780`
t=0 0
a=control:rtsp://server.example.com/media.3gp
a=range:npt=0-150.0
a=maxprate:28.0
m=audio 0 RTP/AVP 97
`b=AS:12`
`b=TIAS:8480`
a=maxprate:10.0
a=rtpmap:97 AMR/8000
a=fmtp:97 octet-align;
a=control:rtsp://server.example.com/media.3gp/trackID=1
m=video 0 RTP/AVP 99
```


---


 <h3 style="background:#cbc9c9;"> 写在最后</h3>

第一：在建立对等连接时 ，answer在SDP中携带自己的最高能力（甚至是比自己能力更高的的编解码能力），这样就可以避免一开始分屏的模糊状态。

第二：chrome中有一个配置项，可以设置降低分辨率以维持帧率（帧率优先），还是降低帧率以维持分辨率（分辨率优先）
[优先降分辨率还是优先降帧率](https://w3c.github.io/webrtc-pc/#dom-rtcdegradationpreference)

RTCDegradationPreference Enumeration description

|字段|含义|
|--|--|
|maintain-framerate	|Degrade resolution in order to maintain framerate.|
|maintain-resolution	|Degrade framerate in order to maintain resolution.|
|balanced	|Degrade a balance of framerate and resolution.|














