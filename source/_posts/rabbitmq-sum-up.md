---
title: 总结今日RabbitMQ线上故障修复的过程
date: 2018-04-02 16:22:28
tags:
- RabbitMQ
- 技术
- 总结
- 2018
---

愚人节过后的第一个工作日，生产环境就发生了关于RabbitMQ的故障。
> 值得庆幸的是，这个bug是我主动发现了，然后被我偷偷摸摸的修复了，而且故障的时段对业务没有影响。

**发现问题**

偶然间，我瞄了一眼生产环境RabbitMQ的图形化界面，发现了两个症状：

1. 消息曲线图没有波动了（也就是说，没有消息进来了）。
2. 图形化界面Node项下的Disk space，直接红了。

我第一反应，是肯定磁盘空间满了，要不然也不会出现症状二。

<!-- more -->

**定位过程**

先是，进入系统后，我立马查看了下空间，果不其然，真的满了。（前任运维真的很坑，就买了20G，系统本身的相关资源就占用了5G左右。第一反应先甩锅，哈哈～）

```
[root@10-19-121-25 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G   19G   48M 100% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/vdb         59G   52M   56G   1% /data
```

然后，更近一步的定位到具体的文件，发现是被RabbitMQ的日志文件占满了。

```
[root@10-19-121-25 rabbitmq]# ll
total 15733292
-rw-r--r-- 1 root root 16110774320 Apr  2 14:30 rabbit@10-19-121-25.log
-rw-r--r-- 1 root root      103023 Apr  2 10:45 rabbit@10-19-121-25-sasl.log
```
打开文件，发现都是一些业务机器与RabbitmMQ进行连接之类的数据，如下：

```
=INFO REPORT==== 2-Apr-2018::14:49:31 ===
accepting AMQP connection <0.1468.0> (10.19.116.71:44660 -> 10.19.121.25:5672)

=INFO REPORT==== 2-Apr-2018::14:49:31 ===
connection <0.1468.0> (10.19.116.71:44660 -> 10.19.121.25:5672): user 'advertisement' authenticated and granted access to vhost 'ad'

=INFO REPORT==== 2-Apr-2018::14:49:31 ===
closing AMQP connection <0.1468.0> (10.19.116.71:44660 -> 10.19.121.25:5672, vhost: 'ad', user: 'advertisement')
```

**快速解决**

为了保证业务能够正常的运行，我是先把日志文件干掉了。

```
[root@10-19-121-25 rabbitmq]# rm rabbit\@10-19-121-25.log
```

按理说，日志文件删除掉之后，空间应该释放掉；然而并不是这样。
在G哥上查了一下，发现了原因：

> 在Linux或Unix系统中，通过rm或文件管理器删除文件，只是将文件与目录结构解除链接(unlink)，然而如果文件还被某个程序打开着的话，那么该程序仍然能读取到数据，所以磁盘空间一直被占用着。

```
[root@10-19-121-25 ~]# lsof | grep deleted
beam.smp  18623 root   48w      REG              252,1 16110774320     285331 /usr/local/rabbitmq/var/log/rabbitmq/rabbit@10-19-121-25.log (deleted)
```

通过上面的命令，我找到了这个被我删除的文件。按照G哥的说法，先把对应的应用Kill掉，然后就可以释放空间了。

```
[root@10-19-121-25 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  3.6G   16G  19% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/vdb         59G   52M   56G   1% /data
```

我把对应的应用停掉后，发现真的管用了，空间也就释放出来了。`为了验证问题是否修复，又把对应的应用再启动起来，结果图形化管理界面终于有波动了 -- 问题修复了。`

**彻底解决**

上面的快速解决，只能“治标不治本”（因为很快磁盘空间又会被写满）；更彻底的方法，是修改日志记录的等级。

```
[root@10-19-121-25 rabbitmq]# cat var/log/rabbitmq/rabbit\@10-19-121-25.log | head -n 50

=INFO REPORT==== 2-Apr-2018::14:49:28 ===
Starting RabbitMQ 3.6.15 on Erlang 20.2
Copyright (C) 2007-2018 Pivotal Software, Inc.
Licensed under the MPL.  See http://www.rabbitmq.com/

=INFO REPORT==== 2-Apr-2018::14:49:28 ===
node           : rabbit@10-19-121-25
home dir       : /root
config file(s) : /usr/local/rabbitmq/etc/rabbitmq/rabbitmq.config (not found)
cookie hash    : Ayi7Hq26BFLtKASvASLSpg==
log            : /usr/local/rabbitmq/var/log/rabbitmq/rabbit@10-19-121-25.log
sasl log       : /usr/local/rabbitmq/var/log/rabbitmq/rabbit@10-19-121-25-sasl.log
database dir   : /usr/local/rabbitmq/var/lib/rabbitmq/mnesia/rabbit@10-19-121-25
```

当初不知道怎么安装的，在安装目录下，没有找到配置文件rabbitmq.config，最后还是通过日志文件才找到的路径（`config file(s) : /usr/local/rabbitmq/etc/rabbitmq/rabbitmq.config (not found)`），只不过显示配置文件不存在而已。于是，创建这个配置文件，并添加如下代码：

```
[root@10-19-121-25 rabbitmq]# vi rabbitmq.config 
[
 {rabbit,
   [
     {log_levels, [{connection, error}, {channel, error}]}
   ]
 }
].
```

> 关于log_levels，RabbitMQ默认为：{connection, info}

**RabbitMQ日志事件类别**
- channel：针对所有与AMQP channels相关的事件
- connection：针对所有与网络连接相关的事件
- federation：针对所有与federation相关的事件
- mirroring：针对所有与mirrored queues相关的事件

**RabbitMQ日志级别**
- none：不记录日志事件
- error：只记录错误
- warning：只记录错误和警告
- info：记录错误，警告和信息
- debug：记录错误，警告，信息以及调试信息

