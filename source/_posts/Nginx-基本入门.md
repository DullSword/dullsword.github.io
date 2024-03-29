---
title: Nginx 基本入门
comments: true
toc: true
excerpt: ' '
date: 2019-10-31 15:04:19
categories: Nginx
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： somnus_小凯
原文标题： Nginx基本入门
链接： https://blog.csdn.net/u012486840/article/details/53098890
来源： CSDN博客
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

# 静态HTTP服务器

首先，Nginx是一个HTTP服务器，可以将服务器上的静态文件（如HTML、图片）通过HTTP协议展现给客户端。

配置：

``` nginx
server {
    listen 80; # 端口号
    location / {
        root /usr/share/nginx/html; # 静态文件路径
    }
}
```

# 反向代理服务器

什么是反向代理？

客户端本来可以直接通过HTTP协议访问某网站应用服务器，如果网站管理员在中间加上一个Nginx，客户端请求Nginx，Nginx请求应用服务器，然后将结果返回给客户端，此时Nginx就是反向代理服务器。

![](reverse_proxy.png)

配置：

``` nginx
server {
    listen 80;
    location / {
        proxy_pass http://192.168.0.112:8080; # 应用服务器HTTP地址
    }
}
```

既然服务器可以直接HTTP访问，为什么要在中间加上一个反向代理，不是多此一举吗？反向代理有什么作用？继续往下看，下面的负载均衡、虚拟主机，都基于反向代理实现，当然反向代理的功能也不仅仅是这些。

# 负载均衡

当网站访问量非常大，也摊上事儿了。因为网站越来越慢，一台服务器已经不够用了。于是将相同的应用部署在多台服务器上，将大量用户的请求分配给多台机器处理。同时带来的好处是，其中一台服务器万一挂了，只要还有其他服务器正常运行，就不会影响用户使用。

Nginx可以通过反向代理来实现负载均衡。

![](load_balancing.jpg)

配置：

``` nginx
upstream myweb {
    server 192.168.0.111:8080; # 应用服务器1
    server 192.168.0.112:8080; # 应用服务器2
}
server {
    listen 80;
    location / {
        proxy_pass http://myweb;
    }
}
```

# 虚拟主机

有的网站访问量大，需要负载均衡。然而并不是所有网站都如此出色，有的网站，由于访问量太小，需要节省成本，将多个网站部署在同一台服务器上。

例如将`www.aaa.com`和`www.bbb.com`两个网站部署在同一台服务器上，两个域名解析到同一个IP地址，但是用户通过两个域名却可以打开两个完全不同的网站，互相不影响，就像访问两个服务器一样，所以叫两个虚拟主机。

配置：

``` nginx
server {
    listen 80 default_server;
    server_name _;
    return 444; # 过滤其他域名的请求，返回444状态码
}
server {
    listen 80;
    server_name www.aaa.com; # www.aaa.com域名
    location / {
        proxy_pass http://localhost:8080; # 对应端口号8080
    }
}
server {
    listen 80;
    server_name www.bbb.com; # www.bbb.com域名
    location / {
        proxy_pass http://localhost:8081; # 对应端口号8081
    }
}
```

在服务器`8080`和`8081`分别开了一个应用，客户端通过不同的域名访问，根据`server_name`可以反向代理到对应的应用服务器。

虚拟主机的原理是通过HTTP请求头中的`Host`是否匹配`server_name`来实现的，有兴趣的同学可以研究一下HTTP协议。

另外，`server_name`配置还可以过滤有人恶意将某些域名指向你的主机服务器。

# FastCGI

Nginx本身不支持PHP等语言，但是它可以通过`FastCGI`来将请求扔给某些语言或框架处理（例如PHP、Python、Perl）。

``` nginx
server {
    listen 80;
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME /PHP文件路径$fastcgi_script_name; # PHP文件路径
        fastcgi_pass 127.0.0.1:9000; # PHP-FPM地址和端口号
        # 另一种方式：fastcgi_pass unix:/var/run/php5-fpm.sock;
    }
}
```

配置中将`.php`结尾的请求通过`FashCGI`交给`PHP-FPM`处理，`PHP-FPM`是PHP的一个`FastCGI`管理器。有关`FashCGI`可以查阅其他资料，本文不再介绍。

`fastcgi_pass`和`proxy_pass`有什么区别？下面一张图带你看明白：

![](fastcgi_pass_and_proxy_pass.jpg)