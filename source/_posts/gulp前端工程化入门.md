---
title: gulp前端工程化入门
date: 2019-3-28
tags: [gulp, 打包工具] 
---

## 安装

初始化package.json文件
```
npm init
```

作为项目的开发依赖（devDependencies）安装：
```
npm install --save-dev gulp
```

<!--more-->

全局安装 gulp：：
```
npm install --g gulp
```

## 配置 和 常用插件

### package.json配置：
```
{
  "name": "ipvt_webrtc_client_base",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "gulp dev",
    "dev": "gulp dev",
    "clean": "rimraf dist",
    "build": "rimraf dist && gulp build"
  },
  "dependencies": {
    "express": "3.x"
  },
  "devDependencies": {
    "concat": "^1.0.3",
    "gulp": "^3.9.1",
    "gulp-clean": "^0.3.2",
    "gulp-concat": "^2.6.1",
    "gulp-foreach": "^0.1.0",
    "gulp-javascript-obfuscator": "^1.1.5",
    "gulp-mini-css": "^0.0.3",
    "gulp-order": "^1.2.0",
    "gulp-requirejs-optimize": "^0.3.2",
    "gulp-sourcemaps": "^2.6.5",
    "gulp-uglify": "^1.5.4",
    "gulp-util": "^3.0.8",
    "stream-combiner2": "^1.1.1"
  }
}

```

### gulpfile.js 配置

```

var gulp = require('gulp');
var concat = require('gulp-concat');   // js 合并
var uglify = require('gulp-uglify');   //js压缩 
var order = require("gulp-order");   
var foreach = require('gulp-foreach');
var gutil = require('gulp-util');
var combiner = require('stream-combiner2');
var sourcemaps = require('gulp-sourcemaps');
var javascriptObfuscator = require('gulp-javascript-obfuscator');   //js压缩混淆的gulp插件

var src_js = './src/';
var dest_js = './dest/';

var handleError = function (err) {
    var colors = gutil.colors;
    console.log('\n');
    gutil.log(colors.red('Error!'));
    gutil.log('fileName: ' + colors.red(err.fileName));
    gutil.log('lineNumber: ' + colors.red(err.lineNumber));
    gutil.log('message: ' + err.message);
    gutil.log('plugin: ' + colors.yellow(err.plugin))
}

gulp.task('minjs', function () {
    var combined = combiner.obj([
        gulp.src([ 'src/main.js', 'src/index.js'])
            .pipe(foreach((stream, file) => {
                console.log(file.path);
                return stream;
            }))
            .pipe(concat('output.min.js')),
        sourcemaps.init(),
        uglify({
            mangle: true,
            compress: true,
            preserveComments: false,
            toplevel: true
        }),
        javascriptObfuscator({
            compact:true,
            sourceMap: true
        }),
        sourcemaps.write('./'),
        gulp.dest(dest_js)
    ])
    combined.on('error', handleError)
})

gulp.task('default',function(){
    gulp.run('minjs');
});

gulp.task('default', ['minjs']);


```

### 文件操作插件

**1、gulp-rename**

描述：重命名文件。

```
var rename = require("gulp-rename");

gulp.src('./hello.txt')
  .pipe(rename('gb/goodbye.md'))    // 直接修改文件名和路径
  .pipe(gulp.dest('./dist')); 
 
gulp.src('./hello.txt')
  .pipe(rename({
    dirname: "text",                // 路径名
    basename: "goodbye",            // 主文件名
    prefix: "pre-",                 // 前缀
    suffix: "-min",                 // 后缀
    extname: ".html"                // 扩展名
  }))
  .pipe(gulp.dest('./dist'));
```

**2、gulp-concat**

描述：合并文件。
```
var concat = require('gulp-concat');

gulp.src('./js/*.js')
    .pipe(concat('all.js'))         // 合并all.js文件
    .pipe(gulp.dest('./dist'));
    
gulp.src(['./js/demo1.js','./js/demo2.js','./js/demo2.js'])
    .pipe(concat('all.js'))         // 按照[]里的顺序合并文件
    .pipe(gulp.dest('./dist'));
```


