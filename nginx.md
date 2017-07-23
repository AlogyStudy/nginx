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

Nginx允许针对不同的server做不同的Log(有的web服务器不支持，如：lighttp)

```
access_log logs/z.com.access.log main; // 配置单独的log日志
```

# 定时任务

> shell 脚本

shell 脚本注意空格

变量声明
```
a // 定义
$a // 使用
```

输出执行返回结果
```
echo `date -d yesterday %Y%m%d` // 第一种: ``
echo $(date -d yesterday %Y%m%d) // 第二种: $()
```

日志切割
```
#!/bin/bash
LOGPATH=/usr/local/nginx/logx/z.com.access.log
BASEPATH=/data

bak=$BASEPATH/$(date -d yesterday +%Y%m%d%H%M).zcom.access.log

#echo $bak

mv $LOGPATH $bak
touch $LOGPATH

kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
```

# Location

`location` 有定位的意思，根据Uri来进行不同定位

在虚拟主机的配置中，是必不可少的location可以把网站的不同部分，定位到不同的处理方式上。
比如。碰到.php，如何调用PHP解释器，这时需要location。

location 的语法
```
location [=|~|~*|^~] patt {
}
```
中括号可以不写任何参数，此时称之为一般匹配
也可以写参数
因此，大类型可以分为3种：
1. location = patt {} [精准匹配]
2. location patt {} [一般匹配]
3. location ~ patt {} [正则匹配]

## 精准匹配

如何发挥作用?

首先看是否有精准匹配，如果有，则停止匹配过程.
```
location = patt {
    config A
}
```
如果 $uri == patt, 匹配成功,使用 `config A`


判断使用的那个location
```
location = / {
    root /var/www/html/;
    index index.htm index.html;    
}
location / {
    root /usr/local/nginx/html;
    index index.html index.htm;
}
```

如果访问的是:`http://www.xxx.com`
定位流程：
1. 精准匹配中 `/`，得到`index` 页为`index.htm`
2. 再次访问 `/index.htm`， 此次内部跳转uri是`/index.htm`,跟目录为`/usr/local/nginx/html`
3. 最终结果，访问了 `/usr/local/nginx/html/index.htm`


精准匹配测试
```
location = /index.htm { // 添加匹配文件，而非目录.
}
location /index.htm {
}
```

`一般匹配`与`精准匹配`冲突时，`精准`发挥作用.

## 正则匹配
 

```
location / {
    root /usr/local/nginx/html;
    index index.html index.htm;
}

location ~ image {
    root /var/www/image;
    index index.html index.htm;
}
```

如果访问： `http://xxx.com/image/a.jpg`
这时，`/`与`/image/a.jpg` 匹配，同时`image`正则与`image/a.jpg`也能匹配，谁发挥作用?

`一般匹配`与`正则匹配`冲突，`正则`发挥作用。

图片真正访问的地址：`/var/www/image/a.jpg`


```
locatiorn / {
  root /usr/loca/nginx/html;
  index index.html index.htm;
}

location /foo {
  root /var/www/html;
  index inde.html
}
```

当访问`http://xxx.com/foo`

对于uri为`/foo`,两个location的patt，都能够匹配。
即`/`能够左前缀匹配，`/foo`也能左前缀匹配.
此时真正访问的是`/var/www/html/foo`.
原因:`/foo`匹配更长，因此使用.

## location整体流程

正则匹配有顺序关系，从上到下.有匹配上的就退出.
普通匹配不需要考虑顺序，挨个匹配，最长的就是匹配上.

![](./_image/2017-02-08-23-35-47.jpg)


location 的匹配过程：
1. 先判断精准匹配，如果匹配成功，立即返回结果并结束解析过程.
2. 判断一般命中，如果有多个命中，`记录`下来`最长`的匹配结果. (注意：记录但不结束，最长为准).
3. 继续正则表达式的解析结果，按配置里的正则表达式顺序为准，由上到下开始匹配，一旦匹配成功，理解返回结果，并结束解析过程.


延伸分析：
* 普通匹配顺序无所谓，是按照匹配长短来确定的.
* 正则匹配是按照顺序来匹配.


# rewrite

rewrite(重写)语法详解

重写的规则可以放在`location`或者`service`中.

常用的命令
1. if 空格 (条件) {} # 设定条件，再进行重写
2. set # 设置变量
3. return # 返回状态码
4. break # 跳出rewrite
5. rewrite # 重写


