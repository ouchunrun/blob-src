---
title:  Web Worker 基础教程
date: 2019-6-27
tags: [JS, Worker] 
---



## 概述

Web Worker 的作用，就是为 JavaScript 创造多线程环境，允许主线程创建 Worker 线程，将一些任务分配给后者运行。在主线程运行的同时，Worker 线程在后台运行，两者互不干扰。等到 Worker 线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被 Worker 线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

Worker 线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了 Worker 比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。



Web Worker 有以下几个使用注意点。

（1）**同源限制**

分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

（2）**DOM 限制**

Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用`document`、`window`、`parent`这些对象。但是，Worker 线程可以`navigator`对象和`location`对象。

注：实践中有遇到过一个问题，postMessage过去的值是null，之前没有注意过DOM

（3）**通信联系**

Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。

（4）**脚本限制**

Worker 线程不能执行`alert()`方法和`confirm()`方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

（5）**文件限制**

Worker 线程无法读取本地文件，即不能打开本机的文件系统（`file://`），它所加载的脚本，必须来自网络。



## 一、实践

基本用法网上很多文章讲的很多很清楚，这里我想说一些别的。为了真正的理解worker线程不阻塞，你可以先运行下面的例子做个对比：

JS单线程执行示例：

```javascript
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>web worker 测试</title>
</head>
<body>

<div>
    <button id="begin" onclick="count();">Begin</button>
    <button id="color-button" style="color:blue" onclick="colorChange();">Now you can change it!</button>
</div>

<script>
    function colorChange(){
        let colorButton = document.getElementById('color-button')

        console.log('Changing color!')
        if(colorButton.style.cssText === 'color: red;'){
            colorButton.style.cssText = 'color: blue;'
        }else{
            colorButton.style.cssText = 'color: red;'
        }
    }
</script>

<script>
    function count(){
        let counter = 1000000001

        while( counter--){
            if( counter % 100000000 === 0) console.log(`While loop is Currently at: ${counter}`)
        }

        console.log('------------ While Loop has Ended! -----------')
    }
</script>
</body>
</html>
```

S单线程执行结果：

