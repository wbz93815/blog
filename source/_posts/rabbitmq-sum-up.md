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


