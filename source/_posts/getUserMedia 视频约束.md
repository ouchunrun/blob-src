---
title: getUserMedia 视频约束
date: 2018-10-25
tags: [WebRTC, getUserMedia] 
---


### **MediaStreamConstraints**

MediaStreamConstraints对象用于指定要请求的轨道类型（音频，视频或二者）以及（可选）每个轨道的任何要求。

约束项可以具有下面这两个属性中的一个或两个：

- video—表示是否需要视频轨道

- audio—表示是否需要音频轨道

当这个约束项同时包括音频和视频流的时候，看起来就是这样的：

```
var Constraints = {
    audio: true,
    video: true,
}
```

<!--more-->

如果我们只是想获取视频，并不需要音轨的时候就可以将audio属性设置成false，反之亦然：

```
// 只获取视频
var Constraints = {
    audio: false,
    video: true,
}

// 只获取音频
var Constraints = {
    audio: true,
    video: false,
}
```

---

### **MediaTrackConstraints**

音频和视频属相可以是下面这两种类型的值：

- 像上面的例子一样是一个布尔值
- 一个MediaTrackConstraints对象，其提供了像宽度和高度这样必须与轨道匹配的特定属性。


**视频轨道约束：分辨率**

可以使用宽度和高度属性从网络摄像头请求一定的分辨率。

**关键字**

如果分辨率对于你来说很重要，而且设备和浏览器不能够保证分辨率的时候，你可以用min，max和exact关键字来帮助你从任何设备中得到最佳的分辨率。这些关键字可以应用到任何MediaTrackConstraint属性中。

但是，用exact的时候，，如果不存在支持精确分辨率的摄像头，则返回的promise将会被拒绝并给出OverconstrainedError错误，而且不会提醒用户。

那些没有min，max和exact这些关键字描述的值会被视为“ideal”（理想）值，它本身就是一个关键字，但不是强制性的。也就是说，下面两种写法是相同的：

```
// 写法一
var Constraints = {
    audio: true,
    video: {
        width: { ideal: 640};
        height: {ideal: 480},
    },
}

// 写法二
var Constraints = {
    audio: true,
    video: {
        width: 640;
        height: 480,
    },
}
```


**视频轨道约束：帧速率**

因为帧速率不仅对视频质量，还对带宽有着直接影响，所以在某些情况下，比如通过低带宽连接发布视频流的时候，限制帧速率可能是个好主意。

```
var Constraints = {
    audio: true,
    video: {
        width: { ideal: 640};
        height: {ideal: 480},
        framRate: { min: 5,ideal: 15, max: 30},
    },
}
```

**视频轨道约束：宽高比**

```
height divided by width – usually 4/3 (1.33333333333) or 16/9 (1.7777777778)
```

常见标准分辨率

|Width |Height|	Aspect Ratio|
|:---:|:---:|:---:|:---:|
|1280|	720|    16:9|
|960|	720|	4:3|
|640|	360|    16:9|
|640|	480|	4:3|
|320|	240|	4:3|
|320|	180|	16:9|


**使用特定的网络摄像头或者麦克风**

下面这个约束属性同时适用于音频和视频轨道：deviceId。它指定了被用于捕捉流的设备ID。**这个设备ID是唯一的，并且在同一个来源的会话中是相同的**。你需要首先使用MediaDevices.enumerateDevices()来获取设备id。

一旦你知道了deviceId之后，你就可以要求指定的摄像头和麦克风了：
```
var Constraints = {
    audio: true,
    video: {
        width: { ideal: 640};
        height: {ideal: 480},
        framRate: { min: 5,ideal: 15, max: 30},
        deviceId: {
            exact": "24ec72d98d664c710837afcee2343f50d9ab6e40b1474e9b28e5231efeb63cd0"
        }
    },
}
```

还有一个属性是groupId，这是一个实体设备中所有媒体源共享的ID。举个例子，在同一个耳机上的麦克风和扬声器就会共享同样的groupId。

**MediaDevices.getSupportedConstraints()**

这个函数会返回一个字典，列出用户代理支持的约束。部分浏览器支持。

**MediaStreamTrack.getSettings()**

你还可以通过track.getSettings检查有哪些约束是支持的。它会返回一个包括了所有可用约束的对象，包括了那些浏览器虽然支持的，但是默认值没有通过代码改变的。

```
navigator.mediaDevices.getUserMedia(constraints).then(function(stream){
    video.srcObject = stream;
    stream.getTracks().forEach(function(track){
        console.log(track.getSettings());
    })
}).catch(function(error){
    coonsole.log(error.mssage);
});
```

[getUserMedia()视频约束](http://webrtc.org.cn/getusermedia-video-constraints/)











