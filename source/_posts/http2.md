---
title: 博客升级Http2.0的过程
date: 2018-04-08 18:20:28
tags:
- HTTP2.0
- Nginx
- 技术
- 2018
---

**HTTP 2.0** 于2015年5月问世，距离上一个版本（HTTP 1.1）已有16年之久。然后，现在很多企业仍在使用1.1的版本，估计是针对这类需求不大；而我这次升级h2的初衷，主要有两点：
- 自主配置学习
- 提升博客打开速度

**简述HTTP 2.0特性**

- `使用二进制格式传输数据` 也就是0和1的组合；而1.x的版本是文本的格式，野史称：文本格式解析存在缺陷，文本的表现形式存在多样性，无法兼容各种各样的场景。
- `连接共享（多路复用）` 直白的理解：所有请求都可以在一个TCP连接并发完成；而1.x的版本，是建立N个链接来完成N个请求。
- `Header压缩` 在1.x版本中，header中的Cookie和UserAgent很容易膨胀，每次都要重复发送；因此，2.0版本client、server端都缓存了一份header数据，同时，还使用了压缩算法对header进行压缩，避免了重复header的传输，减少了数据传输的大小。
- `服务端推送` Client请求Server页面内容时，Server会自动将页面所需要的其它资源发送给Client。举个例子：网页上有一个style.css的请求，在Client收到style.css数据的同时，Server会将style.js的文件推送给Client，当Client再次尝试获取style.js时就可以直接从缓存中获取到，不用再发请求了。

<!-- more -->

**配置过程**

`安装依赖`

由于我的web服务器用的是Nginx，因此就它为例；要使用http2.0，须依赖两个模块 http_ssl_module（https）、http_v2_module（http 2.0）。安装方式分两种：

`全新安装Nginx`

编译安装时，在./configure后加上 **--with-http_ssl_module**、 **--with-http_v2_module**，然后继续执行make、make install等之类的操作，这里不再详细描述。

`已安装Nginx，但缺少两个模块`

1、先通过nginx -V，获取历史安装的配置信息，如下：
```
[root@VM_0_4_centos nginx-1.13.9]# nginx -V
nginx version: nginx/1.13.9
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx-1.13.9 --error-log-path=/data/log/nginx/error.log --http-log-path=/data/log/nginx/access.log --with-http_ssl_module --with-http_dav_module --with-http_flv_module --with-http_realip_module --with-http_gzip_static_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-debug
```

2、在源码目录中，将这两个模块加上，再次进行编译，如下：
```
[root@VM_0_4_centos nginx-1.13.9]# ./configure --prefix=/usr/local/nginx-1.13.9 --error-log-path=/data/log/nginx/error.log --http-log-path=/data/log/nginx/access.log --with-http_ssl_module --with-http_dav_module --with-http_flv_module --with-http_realip_module --with-http_gzip_static_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-debug --with-http_v2_module
```

3、执行make就行，`不能执行make install`，否则就是重新安装Nginx。

4、备份安装目录中的Nginx，拷贝源码目录中的Nginx到安装目录sbin中，如下：
```
[root@VM_0_4_centos nginx-1.13.9]# mv /usr/local/nginx-1.13.9/sbin/nginx /usr/local/nginx-1.13.9/sbin/nginx.bak
[root@VM_0_4_centos nginx-1.13.9]# cp ./objs/
autoconf.err        nginx               ngx_auto_config.h   ngx_modules.c       src/                
Makefile            nginx.8             ngx_auto_headers.h  ngx_modules.o       
[root@VM_0_4_centos nginx-1.13.9]# cp ./objs/nginx /usr/local/nginx-1.13.9/sbin/
```

5、在conf文件中，server下的listen项中，追加**http2**，如下：
```
[root@VM_0_4_centos nginx-1.13.9]# vi conf/vhost/hexo-blog.conf 
server {
    listen 443 ssl http2;
    server_name  baozhou.wang;
    charset utf-8;
    root /data/www/baozhou-blog/public;

    ssl on;
    ssl_certificate /data/certificate/baozhou.wang/baozhou.wang_bundle.crt;
    ssl_certificate_key /data/certificate/baozhou.wang/baozhou.wang.key;
}
``` 

6、最后，测试配置无误后，然后重启Nginx。
> 这里必须先kill掉Nginx进程，再启动，才能生效http2.0；我使用nginx -s reload始终不生效。

**性能对比**

通过下图(分别是使用http2.0之前、使用http2.0之后)，`页面打开速度明显提升了12.6%`。
<center>![使用http2.0之前](http://img.baozhou.wang/2018/http2_before.png "使用http2.0之前")</center>
<center>![使用http2.0之后](http://img.baozhou.wang/2018/http2_after.png "使用http2.0之后")</center>

