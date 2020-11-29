---
title: Mac下破解GitKraken 7.4.0
date: 2020-11-29 18:50:00
categories:
- 技术
tags:
- 2020
- 破解
- 软件
- mac
---

### 安装Node、yarn

```
brew install node
brew install yarn
```

### 屏蔽更新

绑定 Host.
```
127.0.0.1 release.gitkraken.com
```

<!-- more -->

### 登陆 gitkraken

可以自己注册一个账号

### 破解

#### 下载破解项目进行编译
```
git clone git@github.com:wbz93815/GitCracken.git
cd GitCracken/GitCracken
rm yarn.lock
yarn install
yarn build
```

#### 执行破解命令
```
# windows下这个命令也支持
node dist/bin/gitcracken.js patcher --asar gitkraken的安装目录/resources/app.asar
```

<center>![执行破解gitkraken命令之后](https://file.baozhou.wang/2020/1129.png "执行破解gitkraken命令之后")</center>

#### 重启软件，即可看到右下角`PRO`