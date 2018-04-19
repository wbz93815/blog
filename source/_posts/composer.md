---
title: 关于Composer的安装和使用
date: 2018-04-19 12:03:15
tags:
- Composer
- php
- 技术
- 2018
---

> 搞了多年的PHP的研发，针对Composer却还是那么熟悉又陌生，很是尴尬；由于最近要入坑Laravel，所以再了解下它的“前世今生”。

**Composer是什么**

Composer 是由国外的两位PHP大佬（Nils Adermann && Jordi Boggiano）开发，并且于2012年3月1日发布。

Composer 是PHP包的管理器，说白了就是PHP项目依赖包的仓库。它是为了`解决项目中需要引入各种版本的库`而生。

**安装方式**

Composer 的安装方式分为两种：局部安装 和 全局安装。所谓局部安装，即：只可在Composer的安装目录中使用；反之，则是在系统的任何位置都可使用。

<!-- more -->

**Linux下全局安装Composer**

> 安装前置条件：PHP版本需要 5.3.2+ 以上。

1. 下载安装脚本 composer-setup.php。（注：我的PHP版本是7.1，所以是php7）
```
[root@web ~]$ cd /home/baozhou.wang/src
[baozhou.wang@web src]$ php7 -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
```

2. 执行安装过程，生成可执行文件 composer.phar
```
[baozhou.wang@web src]$ php7 composer-setup.php 
All settings correct for using Composer
Downloading...

Composer (version 1.6.4) successfully installed to: /home/baozhou.wang/src/composer.phar
Use it: php composer.phar
```

3. 删除安装脚本
```
php7 -r "unlink('composer-setup.php');"
```

4. 将可执行文件composer.phar 安装到系统环境变量 PATH 所包含的路径下面
```
[baozhou.wang@web src]$ mv ./composer.phar ../bin/composer
```

5. 输入命令 composer -v，打印出以下开头的信息，则代表安装成功。
```
[baozhou.wang@web ~]$ composer -v
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 1.6.4 2018-04-13 12:04:24
```

> 关于Composer更全面的资料，请参考 [Composer 中文官网](https://www.phpcomposer.com)