**if 语法**

```
if 空格 (条件) { // 空格不允许少
    重写默认
}
```

条件写法：
1. `=`来判断相等，用于字符串比较
2. `~`用正则来匹配(区分大小写) , `~*`用来正则匹配(不区分大小写)
3. -f-d-e 来判断是否为文件，为目录，是否存在.

**相等**
```
location / {
  if ($remote_addr = 192.168.1.12) {
      return 403;
  }
  root html;
  index index.html index.htm;
}
```

**正则**
```
location / {
  if ($http_user_agent ~* chrome) {
      rewrite ^.*$ /chrome.html;
      #  不加 break; 会导致循环重定向， 页面现象， 500 内部服务器错误
      break;
  }
  root html;
  index index.html index.htm;
}
```

**-f-d-e**
除了 `nginx.conf`配置文件中可以看到变量
在文件`conf/fastcig.conf`查看全部nginx可以引用的变量.

`$fastcgi_script_name` 当前访问的uri.
例如：`/abc.html`

```
location {
    if (!-e $document_root$fastcgi_script_name) {
      rewrite ^.*$ /404.html;
      break;
    }
    root html;
    index index.html index.htm;
}
```

以`http://xxx.com/abcds.html`,不存在的页面为例.
注意：此处需要加`break`，因为观察访问日志，日志中显示的访问路径，依然是地址栏输入的uri.(GET /abcds.html HTTP/1.1)。
提示：服务器内部的`rewrite`和302跳转不一样。跳转的话是URL变化，重新发送请求404.html,而内部rewrite，仅仅是重新读取404.html的内容，上下文没有变化,就是 `fastcgi_script_name` 仍然是 `/abcds.html`,因此会循环重定向。


**set**
set是设置变量使用，可以达到多条件判断时做标记使用。
达到apache下的`rewrite_condition`的效果。


判断chrome浏览器，并且不用break。
```
if ($http_user_agent ~* chrome) {
  set $ischrome 1;
}

if ($fastcgi_script_name = chrome.html) {
  set $ischrome 0;
}

if ($ischrome 1) {
 rewrite ^.*$ chrome.html;
}
```

# 编译PHP

安装mysql：`yum install mysql mysql-devel`

nginx+php的编译.
apache一般是把php当作自己的一个模块来启动的，而nginx则是把http请求的变量(如get，user_agent等)转发给php进程，即PHP独立进程，与nginx进行通讯.称之为`fastcgi`运行方式.
因此，为此apache所编译的PHP，是不能用于nginx的，需要重新编译。

编译的PHP有如下功能：

1. 链接mysql
2. gd库
3. ttf字体
4. fpm(fastcgi)方式运行.


前期安装
```
> yum install gd
> yum install freetype
> yum install gd-devel
```
配置与安装
```
./configure --prefix=/usr/local/fastphp \
--with-mysql=mysqlnd \
--enable-mysqlnd \
--with-gd \
--enable-gd-native-ttf \
--enable-gd-jis-conv \
--enable-fpm

make && make install
```

编译完毕后：
```
> cp /usr/local/src/php-5.4.19/php.ini-development ./lib/php.ini

> cd etc/
> cp etc/php.fpm.conf.default etc/php-fpm.conf
```

运行：
```
> ./sbin/php-fpm
```

nginx与PHP是相互独立的，PHP以9000端口作为进程独立运行。nginx接收到请求，把请求转给PHP进程
配置核心：把请求的信息转发给9000端口的PHP进程，让PHP进程处理指定目录下的PHP文件.

修改nginx配置文件:`conf/nginx.conf`

```
location ~ \.php$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000; # 发送到9000端口进程
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  # 寻找PHP目录地址
    include        fastcgi_params;
}
```

解释:

1. 碰到PHP文件
2. 把根目录定位到html
3. 把请求上下文转交给9000端口的PHP进程。
4. 并告知PHP进程，当前脚本是`$document_root$fastcgi_script_name`(注：PHP会去寻找当前脚本位置，所以脚本位置要指对)
5. PHP处理之后，返回给nginx服务器.

 
# url重写
 
注意：用url重写，正则里如果有`{}`，整个正则需要使用`""`包起来

