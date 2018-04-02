```
[root@10-19-121-25 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G   19G   48M 100% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/vdb         59G   52M   56G   1% /data
```

```
[root@10-19-121-25 rabbitmq]# ll
total 15733292
-rw-r--r-- 1 root root 16110774320 Apr  2 14:30 rabbit@10-19-121-25.log
-rw-r--r-- 1 root root      103023 Apr  2 10:45 rabbit@10-19-121-25-sasl.log
```

```
[root@10-19-121-25 rabbitmq]# rm rabbit\@10-19-121-25.log
```

```
[root@10-19-121-25 ~]# lsof | grep deleted
beam.smp  18623 root   48w      REG              252,1 16110774320     285331 /usr/local/rabbitmq/var/log/rabbitmq/rabbit@10-19-121-25.log (deleted)
```

```
[root@10-19-121-25 ~]# kill 18623
[root@10-19-121-25 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  3.6G   16G  19% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
/dev/vdb         59G   52M   56G   1% /data
```

```
[root@10-19-121-25 ~]# ps -ef | grep zabbix
root      8582  8148  0 14:48 pts/0    00:00:00 grep zabbix
```

```
[root@10-19-121-25 ~]# ps -ef | grep zabbix
root      8582  8148  0 14:48 pts/0    00:00:00 grep zabbix
```

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
控制日志的粒度.其值是日志事件类别(category)和日志级别(level)成对的列表．
level 可以是 'none' (不记录日志事件), 'error' (只记录错误), 'warning' (只记录错误和警告), 'info' (记录错误，警告和信息), or 'debug' (记录错误，警告，信息以及调试信息).

目前定义了４种日志类别. 它们是：

channel -针对所有与AMQP channels相关的事件
connection - 针对所有与网络连接相关的事件
federation - 针对所有与federation相关的事件
mirroring -针对所有与 mirrored queues相关的事件
Default: [{connection, info}]

