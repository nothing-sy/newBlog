---
title: Nginx配置localhost到github page
date: 2021-7-27
categories:
 - Nginx
---

> 旨在了解nginx基本配置



1. 下载和安装nginx

>官方下载好以后启动nginx，以下为基础的指令
>
>start nginx.exe #启动nginx
>
>nginx -s xxxx(以下指令)
>
>- `stop`  - 快速关机 
>
>- `quit`  - 较慢地关机 完成当前执行中的任务，再结束
>
>- `reload`  - 重新加载配置文件 
>
>- `reopen`  - 重新打开日志文件 





2. 配置本地localhost代理到github page 完整路径：https://heartofblack.github.io/Blog/

```nginx
#配置
server {
        listen       80;
        server_name  localhost;
			
        location /Blog/ { #location 后面为匹配规则,
			proxy_pass https://heartofblack.github.io; 
        #要代理到的服务器,后面不以/结尾，则会把匹配的字段加到后面实际访问 https://heartofblack.github.io/Blog/，
        #如果最后加了斜杠，则不会加上匹配字段
        }
}
```



3. 配置本地静态文件路径

```nginx
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root D:/test;  #可以直接写盘符位置，则代表访问localhost 路径指向 D:/test目录下的 index.html文件， 如果不带盘符，则默认为nginx安装的根目录，比如 root html 则为naginx安装根目录下的html文件夹
			index index.html;
        }
        }
```

