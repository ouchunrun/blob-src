---
title: JS文件加载过程中产生的依赖问题定位过程
date: 2018-3-14
tags: [JS, 浏览器, 文件加载依赖] 
---

最近在使用 [SIPML5](https://github.com/DoubangoTelecom/sipml5) 尝试做底层代码和业务层代码分离工作得时候，遇到一个问题：
整个项目代码使用继承封装实现，原有的做法是每个父类加载完，再加载其他的子类文件，现在考虑把所有要加载的文件放在一个地方统一处理，
但是就算父类先通过DOM SCRIPT形式先添加，也会有变量未定义的错！无论父类怎么提前，也没能解决这个问题。这里主要记录一下这个
问题处理的过程，供以后参考。

<!--more-->

先看一下动态动态脚本加载的代码：
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

第一种方法：在项目加载正常的情况下，参照文件的加载顺序，列出加载的文件列表，通过上面提到的函数进行加载，大致如下
(ps：文件较多，列了一部分)
```javascript
// add tinySIP API
add_js_scripts('head',
    'src/tinySAK/src/tsk_base64.js',
    'src/tinySAK/src/tsk_buff.js',
    'src/tinySAK/src/tsk_fsm.js',
    'src/tinySAK/src/tsk_md5.js',
    'src/tinySAK/src/tsk_param.js',
    'src/tinySAK/src/tsk_ragel.js',
    'src/tinySAK/src/tsk_string.js',
    'src/tinySAK/src/tsk_utils.js',
    'src/tinySAK/src/tsk_blob.js',
    'src/tinySAK/src/tsk_other.js',
    // ......
    )
```
结果不如人意，这种方式下页面只能偶尔正常加载，你可能会好奇，为什么是偶尔？那是因为在多次强制刷新或清除历史记录情况下，
页面偶尔正常加载。题外话，因为怀疑页面加载不成功可能是缓存的问题，所以看了一下F5同ctrl+F5的区别，有兴趣可能看一下
[【前端词典】F5 同 Ctrl+F5 的区别你可了解](https://newsn.net/say/webstorm-shortcut.html)，作者写的很明了。

了解了F5同ctrl+F5的区别后，刷新时也做了强制刷新的限制，仍然不能解决问题，第一种方法到这里就放弃了。

----------------------


接下来，在查找"JS加载依赖"的时候，看到[javascript文件加载过程中产生的依赖问题](https://blog.csdn.net/liangklfang/article/details/50185475)
这篇文章，这里面提到：
> 动态脚本加载技术是非阻塞的javascript下载中最常用的模式，可以跨浏览器。这种方式在元素被添加到页面之后立刻开始下载，这样无论在何处启动下载，
文件的下载和运行都不会阻塞其它的页面处理过程，甚至可以将代码放在head部分而不会对其余部分的脚本产生影响，下载文件的HTTP链接的情况除外。

元素一旦添加到页面后就立刻下载并运行，也就是说，如果运行之前，所依赖的文件没有加载完成的话，就会出现变量未定义的情况。
所以考虑所有加载的文件，都等前一个脚本加载完成了再加载，大概就是这样：
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

注意：无阻塞加载脚本的核心技术就是动态的创建script的dom节点。

>无阻塞脚本加载技术还有个好处就是，那些和页面展示无关的脚本无须非要放在onload事件里执行，它随时随地可以运行简直就是完美。
不过无阻塞脚本有个很大的隐患，这个隐患是很多会使用无阻塞脚本技术的程序员都会忽视的问题，这个问题就是无阻塞脚本很容易产生
“变量未定义”的问题，这个问题的本质就是无阻塞脚本会破坏js脚本加载顺序的问题，当某个脚本依赖于另一个脚本时候，
而另一个脚本又没有加载执行完毕，最后就会产生“变量未定义”的问题，例如jQuery没有提前加载，因此使用$时候提示$变量未定义。


到这里，问题就差不多解决了！

本文的主要目的就是记一次对文件加载依赖的处理过程，如有不妥的地方，希望各位多指点一二，不吝赐教。


---

参考资料

[javascript文件加载过程中产生的依赖问题](https://blog.csdn.net/liangklfang/article/details/50185475)

[无阻塞加载js，防止因js加载不了影响页面显示](https://www.cnblogs.com/woodk/p/4732268.html)

[探真无阻塞加载javascript脚本技术，我们会发现很多意想不到的秘密](http://www.cnblogs.com/sharpxiajun/p/4072396.html)

[高性能Javascript：脚本的无阻塞加载策略](http://developer.51cto.com/art/201410/453722.htm)

[js文件加载优化](https://segmentfault.com/a/1190000004448625)




















    





