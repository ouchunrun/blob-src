---
title: 监听localStorage变化设置页面跳转
date: 2018-3-18
tags: [JS, localStorage] 
---

“当同源页面的某个页面修改了localStorage,其余的同源页面只要注册了storage事件，就会触发” 

同页面监听，重写localStorage的方法，抛出自定义事件，根据localStorage的值判断，进行不同的操作：

```
 var orignalSetItem = localStorage.setItem;
    localStorage.setItem = function(key,newValue){
        var setItemEvent = new Event("setItemEvent");
        setItemEvent.key = key;
        setItemEvent.newValue = newValue;
        window.dispatchEvent(setItemEvent);
        orignalSetItem.apply(this,arguments);
    };
    window.addEventListener("setItemEvent", function (e) {
        if(e.key === 'b_release_mode'){
            if(e.newValue === "true"){
                console.warn("Currently using compressed code !");
                window.location.href = 'http://localhost:8080/';
            }else if(e.newValue === "false"){
                console.warn("Switch to uncompressed code !");
                window.location.href = 'http://webpack.sourcemap.com/';
            }
        }
    });
```


参考：https://blog.csdn.net/hl_qianduan/article/details/83273602