**3、 gulp-filter**

描述：在虚拟文件流中过滤文件。
```
var filter = require('gulp-filter');

const f = filter(['**', '!*/index.js']);
gulp.src('js/**/*.js')
    .pipe(f)                        // 过滤掉index.js这个文件
    .pipe(gulp.dest('dist'));

const f1 = filter(['**', '!*/index.js'], {restore: true});
gulp.src('js/**/*.js')
    .pipe(f1)                       // 过滤掉index.js这个文件
    .pipe(uglify())                 // 对其他文件进行压缩
    .pipe(f1.restore)               // 返回到未过滤执行的所有文件
    .pipe(gulp.dest('dist'));       // 再对所有文件操作，包括index.js
```

**4、del (替代gulp-clean)**
```
var del = require('del');

del('./dist');                      // 删除整个dist文件夹
```

**5、gulp-clean**
描述：清除文件

```
var clean = require('gulp-clean');  // 清除文件
// 清除dest目录下所有文件
gulp.task("clean", function(){
    return gulp.src('dest/')
        .pipe(clean());
})
```


### 压缩

**1、 gulp-uglify**
描述：压缩js文件大小。

```
var uglify = require("gulp-uglify");

gulp.src('./hello.js')
    .pipe(uglify())                 // 直接压缩hello.js
    .pipe(gulp.dest('./dist'))
    
 gulp.src('./hello.js')
    .pipe(uglify({
        mangle: true,               // 是否修改变量名，默认为 true
        compress: true,             // 是否完全压缩，默认为 true
        preserveComments: 'all'     // 保留所有注释
    }))
    .pipe(gulp.dest('./dist'))
```


## 其他

1、任务按顺序执行

默认的，task 将以最大的并发数执行，也就是说，gulp 会一次性运行所有的 task 并且不做任何等待。如果你想要创建一个序列化的 task 队列，并以特定的顺序执行，你需要做两件事：

- 给出一个提示，来告知 task 什么时候执行完毕，
- 并且再给出一个提示，来告知一个 task 依赖另一个 task 的完成。

对于这个例子，让我们先假定你有两个 task，"one" 和 "two"，并且你希望它们按照这个顺序执行：

- 在 "one" 中，你加入一个提示，来告知什么时候它会完成：可以再完成时候返回一个 callback，或者返回一个 promise 或 stream，这样系统会去等待它完成。

- 在 "two" 中，你需要添加一个提示来告诉系统它需要依赖第一个 task 完成。

因此，这个例子的实际代码将会是这样：

```
var gulp = require('gulp');

// 返回一个 callback，因此系统可以知道它什么时候完成
gulp.task('one', function(cb) {
    // 做一些事 -- 异步的或者其他的
    cb(err); // 如果 err 不是 null 或 undefined，则会停止执行，且注意，这样代表执行失败了
});

// 定义一个所依赖的 task 必须在这个 task 执行之前完成
gulp.task('two', ['one'], function() {
    // 'one' 完成后
});

gulp.task('default', ['one', 'two']);
```

2、文件按顺序压缩

```
var foreach = require('gulp-foreach');

gulp.task('minjs', function () {
    var combined = combiner.obj([
        gulp.src([ 'src/main.js', 'src/index.js'])
            .pipe(foreach((stream, file) => {
                console.log(file.path);
                return stream;
            }))
        gulp.dest(dest_js)
    ])
    combined.on('error', handleError)
})
```


3、sourceMap文件生成

```
var sourcemaps = require('gulp-sourcemaps');

gulp.task('minjs', function () {
    var combined = combiner.obj([
        gulp.src([ 'src/main.js', 'src/index.js'])
        sourcemaps.init(),
        sourcemaps.write('./'),
        gulp.dest(dest_js)
    ])
    combined.on('error', handleError)
})
```

