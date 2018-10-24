---
title: Edge 视频分辨率控制研究
date: 2018-10-23
tags: [WebRTC, 浏览器，Edge] 
---

#### **研究的主要目的**
>本次研究的主要目的是测试能否在**Edge**上控制分辨率，研究内容主要为以下两个方面：

- 1、GUM取流时控制帧率、分辨率。
- 2、GUM发流时通过`profile-level-id`控制帧率、分辨率。


#### **验证的设备：**

> - USB2.0 Camera 大河爱沙
> - Aoni(奥尼) HD Camera (1bcf:2283) 和 Gsou Audio Webcam (1871:0149) 极速 （这两款设备的设备支持能力相同）
> - Logitech Webcam C930e (046d:0843)
> 注：因为edge暂时没有提供获取原生设备能力的接口，所以目前使用Chrome的`chrome://media-internals`页面工具列出原生设备的支持能力作为测试参考数据。


<!--more-->

#### **验证结果：**

> **GUM取流时控制帧率、分辨率验证结果：**
>
> - （1）Edge中GUM尝试获取原生设备支持的能力时，不是所有的都能获取到，**也就是说只能获取一个子集**；要获取非原生设备支持的能力时（包括帧率、分辨率），都会 error
> - （2）Edge 在GUM获取失败时，不会触发OverconstrainedError提醒。
>
> **GUM发流时控制帧率、分辨率验证结果：**
>
> - （1）改变 `m`行中编解码器的先后顺序，可以改变编解码器的优先级。
> - （2）无论是改变local 还是remote 的`profile-level-id`值，没有任何效果，不会改变分辨率。



验证过程见后文！

---

**以下验证均使用 exact方式控制，例如：**
```
var constraints = {
    audio: true,
    video: {
        frameRate: { exact: 30 },
        width: {
            exact: 640,
        },
        height: {
            exact: 480,
        },
    }
}
```

#### **一、GUM取流时控制帧率、分辨率验证验证过程**

>  **（1）Edge | USB2.0 Camera (1871:0101)大河爱沙**

 使用Chrome的chrome://media-internals页面工具可以列出**大河爱莎**原生设备的支持能力。通过Constraints设置resolution && FrameRate验证结果如图所示。

![edge 大河爱沙 ](../../images/edge.jpg)

**注**：图中所列出的`framePerSecond`的值均为 **近似值** 而非精确值。

<center>图1、**大河爱莎** 分辨率、帧率控制验证结果</center>

**结果分析**：GUM尝试获取原生设备支持的能力时，`resolution=160x120、resolution=176x144`是获取不到的；GUM尝试获取非原生设备支持的能力时，均`fialed`。

---

> **（2）Edge | Aoni(奥尼) HD Camera** 

通过Constraints设置resolution && FrameRate验证结果如图:

![aoni奥尼验证](../../images/edge-aoni.jpg)

**注**：图中所列出的`framePerSecond`的值均为 **近似值** 而非精确值。

<center>图2、**aoni奥尼** 分辨率、帧率控制验证结果</center>

**结果分析**：

- GUM尝试获取原生设备支持的能力时，`frameRate=7.5`是获取不到的，其他都可以正常获取；GUM尝试获取非原生设备支持的能力时，均`fialed`。
- Edge可以控制帧率，但只有帧率控制在10以下（如上图所示），实际发出去的帧率才会下降，在15以上的帧率，发出去都在**15左右**。

---

> **（3）Edge 在GUM获取失败时，不会触发OverconstrainedError**

**验证设备：Logitech Webcam C930e (046d:0843)** 支持能力见图9。
 
当设置的Constraints不在原生设备支持能力之内时，并没有触发OverconstrainedError提示窗，而是继续取得一个stream，并建立p2p连接。

**结果分析**：获取不到指定的contraints时，edge不报错，其他两个camera验证结果相同，其他分辨率、帧率验证结果相同。

---

### **二、GUM发流时控制帧率、分辨率**


**测试设备：USB2.0 Camera 大河爱沙**

> **（1）改变 m行中编解码器的先后顺序，可以改变编解码器的优先级。**


local m=video行如下：

```
m=video 9 UDP/TLS/RTP/SAVPF 122 107 100 99 96 12
```

未改变的local video编解码器优先级顺序及实际使用情况如图3：

![title](../../images/desc-origin.jpg)
<center>图3、local video编解码器优先级顺序及实际使用情况</center>

通过replace改变local video m行编码器优先级：

```
desc.sdp = desc.sdp.replace("m=video 9 UDP/TLS/RTP/SAVPF 122 107 100 99 96 12", "m=video 9 UDP/TLS/RTP/SAVPF 107 122 100 99 96 12");
```


改变后的video编解码器优先级顺序及实际使用情况如图4：
![title](../../images/desc-change.jpg)
<center>图4、更改后local video编解码器优先级顺序及实际使用情况</center>
 

结论：改变 `m`行中编解码器的先后顺序，可以改变编解码器的优先级。

---

> **（2）无论是改变local 还是remote 的profile-level-id值，设置的参数都不生效。**

constraints 约束条件示例（使用**大河爱沙**原生设备所支持的最大值）：
```
    var constraints = {
        audio: true,
        video: {
            frameRate: { exact: 30 },
            width: {
                exact: 640,
            },
            height: {
                exact: 480,
            },
        }
    }
```

profile-level-id值设置方法（local和remote设置方法相同）：
```
desc.sdp = desc.sdp.replace("a=fmtp:107 profile-level-id=42C02A;packetization-mode=1;level-asymmetry-allowed=1", "a=fmtp:107 profile-level-id=42C03E;packetization-mode=1;level-asymmetry-allowed=1");

```

通过replace方法改变local 或remoted的profile-level-id的level-idc为下表中各值，结果如图5所示：

![level_id更改](../../images/profile-level-id.jpg)

**注**：图中所列出的`framePerSecond`的值均为 **近似值** 而非精确值。

<center>图5、level_id更改测试</center>

**结果分析**：profile-level-id设置无效

> `注`：表中标红字段，在level_idc为该值时，刚建立p2p连接的前两秒，分辨率为640x480px，之后才变为320x240px，有待分析。


---

### **三、原生设备支持能力**

[这里查看 --> chrome原生摄像头能力记录](https://192.168.120.100:9001/blog/post/chrou/chrome%E5%8E%9F%E7%94%9F%E6%91%84%E5%83%8F%E5%A4%B4%E5%88%86%E8%83%BD%E5%8A%9B)

---

**关于其他**

下面这两种方式都可以查看摄像头的分辨率支持能力，但是两种结果并不一样。

> - [WebRTC摄像头分辨率查询器](https://webrtchacks.github.io/WebRTC-Camera-Resolution/)
> - [chrome://media-internals](chrome://media-internals/)






