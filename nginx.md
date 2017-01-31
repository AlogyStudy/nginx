
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


## 编译PHP
```
./configure --prefix=/usr/local/php --enable-fpm
cp /source-path/php.ini-development /usr/local/php/lib/php.ini
```

