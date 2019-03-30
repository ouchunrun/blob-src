---
title: Nginx中一个server配置多个location
date: 2019-3-30
tags: [nginx] 
---

## 启动

启动nginx，到nginx文件目录下，执行：
```
start nginx.exe
```

<!--more-->

停止nginx：

```
nginx -s stop
```

注意：有时候停止nginx会出现如下错误：
```
error: nginx: [emerg] invalid number of arguments in "root" directive in D:\nginx-1.15.0/conf/nginx.conf:83
```
这很可能是因为你在句末少些了分号，遇到这样的错误，可以先检查分号！

## 配置

80 端口配置：
```
    server {
        listen  80;
        server_name  demo1.webrtc.com;
        location / {
             root D:\WebRTC_Code;   // 本地仓库目录
             index  index.html;
        }
    }
```

443 端口配置：
```
server {
        listen       443 ssl;
        server_name  demo.webrtctest.com;

        ssl_certificate      D://nginx-1.15.0//ssl//_wildcard.ipvideotalk.com+2.pem;   # ssl 自签证书
        ssl_certificate_key  D://nginx-1.15.0//ssl//_wildcard.ipvideotalk.com+2-key.pem;

        location / {
            root D:\WebRTC_Code;   // 本地仓库目录
            add_header Access-Control-Allow-Origin *
            index  index.html index.htm;
        }
    }
```

nginx 中配置多个location，解决一个项目加载多个仓库代码问题：

```
server {
        listen       443 ssl;    # 端口
        server_name  style.webrtc.com;     # 域名

        # 配置ssl自签证书——这里路径必须是正的双斜杠
        ssl_certificate      D://nginx-1.15.0//ssl//_wildcard.ipvideotalk.com+2.pem;
        ssl_certificate_key  D://nginx-1.15.0//ssl//_wildcard.ipvideotalk.com+2-key.pem;
    
        location /web {
            root D:\projects\webrtc_demos\code
            add_header Access-Control-Allow-Origin *;
            index  index.html index.htm;
        }

        # 配置 /web/js/webrtc 访问路径
        # 加载https://style.webrtc.com/web/js/webrtc/目录时，加载https://demo1.webrtc.com/dist/output/ 下的文件
        location ^~ /web/js/webrtc/ {
            add_header Access-Control-Allow-Origin *;
            alias D:\projects\webrtc_demo_another\code\dist\output/;
        }

        # 配置 /web/js/webrtc/src 访问路径
        # 加载https://style.webrtc.com/web/js/webrtc/src/目录时，加载https://demo1.webrtc.com/src/ 下的文件
        location ^~ /web/js/webrtc/src/ {
            add_header Access-Control-Allow-Origin *;
            alias D:\projects\webrtc_demo_another\src/;
        }
    }
    
 server {
        listen  443 ssl;
        server_name  demo1.webrtc.com;
        location / {
             root D:\projects\webrtc_demo_another;   // 本地仓库目录
             index  index.html;
        }
    }
```
以上的配置就实现了一个项目加载多个仓库的项目代码问题，并且配置了两个域名，加载出来的代码也在各自的域名下。


注：alias指定的目录是准确的，root是指定目录的上级目录