4、压缩混淆
```
var uglify = require('gulp-uglify');

gulp.task('minjs', function () {
    var combined = combiner.obj([
        gulp.src([ 'src/main.js', 'src/index.js'])
        uglify({
            mangle: true,
            compress: true,
            preserveComments: false,
            toplevel: true
        }),
        javascriptObfuscator({
            compact:true,
            sourceMap: true
        }),
        gulp.dest(dest_js)
    ])
    combined.on('error', handleError)
})

```

完整的配置示例参考gulpfile.js 文件！

## 添加版本号（?hash）

1、改变文件名称

- 这种方式必须同时改变资源的文件名和html里面引用的文件名，并且一一对应
- 可以用 *gulp-rev gulp-rev-collector* 两个插件实现安装：
```
npm install --save-dev gulp-rev gulp-rev-collector
```
- 效果如下：
```
{
  "output.min.js": "output-d6592a15e8.min.js"
}
```

修改*gulp-rev*的配置也能实现*main.min.js?61e0be79*的输出格式，但是比较麻烦，用`gulp-rev-dxb`插件就可以了！

配置的修改可以参考这里：[gulp版本号?v=](https://blog.csdn.net/m0_37285193/article/details/81566243)

2、在文件后缀名之后通过添加 ?hash 实现

示例：
```
<script src=”main.min.js?61e0be79”></script> 等价于 
```

安装插件：
```
npm install --save-dev gulp-rev-dxb gulp-rev-collector-dxb
```


参数说明:

- rev() ：给文件添加版本号
- rev.manifest() ：生成版本号清单文件

```
var rev = require('gulp-rev-dxb');	// 生成版本号清单

gulp.task('minjs', function () {
    var combined = combiner.obj([
        gulp.src([ 'src/main.js', 'src/index.js'])
            .pipe(foreach((stream, file) => {
                console.log(file.path);
                return stream;
            }))
            .pipe(concat('output.min.js'))
            .pipe(rev())   // 给文件添加版本号
            .pipe(gulp.dest(dest_js))
            .pipe(rev.manifest())    // 生成版本号清单文件
            .pipe(gulp.dest(dest_js +'/rev')),
        gulp.dest(dest_js)
    ])
    combined.on('error', handleError)
})
```

效果如下：
```
{
  "output.min.js": "output.min.js?v=91f3aa04c9"
}
```

> 这里有个问题，文件生成了,rev-manifest.json也生成了，，虽然rev-manifest.json里面生成了版本映射，但是文件并没有添加版本号信息！


3、gulp版本tag生成
```
var git = require('gulp-git');
var fs = require('fs');
gulp.task('create-new-tag', function (cb) {
    var version = getPackageJsonVersion();
    console.log(version);
    git.tag(version, 'Created Tag for version: ' + version, function (error) {
        if (error) {
            return cb(error);
        }
        git.push('origin', 'master', {args: '--tags'}, cb);
    });

    function getPackageJsonVersion () {
        // 这里我们直接解析 json 文件而不是使用 require，这是因为 require 会缓存多次调用，这会导致版本号不会被更新掉
        return JSON.parse(fs.readFileSync('./package.json', 'utf8')).version;
    };
});

```

执行`gulp createNewTag`命令能够把生成的tag上传到最近的一次提交，效果如下：

```
commit 2171b07ab5d91da35261a5a4919f7345475f7ec2 (HEAD -> master, tag: 1.0.0, origin/master, origin/HEAD)
```

这里的vertion值是通过package.json中的version获取的，所以每次打版本tag，只能先修改package.json的值。如果有动态生成tag的方法，更好！


## 参考

- [精通gulp常用插件](https://segmentfault.com/a/1190000008349859)
- [Gulp & webpack 配置详解](https://www.jianshu.com/p/2d9ed1fe3e8c)
- [gulp-javascript-obfuscator](https://github.com/javascript-obfuscator/gulp-javascript-obfuscator)
- [改变版本号以及创建一个 git tag](https://www.gulpjs.com.cn/docs/recipes/bump-version-and-create-git-tag/)





