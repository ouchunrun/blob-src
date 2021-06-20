---
title: Nginx 配置
date: 2021-6-20
tags: [Nginx]
---

### Nginx proxy_pass

1、示例：
```
server {
    listen  443 ssl;
    server_name  nginx.test.com;   # 本地配置域名
    location / {
        root D:\projects\shared\nginxTest;
        index  index.html;
    }
    location /data-get/ {   # 需要代理的访问路径
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Credentials true;
        add_header Access-Control-Allow-Headers 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Requested-With';
        add_header Access-Control-Allow-Methods 'GET,POST,OPTIONS';
        proxy_pass https://192.168.131.111/data-get/;
    }
}
```

<!--more-->

2、配置说明：https://nginx.test.com/data-get 路径下的所有请求，都转到https://192.168.131.111/data-get/服务器上

3、第二步proxy时存在跨域问题，需要服务器端添加允许跨域处。（以下字段非全部必须）
```
add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Credentials true;
add_header Access-Control-Allow-Headers 'Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Requested-With';
add_header Access-Control-Allow-Methods 'GET,POST,OPTIONS';
```

4、可能出现 `403 Forbidden` 禁止访问问题，还是需要服务器做处理！

### 不同的访问路径设置不同的文件访问路径

- 示例：

```
server {
    listen       443 ssl;
    server_name  abc.examlpes.com;

    ssl_certificate      D://nginx-1.15.0//ssl//_wildcard.ipvideotalk.com+2.pem;
    ssl_certificate_key  D://nginx-1.15.0//ssl//_wildcard.ipvideotalk.com+2-key.pem;


    location /web {
        root D:\projects\code;
        add_header Access-Control-Allow-Origin *;
        index  index.html index.htm;
    }
    # 不同的访问路径设置不同的文件访问路径
    location ^~ /web/js/webrtc/ {
        add_header Access-Control-Allow-Origin *;
        alias D:\projects\code2\dist\output/;
    }

    location ^~ /web/js/webrtc/src/ {
        add_header Access-Control-Allow-Origin *;
        alias D:\projects\code2\src/;
    }
}
```

- 配置的时候遇到一个问题，配置都正确的，但是页面加载始终不对，后来发现是nginx重启是没有完全kill掉，要把nginx进程kill到0，再去重启才可以。

**windows下的nginx后台进程真的坑！！**
**windows下的nginx后台进程真的坑！！**
**windows下的nginx后台进程真的坑！！**

> 注意：！！！记住，修改了nginx配置，重启nginx后没有任何反应，你应该先查看nginx是否真的重启了，这很重要，否则怎么改都是没用的！


### nginx win32配置遇到的问题和解决方法

- 场景：项目放在了F盘根目录下。nginx在c盘根目录下。
- 问题：nginx配置后页面无法访问
    - nginx 配置如下：
```
location /web {
    root F:\workspace\code;
    add_header Access-Control-Allow-Origin *;
    index  index.html index.htm;
}
```

- 原因：
    - NGINX win32 的ssl_certificate 配置路径是相对当前配置文件的路径，所以使用../的方式能够找到配置文件
    - NGINX win32 的location配置路径是针对nginx的当前所在路径的，比如我当前nginx所在路径为c盘根目录。访问文件时会自动添加nginx所在路径，所以实际访问的是'c:/nginx/code'。
    - NGINX  找不到文件时，会根据nginx当前路径加上root路径去查找，这也就是为什么error.log里面有那个前缀的原因
    - NGINX win32 无法跨盘访问文件
    - 备注：nginx win 64 目前已经能够访问绝对路径，不存在NGINX win32的这个问题。

> 备注：nginx访问的路径自动添加了'cygdirve/c/nginx/' 。

