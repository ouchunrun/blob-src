---
title: VS code 代码跳转配置
date: 2021-6-20
tags: [webpack]
---


## 一、webpack

Webpack是一款用户打包前端模块的工具，它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。主要是用来打包在浏览器端使用的javascript的。同时也能转换、捆绑、打包其他的静态资源，包括css、image、font file、template等

webpack的官网： [http://webpack.github.io/](http://webpack.github.io/)

<!--more-->

---
## 二、运行 webpack

直接运行一下命令，可显示错误信息：
```
$ webpack --display-error-details
```
或者配置其他参数：

```
$ webpack --config XXX.js   //使用另一份配置文件（比如webpack.config2.js）来打包

$ webpack --watch   //监听变动并自动打包

$ webpack -p    //压缩混淆脚本，这个非常非常重要！

$ webpack -d    //生成map映射文件，告知哪些模块被最终打包到哪里了
复制代码
```

非压缩打包：
```
webpack --mode=development
```


---

## 三、webpack打包多个js文件

webpack.config.js 文件配置

**1、多个js文件不合并打包(分别打包)**

```
const path = require("path");
module.exports = {
    mode: "development", //打包为开发模式
    // 入口配置的对象中，属性为输出的js文件名，属性值为入口文件
    entry: {
    	main1:"./src/main1",
    	main2:"./src/main2"
    }, //入口文件,从项目根目录指定
    output: { //输出路径和文件名，使用path模块resolve方法将输出路径解析为绝对路径
        path: path.resolve(__dirname, "../dist/js"), //将js文件打包到dist/js的目录
        filename: "[name].js" //使用[name]打包出来的js文件会分别按照入口文件配置的属性来命名
    }
}
```

**2、多个js中部分合并打包成一个js文件**
```
const path = require("path");
module.exports = {
    mode: "development", //打包为开发模式
    // 出口对象中，属性为输出的js文件名，属性值为入口文件
    entry: {
    	main1:"./src/main1",
    	main:["./src/main2","./src/main3"]
    }, //入口文件,从项目根目录指定
    output: { //输出路径和文件名，使用path模块resolve方法将输出路径解析为绝对路径
        path: path.resolve(__dirname, "../dist/js"), //将js文件打包到dist/js的目录
        filename: "[name].js" //使用[name]打包出来的js文件会分别按照入口文件配置的属性来命名
    }
}
```

**3、多个js全部打包成一个js文件**

```

const path = require("path");
module.exports = {
    mode: "development", //打包为开发模式
    // 出口对象中，属性为输出的js文件名，属性值为入口文件
    entry: ["./src/main1","./src/main2","./src/main3"], //入口文件,从项目根目录指定
    output: { //输出路径和文件名，使用path模块resolve方法将输出路径解析为绝对路径
        path: path.resolve(__dirname, "../dist/js"), //将js文件打包到dist/js的目录
        filename: "main.js" 
    }
```

原文地址：https://blog.csdn.net/weixin_36185028/article/details/81117730 《webpack4打包多个js文件》

**4、上述提到的部分打包方式，个人认为，比较合适于文件较少的情况，所以针对文件较多的情况，还需要其他的处理，用来遍历文件：**


**入口配置文件:**

匹配js路径下除exr_js文件外的其他的JS脚本
```
var JS_PATH_WEBRTC = {
    js: {
        pattern: ['./js/**/[^_]*.js', '!./js/ext_js/**/[^_]*.js'],
        dst: path.resolve(__dirname, 'dist/output'),
    }
}
```


**遍历除所有需要打包的JS文件路径:**
```

function getJSEntries(config) {
    var fileList = glob.sync(config.pattern)
    return fileList.reduce(function(previous, current) {
        previous.push(path.resolve(__dirname, current))
        return previous
    }, [])
}
```

**多个入口文件配置：**
```
module.exports = [
    {
        entry: ['./common/debug.js'],   
        output: {
            filename: 'debug.js',
            path: path.resolve(__dirname, 'dist/output'),
            publicPath: '/output'
        }
    },
    {
        entry: ['./common/adapter.js'],  
        output: {
            filename: 'adapter.js',
            path: path.resolve(__dirname, 'dist/output'),
            publicPath: '/output'
        }
    },
    {
        entry: webrtc_release_mode ? releaseJS : './js/unrelease_load.js',
        output: {
            filename: 'gsRTC.js',
            // path: JS_PATH.js.dst,
            path: path.resolve(__dirname, 'dist/output'),
            publicPath: '/output'
        },
    },
]

```
或
```
module.exports = {
    devtool: 'source-map',
    entry:{
        debug: getJSEntries(JS_PATH_DEBUG.js),
        adapter: getJSEntries(JS_PATH_ADAPTER.js),
        gsRTC: b_release_mode ? release_lists : './common/gsRTC_unrelease_load.js',
    },
    output: {
        filename:'[name].js',
        path: path.resolve(__dirname,'dist/output'),
        publicPath: '/output'
    },
    devServer: {
        contentBase: "./test",     //本地服务器所加载的页面所在的目录
        historyApiFallback: true,  //不跳转
        inline: true               //实时刷新
    }
}
```


参考文章：https://www.jianshu.com/p/439764e3eff2《【WEBPACK】分离css单独打包》


---

## 四、webpack生成Source Maps文件

在 webapck.config.js 中添加配置，生成生成Source Maps脚本：

```
module.exports = {
    devtool: 'source-map',   // 生成Source Maps
    entry:{
        debug: ['./common/debug.js'],   
        adapter: ['./common/adapter.js'], 
        gsRTC: webrtc_release_mode ? releaseJS : './js/unrelease_load.js'
    },
    output: {
        filename:'[name].js',
        path: path.resolve(__dirname,'dist/output'),
        publicPath: '/output'
    },
}
```

devtool有四种不同的配置：


|devtool选项                  | 配置结果                    |
|:---------------------------:|:---------------------------:|
|source-map      |在一个单独的文件中产生一个完整且功能完全的文件。这个文件**具有最好的source map**，但是它会减慢打包速度；|
|cheap-module-source-map   |在一个单独的文件中生成一个**不带列映射的map**，不带列映射提高了打包速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便；|
|eval-source-map      |使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。在开发阶段这是一个非常好的选项，在生产阶段则一定不要启用这个选项；|
|cheap-module-eval-source-map  |这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点；|


## 五、使用webpack构建本地服务器

安装
```
npm install --save-dev webpack-dev-server
```

添加配置：

```
module.exports = {
    devtool: 'source-map',   // 生成Source Maps
    entry:{
        debug: ['./common/debug.js'],   
        adapter: ['./common/adapter.js'], 
        gsRTC: webrtc_release_mode ? releaseJS : './js/unrelease_load.js'
    },
    output: {
        filename:'[name].js',
        path: path.resolve(__dirname,'dist/output'),
        publicPath: '/output'
    },
    devServer: {
        contentBase: "./public",//本地服务器所加载的页面所在的目录
        historyApiFallback: true,//不跳转
        inline: true//实时刷新
      } 
}
```

在package.json中的scripts对象中添加如下命令，用以开启本地服务器：

```
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack",
    "server": "webpack-dev-server --open"
  },
```

运行：
```
npm run server
```

这样应该可以看到结果了！

参考：https://www.jianshu.com/p/42e11515c10f《入门Webpack，看这篇就够了》


## [补充] UglifyJSPlugin

安装
```
npm install --save-dev uglifyjs-webpack-plugin
```

在package.json中的scripts对象中添加如下配置：
```
var UglifyJSPlugin = require('uglifyjs-webpack-plugin');


module.exports = {
  optimization: {
    minimizer: [
      new UglifyJSPlugin({
        sourceMap: false,
        uglifyOptions: {
          compress: false,  // 压缩
          minimize: false,
          mangle: false,   // 混淆
          beautify: true,   // 美化
        }
      }),
    ]
  },

};
```