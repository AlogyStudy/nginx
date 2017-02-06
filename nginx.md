# nginx 简介

高性能WEB服务器

Nginx ('engine x') 是一个高性能的HTTP和反向代理服务器，也是一个  IMAP/POP3/SMTP 代理服务器.
Nginx 是由Igor Sysoev为额落实访问量第二的`Rambler.ru` 站点开发的.


## 编译安装：
```
root@# yum install pcre #先安装pcre,正则表达式的库
root@# ./configure --prefix=/usr/local/nginx
root@# make && make install
root@# cp /usr/local/nginx/conf/nginx.conf.default /usr/local/conf/nginx.conf
```

查看`nginx`文件底下目录:
```
conf // 配置文件目录
html // 放置网页文件
logs // 日志目录
sbin // 主要二进制程序文件
```

> 启动nginx

```
> ./sbin/nginx
```

报错：
```
[root@localhost nginx]# ./sbin/nginx 
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] still could not bind()
```
不能绑定80端口，80端口已经被占用.
解决，把占用80端口的软件或服务关闭即可.

```
> kill -9 标识
> pkill -9 服务
> pkill -9 http
```

## 编译PHP
```
./configure --prefix=/usr/local/php --enable-fpm
cp /source-path/php.ini-development /usr/local/php/lib/php.ini
```

# 信号量

nginx有两个进程，主进程`msater`进程。主进程不直接响应浏览器，是管理子进程使用。
浏览器访问会到子进程中响应。

```
> kill -INT pid（进程号）
```
-----
```
> ps aux | grep nginx  // 查看进程号
```

| 参数    |    意义    |
| --------    |  :----  |
| TERM,INT    | Quick shutdown （赶紧销毁进程）    |
| QUIT    | Graceful shutdown （优雅的关闭进程，即等待请求结束后再关闭）    |
| HUP    | Configuration reload ,Start the new worker processes with a new configuration Gracefully shutdown the old worker processes 改变配置文件,平滑的重读配置文件 | 
| USR1    | Reopen the log files 重读日志，在日志按月/日分割时有用     |
| USR2    | Upgrade Executable on the fly 平滑的升级 (升级nginx)    | 
| WINCH    | Gracefully shutdown the worker processes 优雅关闭旧的进程(配合USR2来进行升级)    | 

```
> kill -信号选项 nginx的主进程号
> kill -HUP 1480  （修改配置文件并不需要重启服务器，才能生效）
```

在linux中，一个文件对应一个节点`inode`,`inode`是文件真正在磁盘上的位置。文件名字是表现。
日志需要备份，文件在被nginx进程所打开，不能使用`mv`命令。 
除了使用`mv`命令，还需要建立新的文件，同时需要告知nginx服务器，来读取最新的日志文件。

```
> kill -信号控制 `cat /xxx/path/log/nginx.pid`
> kill -USR1 1480
> kill -USR1 `cat /xxx/path/log/nginx.pid`
> kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
```
-----
```
> ./sbin/nginx  -h

> ./sbin/nginx -s reload // 关闭
> ./sbin/nginx // 开启
> ./sbin/nginx -s reopen // 重读日志，相当于 // kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
> ./sbin/nginx -t // 检查日志文件是否准确
```

# 虚拟主机配置

> nginx配置段

**全局区**
worker_processes 1; // 有一个工作区域的子进程,可以自行修改，但太大无意义，因为要争夺CPU，一般设置为`CPU数*核数`

```
#user  nobody;
worker_processes  1; # 工作进程,子进程
```

**Evnet区**

一般配置nginx连接的特性
例如，1个worker能同时允许多少链接 

   
> Http段

```
http { // 这是配置http服务器的主要段
	Server1 { // 这是虚拟主机段
		Location { // 定位，把特殊的路径或文件再次定位，如image目录单独处理// 如 .php 单独处理

		}
	}
	Server2 {

	}
}
```


nginx配置文件：
```
#user  nobody;
worker_processes  1; # 工作进程,子进程

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

# 配置nginx连接的特性
events {
# 1个worker能同时允许多少链接 
    worker_connections  1024;  # 这是指 一个子进程最大允许1024个链接
}


# http 段是配置http服务器的主要段
http {
    include       mime.types;
    default_type  application/octet-stream;


    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    server { # 这是虚拟主机
        listen       80;
        server_name  localhost;

        #access_log  logs/host.access.log  main;

        location / { #
            root   html;
            index  index.html index.htm;
        #error_page  404              /404.html;

        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        #
        #}

        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    include        fastcgi_params;
        #}

        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443;
    #    server_name  localhost;

    #    ssl                  on;
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_timeout  5m;

    #    ssl_protocols  SSLv2 SSLv3 TLSv1;
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers   on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

简单的虚拟主机配置：

```
server {
	listen 80;
	server_name z.com;
	location / {
		root z.com; # root 路径可以是相对路径 相对nginx安装的根目录
        # 可以是绝对路径   /usr/loca/nginx;
		index index.html index.htm;
	}
}
server {
	listen 2022;
	server_name z.com;
	location / {
		root /home/mm/twinkle/; 
		index index.html index.htm;
	}
}
```

```
> ifconfig // 查看信息
```

# 日志管理

nginx的server段，可以看到的信息有：
`#access_log  logs/host.access.log  main;`该虚拟主机，它的访问日志的文件是`logs/host.access.log` 使用的格式是`main`格式.
除了`main`格式,还可以自定义其它格式.

main格式是什么?
```
#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
#                  '$status $body_bytes_sent "$http_referer" '
#                  '"$http_user_agent" "$http_x_forwarded_for"';
```
`main`格式是定义好的一种日志的格式，并起个名字，便于引用.
例如：
`main`类型的日志，记录了`remove_addr...http_x_forwarded_for`等选项.

```
remote_addr // 远程地址
remote_user[$time_local] // 访问远程的时间， 如果访问http头信息没有携带，是空的。
$request  // 请求方法`get` & `post`
$status // 请求状态码 例如：400
$body_bytes_sent // 主体发送的字节
$http_referer // referer头
$http_user_agent // 用户代理/蜘蛛
$http_x_forwarded_for // 被转发的请求原始IP (在经过代理时，代理把本来IP加在此头信息中，传输原始IP. )
```

蜘蛛：搜索引擎的服务器.
[百度robots](https://www.baidu.com/robots.txt)
蜘蛛协议.


Nginx允许针对不同的server做不同的log




