---
title: WebRTC Simulcast
date: 2018-10-26
tags: [WebRTC, SDP] 
---

### WebRTC Simulcast on Chrome：

chrome使用原有的sdp，在setRemoteDescription的之前添加多个srrc达到发送多个流的simulcast功能。


Using Native WebRTC simulcast support in Chrome：

- Step one: Add simulcast ssrcs and "SIM" group to the offer SDP.
- Step two?: Add a renegotiation.
- Step three: Add the x-conference flag to the video track in the answer SDP.(a=x-google-flag:conference)
- Step four: Decide what substreams you are interested in
- Step five: Increase the bandwidth


<!--more-->



### WebRTC Simulcast on Firefox：

firefox 则是通过 `RTCRtpSender.setParameters`设置参数来 enable simulcast。
并且在SDP上加上多个m行来发送多个流达到simulcast的功能。 



### Reference website

- Real Time Communications Bits:http://www.rtcbits.com/2014/09/using-native-webrtc-simulcast-support.html

- A playground for Simulcast without an SFU:https://webrtchacks.com/a-playground-for-simulcast-without-an-sfu/
 
- webrtc-simulcast.html: https://cs.chromium.org/chromium/src/chrome/test/data/webrtc/webrtc-simulcast.html?q=webrtc-simulcast.html&sq=package:chromium&dr

- draft-ietf-mmusic-sdp-simulcast-13: https://datatracker.ietf.org/doc/html/draft-ietf-mmusic-sdp-simulcast-13#page-10

- simulcast-playground Demo: https://github.com/fippo/simulcast-playground
