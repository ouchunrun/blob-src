---
title: enumerateDevices 的 DeviceId生存周期
date: 2018-10-25
tags: [WebRTC, deviceId] 
---

 # **描述**

方法` MediaDevices.enumerateDevices() ` 用于收集系统上可用的多媒体输入和输出设备的信息。返回一个Promise


 # **示例**

列出摄像头和麦克风
<!--more-->

```
    navigator.mediaDevices.enumerateDevices()
    .then(function(devices) {
      devices.forEach(function(device) {
        console.log(device.kind + ": " + device.label +
                    " id = " + device.deviceId + "\n groupId =  " + device.groupId);
      });
    })
    .catch(function(err) {
      console.log(err.name + ": " + err.message);
    });
```


直接在控制台输入如上代码，就可以测试

---

### 浏览器deviceId生存周期

# ![Chrome](https://cdn1.iconfinder.com/data/icons/logotypes/32/chrome-32.png) Chrome （ 69.0.3497.92）

（1）chrome 同一个Tab页，刷新页面、访问别的页面再退回当前页面，deviceId 和 groupId 相同

（2）chrome 不同的Tab页下，deviceId 和 groupId 不同


# ![Firefox](https://cdn1.iconfinder.com/data/icons/logotypes/32/firefox-32.png) Firefox（62.0）

（1）Firefox 同一个Tab页，刷新页面、访问别的页面再退回当前页面，deviceId 不变

（2）Firefox 不同的Tab页下，deviceId 不同

（3）groupId 为空，应该是不支持groupId



# ![Edge](https://cdn4.iconfinder.com/data/icons/picons-social/57/56-edge-2-32.png) Edge（17.17134）

（1）每次刷新界面，deviceId 和 groupId 就会改变

（2）Edge 不同的Tab页下，deviceId 和 groupId 不同


# Oprea(55.0.2994.61 )

（1）Oprea 同一个Tab页，刷新页面、访问别的页面再退回当前页面，deviceId 不变

（2）Oprea 不同的Tab页下，deviceId 不同

（3）groupId 为空，应该是不支持groupId


#![Edge](https://cdn1.iconfinder.com/data/icons/logotypes/32/internet-explorer-32.png) IE（11.0.85）


（1）navigator.mediaDevices.enumerateDevices()

（2）`无法获取未定义或 null 引用的属性“enumerateDevices”`

（3）IE 目前不支持enumerateDevices



---

