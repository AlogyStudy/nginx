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



