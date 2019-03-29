---
title: JS文件加载过程中产生的依赖问题定位过程
date: 2019-3-20
tags: [JS, 浏览器, 文件加载依赖] 
---
## 写在前面的话

最近在做底层代码和业务层代码分离工作的过程中，遇到脚本加载过程中产生各种“变量未定义”的问题，之前并没有细致的了解动态脚本加载的本质，也没太关注脚本onload事件，所以今天针对动态脚本写个笔记，毕竟"好记性不如烂笔头"。
<!--more-->


## JS脚本加载

### 几种常用的JavaScript脚本加载方式

- 内部引用JavaScript

```html
<head>
    <script type="text/javascript">
        document.write("Hello World !");
    </script>
</head>
```

- 外部引用JavaScript

```html
<script type="text/javascript" src="index.js"></script>
```
使用script标签的src属性来加载js脚本。通常JavaScript文件可以使用script标签加载到网页的任何一个地方，但是标准的方式是加载在head标签内。为防止网页加载缓慢，也可以把非关键的JavaScript放到网页底部。　


- 内联引用JavaScript：通过HTML标签中的事件属性实现的。
```html
<input type="button" value="点我" onclick="alert('你点击了一个按钮');">
```

### 脚本加载所在位置：顶部？底部？

1、最差情况：将脚本放在顶部

脚本对Web页面的影响如下：

- 脚本会阻塞对其后面内容的呈现。

- 脚本会阻塞对其后面组件的下载。

脚本放在顶部会导致整个页面的呈现被阻塞，因而会产生白屏现象。逐步呈现对于良好的用户体验非常重要，但缓慢的脚本下载延迟了用户所期待的的反馈。

2、最佳情况：将脚本放在底部

放置脚本最好的地方就是页面的底部（虽然请求时间长但对页面的影响很小）。这不会阻止页面内容的呈现，而且页面中的可视组件可以尽快下载。

3、正确的放置：建议使用延迟脚本。

defer属性表明脚本不包含document.write，浏览器得到这一线索就可以继续进行呈现。如果一个脚本可以延迟，那么他一定可以转移到页面的底部。这是加速Web页面的最佳方式。


### 动态加载脚本

#### 异步批量添加外部脚本

很多时候我们由于产品模块的划分，一个页面可能需要加载几个脚本，我们需要考虑两点：

- 1、脚本之间是否有依赖关系，如果存在依赖关系即使我们使用script标签是按照顺序的，但是并行下载是一起下载的，如果出现后面的包先下载完，那么执行脚本时就可能出现错误；

- 2、考虑到效率，一般情况下异步加载比同步加载会快一些。

#### 同步分类（或模块）动态加载

这里的动态加载是指当用户使用到了某个类或者模块才去加载，并且加载不是用户来控制，而是自动的。这里没有具体研究。

#### 异步分类（或模块）动态加载
```javascript
function add_js_scripts(s_elt) {
    var tag_hdr = document.getElementsByTagName(s_elt)[0];
    for (var i = 1; i < arguments.length; ++i) {
        var tag_script = document.createElement('script');
        tag_script.setAttribute('type', 'text/javascript');
        tag_script.setAttribute('src',Style_Domain + "/web/js/webrtc/"+ arguments[i] + "?svn=224");
        tag_hdr.appendChild(tag_script);
    }
};
```


动态脚本加载技术是非阻塞的javascript下载中最常用的模式，可以跨浏览器。这种方式在元素被添加到页面之后立刻开始下载，这样无论在何处启动下载，文件的下载和运行都不会阻塞其它的页面处理过程，甚至可以将代码放在head部分而不会对其余部分的脚本产生影响，下载文件的HTTP链接的情况除外。

注意：无阻塞加载脚本的核心技术就是动态的创建script的dom节点。

无阻塞脚本加载技术还有个好处就是，那些和页面展示无关的脚本无须非要放在onload事件里执行，它随时随地可以运行简直就是完美。不过无阻塞脚本有个很大的隐患，这个隐患是很多会使用无阻塞脚本技术的程序员都会忽视的问题，这个问题就是无阻塞脚本很容易产生“变量未定义”的问题，这个问题的本质就是无阻塞脚本会破坏js脚本加载顺序的问题，当某个脚本依赖于另一个脚本时候，而另一个脚本又没有加载执行完毕，最后就会产生“变量未定义”的问题，例如jQuery没有提前加载，因此使用$时候提示$变量未定义。解决这个问题的思路就是让那些依赖于无阻塞加载的脚本的js代码在脚本加载完毕后才会执行，也就是添加一个方法将无序的脚本加载变得有序。

```javascript
// 解决javascript文件加载过程中产生的依赖问题
function loadRTCScripts(scriptLists, selectElem){
    if (!scriptLists || scriptLists.length <= 0){
        return;
    }
    if (scriptLists .length > 0){
        var url = scriptLists[0];
        scriptLists.splice(0,1);
        var script = document.createElement("script");
        script.type = "text/javascript";
        script.src = Style_Domain+"/web/js/webrtc/" + url + "?svn=224";
        document.getElementsByTagName(selectElem)[0].appendChild(script);

        if (script.readyState){ //IE 6~8
            script.onreadystatechange = function(){
                if (script.readyState === "loaded" || script.readyState === "complete"){
                    script.onreadystatechange = null;
                    console.log(url + " js file load ok!");
                    loadRTCScripts(scriptLists, selectElem);
                }
            };
        } else { //  IE9+, chrome, firefox support
            script.onload = function(){
                console.log(url + " js file load ok!");
                loadRTCScripts(scriptLists, selectElem);
            };
        }
    }
}
```
该方法通过对dom节点onload的监听，在前一个文件准备完成后在加载一下一个，这样就不会存在变量未定义的问题。

在非ie浏览器下有一个onload事件，该事件会在script加载完毕后才会执行，ie浏览器下有onreadystatechange事件，而ie下script的dom节点有一个readystate属性，它的取值如下：

- 1.uninitialized（未初始化）：对象存在尚未初始化；
- 2.loading（正在加载）：对象正在加载数据；
- 3.loaded（加载完毕）：对象数据加载完成
- 4.interactive（交互）：可以操作对象，但是还没有完全加载；
- 5.complete（完成）：对象已经加载完毕。


---

### 参考资料

《高性能网站建设》

[【前端词典】F5 同 Ctrl+F5 的区别你可了解](https://newsn.net/say/webstorm-shortcut.html)

[javascript文件加载过程中产生的依赖问题](https://blog.csdn.net/liangklfang/article/details/50185475)

[无阻塞加载js，防止因js加载不了影响页面显示](https://www.cnblogs.com/woodk/p/4732268.html)

[探真无阻塞加载javascript脚本技术，我们会发现很多意想不到的秘密](http://www.cnblogs.com/sharpxiajun/p/4072396.html)

[高性能Javascript：脚本的无阻塞加载策略](http://developer.51cto.com/art/201410/453722.htm)

[js文件加载优化](https://segmentfault.com/a/1190000004448625)



















    





