---
title: 总结今日安装SS遇到的坑
date: 2018-04-27 16:14:38
tags:
- 技术
- 2018
---

> 由于SS版本升级，导致安装过程中，步步都是坑。内心中一万句：“草泥马”。

然后各种查资料，还好都解决了。具体错误如下：

1、**编译提示错误：`configure: error: mbed TLS libraries not found.`**

**解决方案**：是因为执行过程系统找不到 mbedtls 的扩展。由于我是编译安装的 mbedtls；因此，只要在编译过程中，指定 mbedtls 安装目录相关文件目录的位置即可 `--with-mbedtls-include=/usr/local/mbedtls-2.4.2/include/ --with-mbedtls-lib=/usr/local/mbedtls-2.4.2/lib/`

---------

2、**编译提示错误：`configure: error: The Sodium crypto library libraries not found.`**
<!-- more -->
**解决方案**：是因为执行过程系统找不到 libsodium 的扩展。由于我是编译安装的 libsodium；因此，只要在编译过程中，指定 libsodium 安装目录相关文件目录的位置即可 `--with-sodium-include=/usr/local/libsodium-1.0.12/include/ --with-sodium-lib=/usr/local/libsodium-1.0.12/lib/`。

---------

3、**编译提示错误：` configure: error: The c-ares library libraries not found. `**

**解决方案**：是因为执行过程系统找不到 c-ares 的扩展。由于我是编译安装的 c-ares；因此，只要在编译过程中，指定 c-ares 安装目录相关文件目录的位置即可：`--with-cares-include=/usr/local/c-ares-1.12.0/include/  --with-cares-lib=/usr/local/c-ares-1.12.0/lib/`

---------

4、**启动提示错误：` /usr/local/ss-libev/bin/ss-server: error while loading shared libraries: libmbedcrypto.so.0: cannot open shared object file: No such file or directory `**

**解决方案**：执行以下命令即可
```
[root@server ~]# echo /usr/local/libsodium-1.0.12/lib >> /etc/ld.so.conf.d/local.conf
[root@server ~]# ldconfig
```

---------

5、**启动提示错误：` /usr/local/ss-libev/bin/ss-server: error while loading shared libraries: libcares.so.2: cannot open shared object file: No such file or directory `**

**解决方案**：执行以下命令即可
```
[root@server ~]# echo /usr/local/c-ares-1.12.0/lib/ >> /etc/ld.so.conf.d/local.conf
[root@server ~]# ldconfig
```

---------

6、**启动提示错误：` /usr/local/ss-libev/bin/ss-server: error while loading shared libraries: libmbedcrypto.so.0: cannot open shared object file: No such file or directory `**

**解决方案**：执行以下命令即可
```
[root@server ~]# echo /usr/local/mbedtls-2.4.2/lib >> /etc/ld.so.conf.d/local.conf 
[root@server ~]# ldconfig
```

---------

7、**启动成功后，当客户端发起请求时，提示错误：`ERROR: connect: Invalid argument `**

**解决方案**：删除自定义配置文件中的 local_ 开头的两行（即：`local_address` 和 `local_port`）。