静态文件地址重写到动态目录地址
例如：`goods-10.html` 重写到 `goods.php?id=10`
```
location /zf {
    rewrite "goods-(\d{1,7}).html" /zf/goods.php?id=$1;
}
```

正则表达式支持`后向引用`

```
location /zf {
	index index.php;
	rewrite goods-([\d]+)\.html$ /zf/goods.php?id=$1;
	rewrite article-([\d]+)\.html$ /zf/article.php?id=$1;
	rewrite category-(\d+)-b(\d+)\.html /zf/category.php?=$1&barnd=$2;
}
```

通过两次路由对比写出正则
```
category.php?id=3&barnd=1&price_min=200&price_max=1700&filter_attr=167.229.202.199
category-3-b2-min200-max700-attr167.229.202.199.html

category-(\d+)-b(\d+)-min(\d+)-max(\d+)-attr([\d\.]+)\.html category.php?id=$1&barnd=$2&price_min=$3&$price_max=$4&filter_attr=$5;
```
-----
```
/category.php?id=3&barnd=2&price_min=0&price_max=0&page=2&sort=goods_id&order=DESC

/category-3-b0-min0-max9-attr0-1-goods_id-DESC.html

category-(\d+)-b(\d+)-min(\d+)-max(\d+)-attr([\d\.]+)-(\d+)-(\w+)-(\w+)\.html caetgory.php?id=$1&barnd=$2&price_min=$3&price_max=$4&filter_attr=$5&page=$6&sort=$7&order=$8.html
```

# gzip压缩

网页内容的压缩编码与传输速度优化

响应头中有：
```
Content-Encoding: gzip
```

原理：
浏览器-->请求-->声明可以接受gzip压缩或default压缩或compress或sdch压缩（sdch是google倡导的一种压缩方式，目前支持的服务器尚不多）
从http协议角度看，请求头，声明acceopt->encoding:gzip default sdch (是压缩算法)
服务器-->回应-->把内容gzip方式压缩-->发送浏览器 --> 接收gzip压缩内容（浏览器接收之后是压缩后的二进制文件）--> 解码gzip --> 浏览

> gzip常用参数

```
gzip on|off # 是否开启gizp
gzip_buffers 32 4K| 16 8K # 缓冲（压缩在内存中缓冲几块?每块多大?）
gzip_comp_level[1-9] # 推荐 6 压缩级别（级别越高，压缩越小，越浪费CPU计算资源）
gzip_disable # 正则匹配 #UA 什么样的Uri不进行gzip
gzip_min_length 200 # 开始压缩的最小长度
gzip_http_version 1.0|1.1 # 开始压缩的http协议的版本（如选1.1则满足1.1的才压缩）
gzip_proxied  # 设置请求着代理服务器如何缓存内容
gzip_types text/plain  appliction/xml #对哪些类型的文件用压缩 如txt,xml,html,css
gzip_vary on|off #是否传输gzip压缩标志
```    

写在server段上

```
server {
    gzip on;
    gzip_buffers 32 4K;
    gzip_comp_level 6;
    gzip_min_length 200;
    gzip_types text/css text/xml application/x-javascript;
}
```


注意：图片/mp3等的二进制文件，不必压缩，因为压缩比较小，比如100->80字节，而且压缩也是压缩也是耗费CPU资源


# expires缓存

expires缓存提升网站负载

nginx缓存设置，提高网站性能。
对于网站的图片，尤其是新闻站，图片一旦发布，改动的可能性非常小，用户访问一次之后，图片缓存到浏览器端，且时间比较长的缓存。使用到`nginx的expires`

nginx中设置过期时间，在`location`段，或`if`中写：
```
expires 30s;
expires 30m; // 2分钟过期
expires 2h; // 2小时过期
expires 30d; // 30天过期
```
-----
```
location ~* .\(git|jpg|jpeg|png) {
    root html;
    expires 1d; # 1天
}
```

注意：服务器的日期需要准确，如果服务器的日期和实际日期，可能导致缓存失效。

304也是一种缓存手段，原理是：服务器响应文件内容，同时响应etag标签（内容签名）和last_modified_since 2个标签值，浏览器下次请求时，浏览器要发送这两个头信息标签，服务器检测文件有没有发生变化，没有变化的话，返回头信息（etag和last_modified_since）浏览器知道内容无改变，于是直接调用本地缓存。

这过程中，也请求了服务器，但是传输内容极少，对于变化周期较短，如静态html，css。