![webworker.jpg](https://i.loli.net/2019/06/26/5d130a53e1dfb67225.jpg)

点击 Begin 开始loop循环打印，打印过程中不管你什么时候点击 “Now you can change it!”，也不管你点多少次，“Changing color!”都会在loop循环之后执行。



web worker 代码执行示例：

```javascript
// 第一步：index.html
<div>
    <button id="begin">Begin</button>
    <button id="color-button" style="color:blue">Now you can change it!</button>
</div>


// 第二步：buttons.js
let colorButton = document.getElementById('color-button')

// Hacked refresh
document.getElementById('begin').addEventListener('click', e => location.reload())

colorButton.addEventListener('click', () => {
    console.log('Changing color!')
    if(colorButton.style.cssText === 'color: red;'){
        colorButton.style.cssText = 'color: blue;'
    }else{
        colorButton.style.cssText = 'color: red;'
    }
})

// 第三步：app.js
var worker = new Worker('js/worker.js')

// 第四步：worker.js 
let counter = 1000000001

while( counter--){
    if( counter % 100000000 === 0) console.log(`While loop is Currently at: ${counter}`)
}

console.log('------------ While Loop has Ended! -----------')

```

执行结果：

打开index.html，点击两个按钮，可以看到，loop执行的过程中，color事件也能够触发，像这样：

![webworker.jpg](https://i.loli.net/2019/06/26/5d130f104fd3f31629.jpg)

动手实际操作下，能够很快明白！！！是不是很nice~~



## 二、基本用法

了解了weorker的作用了，接下来看以下webworker的基本用法。

### 2.1 新建线程

主线程采用`new`命令，调用`Worker()`构造函数，新建一个 Worker 线程。

```javascript
var worker = new Worker('work.js');
```

参数是一个脚本文件，该文件就是 Worker 线程所要执行的任务。由于 Worker 不能读取本地文件，所以这个脚本必须来自网络。

### 2.2 消息监听

对于每一个子线程，`self`代表子线程自身，即子线程的全局对象。

```javascript
self.addEventListener('message', function (e) {
    this.postMessage('You said: ' + e.data);
}, false);
```

以上对消息监听的代码实现，等同于以下两种写法：

```javascript
// 写法一
this.addEventListener('message', function (e) {
  this.postMessage('You said: ' + e.data);
}, false);

// 写法二
addEventListener('message', function (e) {
  postMessage('You said: ' + e.data);
}, false);
```

实际开发中我更倾向于下面的写法：

```javascript
var worker = self;

worker.onmessage = function (e) {
    console.log("收到message");
    this.postMessage('You said: ' + e.data);
}

```



> 在主线程中使用时，`onmessage`和`postMessage()` 必须挂在worker对象上，而在worker中使用时不用这样做。原因是，在worker内部，worker是有效的全局作用域。

主线程和子线程通过Worker.postMessage()和Worker.onmessage()方法进行消息的传递和监听。



### 2.3  Worker 区分多个子线程

多个子线程可通过不同的线程名进行区分，主线程也需要对每个子线程进行单独的onmessage监听，示例如下：

```javascript
<!--主线程环境下-->
// 子线程一
var worker1 = new Worker('js/worker1.js');
// 子线程二
var worker2 = new Worker('js/worker2.js');

worker1.onmessage = function (event) {
    console.warn('接收到worker1发送过来的消息 ', event.data);
    doSomething();
};

worker2.onmessage = function (event) {
    console.warn('接收worker2发送过来的消息 ', event.data);
};
```

 

### 2.4 Worker 关闭进程

如：关闭worker1线程

```javascript
// 方法一：在主线程中关闭
worker1.terminate();
// 方法二：子线程自己关闭
self.close()
```



### 2.5 Worker 子线程加载脚本

Worker 内部如果要加载其他脚本，有一个专门的方法`importScripts()`。

```javascript
importScripts('script1.js');
```



该方法可以同时加载多个脚本。

```javascript
importScripts('script1.js', 'script2.js');
```



## 三、数据通信

主线程与 Worker 之间的通信内容，可以是文本，也可以是对象。需要注意的是，这种通信是拷贝关系，即是传值而不是传址，Worker 对通信内容的修改，不会影响到主线程。事实上，浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给 Worker，后者再将它还原。

主线程与 Worker 之间也可以交换二进制数据，比如 File、Blob、ArrayBuffer 等类型，也可以在线程之间发送。下面是一个例子。

```javascript
// 主线程
var uInt8Array = new Uint8Array(new ArrayBuffer(10));
for (var i = 0; i < uInt8Array.length; ++i) {
  uInt8Array[i] = i * 2; // [0, 2, 4, 6, 8,...]
}
worker1.postMessage(uInt8Array);

// Worker1 线程
self.onmessage = function (e) {
  var uInt8Array = e.data;
  postMessage('Inside worker.js: uInt8Array.toString() = ' + uInt8Array.toString());
  postMessage('Inside worker.js: uInt8Array.byteLength = ' + uInt8Array.byteLength);
};
```



但是，拷贝方式发送二进制数据，会造成性能问题。比如，主线程向 Worker 发送一个 500MB 文件，默认情况下浏览器会生成一个原文件的拷贝。为了解决这个问题，JavaScript 允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做[Transferable Objects](http://www.w3.org/html/wg/drafts/html/master/infrastructure.html#transferable-objects)。这使得主线程可以快速把数据交给 Worker，对于影像处理、声音处理、3D 运算等就非常方便了，不会产生性能负担。

如果要直接转移数据的控制权，就要使用下面的写法。

```javascript
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);

// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```



## 四、 主线程的script脚本创建worker

```javascript
<!DOCTYPE html>
  <body>
    <script id="worker" type="app/worker">
       this.addEventListener('message', function (e) {
           console.log("子线程收到主线程发送的消息：", e.data);
            postMessage('你好，我是新的线程');
        }, false);
    </script>
  </body>
</html>
```

上面是一段嵌入网页的脚本，注意必须指定`<script>`标签的`type`属性是一个浏览器不认识的值，上例是`app/worker`。

然后，读取这一段嵌入页面的脚本，用 Worker 来处理。

```javascript
var blob = new Blob([document.querySelector('#worker').textContent]);
var url = window.URL.createObjectURL(blob);
console.warn("创建的blob地址：", url);
var worker3 = new Worker(url);

worker3.postMessage("worker3 ？ ，你好啊");

worker3.onmessage = function (e) {
    console.warn("收到worker 3 的消息：", e.data);
};

```



## 五、实例： Worker 新建 Worker

下面的例子是将worker1 中一个计算密集的任务，分配到10个 Worker。Worker 线程代码如下（worker.js）：	

```javascript
// settings
var num_workers = 10;
var items_per_worker = 1000000;

// start the workers
var result = 0;
var pending_workers = num_workers;
for (var i = 0; i < num_workers; i += 1) {
    var worker = new Worker('worker3.js');
    worker.postMessage(i * items_per_worker);
    worker.postMessage((i + 1) * items_per_worker);
    worker.onmessage = storeResult;
}

// handle the results
function storeResult(event) {
    console.warn("worker 1 收到他的子进程: ", event.data);
    result += event.data;
    pending_workers -= 1;
    if (pending_workers <= 0){
        postMessage(result); // finished!
    }
}
```

上面代码中，Worker 线程内部新建了10个 Worker 线程，并且依次向这10个 Worker 发送消息，告知了计算的起点和终点。计算任务脚本的代码如下。

worker.js  的子线程 core.js 文件计算任务脚本的代码如下。

```javascript
var start;
onmessage = getStart;
function getStart(event) {
    start = event.data;
    console.log("start: ", start);
    onmessage = getEnd;
}

var end;
function getEnd(event) {
    end = event.data;
    console.log("end: ", end);
    onmessage = null;
    work();
}

function work() {
    var result = 0;
    for (var i = start; i < end; i += 1) {
        // perform some complex calculation here
        result += 1;
    }
    postMessage(result);
    close();
}
```



## 六、API

### 6.1 主线程

浏览器原生提供`Worker()`构造函数，用来供主线程生成 Worker 线程。

```javascript
var myWorker = new Worker(jsUrl, options);
```

`Worker()`构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称（默认是空），用来区分多个 Worker 线程。

```javascript
// 主线程
var myWorker = new Worker('worker.js', { name : 'myWorker' });

// Worker 线程
self.name // myWorker
```



Worker()`构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下。

| 方法/属性             | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| Worker.onerror        | 指定 error 事件的监听函数                                    |
| Worker.onmessage      | 指定 message 事件的监听函数，发送过来的数据在`Event.data`属性。 |
| Worker.onmessageerror | 指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。 |
| Worker.postMessage()  | 向 Worker 线程发送消息                                       |
| Worker.terminate()    | 该方法用于立即终止 [`Worker`](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker) 的行为. 本方法并不会等待 worker 去完成它剩余的操作；worker 将会被立刻停止。 |



### 6.2 Worker 线程

Web Worker 有自己的全局对象，不是主线程的`window`，而是一个专门为 Worker 定制的全局对象。因此定义在`window`上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

| 属性/方法            | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| self.name            | Worker 的名字。该属性只读，由构造函数指定。                  |
| self.onmessage       | 指定`message`事件的监听函数。                                |
| self.onmessageerror  | 指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。 |
| self.close()         | 关闭 Worker 线程。                                           |
| self.postMessage()   | 向产生这个 Worker 线程发送消息。                             |
| self.importScripts() | 加载 JS 脚本。                                               |



## FAQ

Q1：子线程间是否可以进行通信？如何通信？

Q2：主线程是否可进行广播？

Q3：子线程可以再创建线程，那如果是创建一个主线程已经创建的线程，这两个线程属于同一个吗？不是的话，是否可以进行函数的相互调用？



## 参考

<http://www.ruanyifeng.com/blog/2018/07/web-worker.html>

<https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers>

（完）

