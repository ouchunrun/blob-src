---
title: 没有stream的情况下获取带有mid的sdp
date: 2019-3-30
tags: [webrtc] 
---

没有流的情况下获取带有mid的sdp
<!--more-->

```
var offerOptions = {
            offerToReceiveAudio: true,
        offerToReceiveVideo: true
    };
var pc = new RTCPeerConnection({"sdpSemantics": "unified-plan"});
pc.createOffer(function (offer) {
    var lines = offer.sdp.split("\n");
    for (var i = 0; i < lines.length; i++) {
        if(lines[i].match("mid:")){
            var mid = lines[i].split(":")[1];
            if(!isNaN(mid) && RTCRtpTransceiver){
                localStorage.setItem("RTCUnifiedPlan_enabled", 'true');
            }else {
                localStorage.setItem("RTCUnifiedPlan_enabled", 'false');
            }
        }
    }
    pc.close();
    pc = null;
}, function (error) {},offerOptions);
```