# 反向代理

nginx反向代理服务器

使用nginx做反向代理和负载均衡支持两个用法：
1. proxy
2. upstream

nginx不自己处理php的相关请求，而是把php的相关转发给apache处理（php不让nginx跑）


![](./_image/2017-02-08-23-35-478.png)

这种形式，就是`动静分离`，更为严谨的说法是反向代理


注释掉nginx解析php的配置，通过代理形式转到apache解析php，（要保证apache能够支持php）
```
location ~ \.php$ {
    proxy_prss http://102.168.1.10:8000;
}
```

nginx服务器通过中间的代理，交代给谁完成任务，或者交代给几个人做，都是可行的。
把任务分配给多台机器，就是负载均衡。 


# 负载均衡

反向代理后端如果有多台服务器，自然可形成负载均衡。
但是`proxy_pass` 如何指向多态服务器？
把多台服务器用`up_stream`指定绑定在一起并起一个组名，然后`proxy_pass`指向该组.
配置服务器：
```
server {
    listen 81;
    server_name localhost;
    root /var/www/image;
}
server {
    listen 82;
    server_name localhost;
    root /var/www/image;
}
```
增加组：
```
#upstream 组名字 {
#    server ip wight=1(权重) fail_timeout=3(连接不上的时间) max_fails=2(连接不上的次数)
#}
upstream imageserver {
    server 192.168.1.100:81 weight=1 fail_timeout=3 max_fails=2;
    server 192.168.1.100:82 fail_timeout=3 max_fails=2;
}
```
然后修改反向代理配置代码:
```
location ~* \.(jpg|jpge|git|png) {
    proxy_pass http://imageserver;
}
```

默认的负载均衡的算法，就是正对后端服务器的顺序，逐个请求。也有其它的负载均衡算法，如一致性哈希，需要安装第三方模块`ngx_http_upstream_consistent_hash`。


问题：
反向代理导致了后端服务器的IP，为前端服务器的IP，而不是客户真正的IP
解决方式：
使用x-forwar
为了不改变用户的IP，需要通过变量存储IP带过去，一般约定俗成是通过头信息是：
```
proxy_set_header X-Forwarded-For $remote_addr;
```
-----
location ~* \.(jpg|jpge|git|png) {
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_pass http://imageserver;
}

# 连接memcached


> 编译或安装memcached

```
git clone https://github.com/memcached/memcached.git
cd memcached
sudo apt-get install autotools-dev
sudo apt-get install automake
./autogen.sh // 生成configure文件
./configure --with-php-config=/etc/php/7.1/fpm // 配置指定php的配置文件
make && make install
```

安装好之后提示的信息：
```
安装好后：Installing shared extensions:     /usr/local/sxin/php7/lib/php/extensions/no-debug-zts-20151012/
```
修改`php.ini`配置文件
```
entension=/usr/local/sxin/php7/lib/php/extensions/no-debug-zts-20170722/memcached.so
```
删除进程，重启php
```
pkill -9 php
```

测试是否成功：
```
<?php
phpinfo();
?>
```
在phpinfo中是否输出memcache模块


安装过程中出现的错误：


![](./_image/2565828848-5974246b75359_articlex.png)


方法1：
需要安装`libevent`，可能会报无法安装，使用wget方式安装
```
sudo apt-get install libevent libevent-deve
```
![](./_image/2024981432-597425cdd3632_articlex.png)

方法2：
```
wget http://monkey.org/~provos/libevent-1.4.14b-stable.tar.gz
tar -zxvf libevent-1.4.14b-stable.tar.gz 
cd libevent-1.4.14b-stable
./configure --prefix=/usr  
make && make install
```

> nginx直连memcached

原理：

![](./_image/1114550157-5973467c7f612_articlex.png)

- nginx需要设定key，去查memcached
- 如果不存在需要回调php，并把key值传给php    

nginx请求memcached时，用什么做key？
一般用 uri arg做key， 如`abc.php?id=2`

```
location / {
	set $memcached_key "$uri";
	memcacehd_pass 127.0.0.1:11211;
	error_page 404 /callback.php
}
```


问题：
如果多台memcaehed，某个key去请求那个memcached？
php又帮nginx把内容存储到哪个memcached？
回调的php，有把信息写在哪儿？
最终问题：多态memcached，nginx与php，如何保持集群上的算法同步.

- 要有稳定集群算法
- nginx与php对于memcached的算法要同步

解决：
nginx hash($uri) --> 某台memcached
php hash($uri) --> 同一台memcached
需要一个hash规则
这样才能保证分布式的数据同步。
默认的nginx分布式是通过计数器来轮流请求，需要第三方模块和一致性哈希应用

# 第3方模块编译

第3方模块编译和一致性哈希应用


> 编译第三方模块   

下载模块，然后重新指定nginx编译参数  

```
./configure --prefix=/usr/local/nginx --add-module=/usr/local/src/ngx_http_consistent/hash/
make && make install
```


> 安装模块后

```
upstream memserver {
	consistent_hash $requrest_uri;
	#server localhost:11211;
	#server localhost:11212;
	#server localhost:11213;
	server 192.168.1.100:11211;
	server 192.168.1.100:11212;
	server 192.168.1.100:11213;
}

server {
	listent 80;
	server_name localhost;
	location / {
		set $memcached_key $request_uri;
		memcached_pass memserver;
		error_page 404 /callback.php;
	}
}
```

在php.ini中修改默认hash取模的策略
```
memcache.hash_strategy=consistent
```

php代码

```
<?php
// 添加多台服务器, // php要添加和nginx一样 的服务器

$mem = new memcache();
// $mem->addServer('localhost', 11211);
// $mem->addServer('localhost', 11212);
// $mem->addServer('localhost', 11213);

$mem->addServer('192.168.1.100', 11211);
$mem->addServer('192.168.1.100', 11212);
$mem->addServer('192.168.1.100', 11213);
```

注意：
在 upstream做负载均衡时，要用IP或远程主机名，不能使用localhsot

# 大访问量优化

大访问量优化整体思路

高性能的服务器的架设

网站的请求量是绝对的，很难降下来。

对于高性能网站，请求量大，如何支撑？
- 要减少请求
	对于开发人员 -- 合并css，背景图片，压缩文件，减少mysql查询
- nginx的expiress，利用浏览器缓存等，减少查询
- 利用cdn来响应请求
- 最终剩下来的，不可避免的请求 -- 服务器集群 + 负载均衡来支撑

> 如何响应高并发请求

既然响应是不可避免的，要做的是把工作内容“平均”分给每台服务器，最理想的状态：每台服务器的性能都被充分利用。

服务器讲究:
像存储数据的，cpu不一定要强，但是硬盘一定要好，安装了ssd固态硬盘。
有的计算复杂的，cpu要高服务器
有的计算不复杂，但是进程多，内存多的服务器

服务器分清：计算密集，IO密集，进程密集。

> nginx观察统计模块

```
./configure --prefix=/usr/local/nginx/ --add-module=/usr/local/ngx_http_consistent_hash_master --with-http_stub_status_module
make && make install
```

修改nginx.conf,配置统计模块参数
```
location /status {
	stub_status on;
	access_log off;
	allow 192.168.1.100;
	deny all;
}
```

`ab工具`压力测试，观察数据


> 优化思路

对于nginx 来说，nginx请求，来响应。访问，mysql或硬盘中的.html文件等等

1. 建立socket连接
2. 打开文件，并沿socket返回


建立`socket`连接能否很多，打开文件是否能够打开很多。


socket中处理的包括：`系统层面`和`nginx`
系统层面：
```
最大连接数 somaxconn
洪水攻击 // 不做洪水抵御 
加快tcp连接回收 recycle
空的tcp是否允许回收利用 reuse
```
-----
```
echo 50000 > /proc/sys/net/core/somaxconn
echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse

echo 0 > /proc/sys/net/ipve4/tcp_syncookies
```
nginx：
```
 // 每个子进程打开的连接（worker_connections）
evnets {
	worker_connnections 10240;
}
//  http连接关闭 
http {
    keepalive_timeout 0;
}
```

文件中处理的包括：`系统层面`和`nginx`
系统层面的限制：
```
ulimit -n 50000 // 设置一个比较大的值
```
nginx：
```
// 子进程允许打开的文件（worker_limit_onfile）
// nginx配置文件中的全局区修改
worker_limit_nofile 10000;
```

响应头中:`Connection: keep-alive`
防止频繁的tcp,还保持时间的话，就浪费
```
http {
    keepalive_timeout 0;
}
```
时间修改为0之后，`Connection: close`

