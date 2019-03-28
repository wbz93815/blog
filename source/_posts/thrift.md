---
title: Thrift框架的介绍及使用
date: 2018-03-28 14:59:36
categories:
- 技术
tags:
- Thrift
- RPC
- 2018
---
# Thrift介绍

**Thrift** 是一种接口描述语言和二进制通讯协议，由Facebook团队开发，但是由 Apache软件基金会 进行开源；底层是由C++编写。
>个人比较粗浅的理解：专门用于跨语言之前的服务调用框架，支持多种开发语言，如：PHP、Java、Go、C++等等。

# 安装Thrift

由于我自己安装过程中，遇到了很多异类的错误，这里就不详述，以免误人子弟；这里就提供下官方给的安装说明吧：
Centos6.5下安装方式，请戳 [链接](https://thrift.apache.org/docs/install/centos)
Windows下安装方式，请戳 [链接](https://thrift.apache.org/docs/install/windows)
Mac os x下安装方式，请戳 [链接](https://thrift.apache.org/docs/install/os_x)
Debian、Ubuntu下安装方式，请戳[链接](https://thrift.apache.org/docs/install/debian)

<!-- more -->

# Thrift 数据类型

**基本类型**

> bool：布尔值
> byte：8 位有符号整数
> i16：16 位有符号整数
> i32：32 位有符号整数
> i64：64 位有符号整数
> double：64 位浮点数
> string：未知编码文本或二进制字符串

**结构类型**

> struct：定义公共的对象

**容器类型**

> list&lt;T&gt;：有序列表，数据类型是T，元素可以重复
> set&lt;T&gt;：无序集合，数据类型是T，元素不可以重复
> map&lt;K, V&gt;：字典类型，K为类型，V为值

**异常类型**

> exception：异常的类型

**服务类型**

> service：对应的服务类，类似于接口Interfacce

# 编写 Thrift 代码

案例代码如下：

```
// 这里的py，代表生成的PYTHON的代码
namespace py RTB 

struct ScanData {
    1: required string macAddress,
    2: required string rssi,
}

// 这里接口的请求参数
struct Request {
    1: optional list<ScanData> allScanData, // 这里是对象的集合；optional 代表可不传递
    2: required i32 deviceId,
}

// 这里是抛出的错误定义
exception RequestException {
    1: required i32 code,
    2: optional string message,
}

// 这里类似定义的接口类
service RtbService {
    // 这里就是你业务中，要实现的接口函数
    string getRtbByDeviceIdAndScanData(1: Request request) throws (1:RequestException requestException);
}
```

生成可用于业务项目中的代码 Client or Server 

```
thrift --gen python hello.thrift
```

# Thrift 传输协议

**TBinaryProtocol** 使用二进制格式的传输协议，比文本协议处理更快，简单。
**TCompactProtocol** 使用VLQ编码进行压缩的传输协议。节省传输空间，传输效率更高。
**TJSONProtocol** 使用Json格式的传输协议。
**TSimpleJSONProtocol**  使用Json只写协议，它不能被Thrift解析，因为使用json时，会丢弃元数据，适合用脚本语言来解析。
**TDebugProtocol** 使用文本传输协议。用于调试。

# Thrift 传输方式

**TFileTransport** 使用文件方式进行传输，该传输协议会写文件。
**TSocket** 使用阻塞式的I/O进行传输。
**TFramedTransport** 使用非阻塞式的方式，按块的大小进行传输。
**TMemoryTransport** 使用内存I/O的方式进行传输。
**TZlibTransport** 使用zlib执行压缩，再进行传输。

# Thrift 服务类型

**TSimpleServer**  一个单线程服务类型，使用标准的阻塞式I/O。 
**TThreadPoolServer** 一个多线程服务类型，使用标准的阻塞式I/O，预先创建一组线程处理请求。
**TNonblockingServer** 一个多线程服务类型，使用非阻塞I/O；Client和Server都需要使用TFramedTransport传输方式。

# 应用 Thrift

**服务端编码步骤**
- 定义处理器（创建TProcessor）
- 定义传输方式（创建TServerTransport）
- 定义传输协议（创建TProtocol）
- 定义服务类型(创建TServer)
- 启动服务。

**客户端编码步骤**
- 定义传输方式（创建TServerTransport）
- 定义传输协议（创建TProtocol）
- 创建Client（基于TServerTransport、TProtocol）
- 调用Client的方法

> 提供了PHP、和Python使用Thrift的Demo； 请戳 [PHP链接](https://github.com/wbz93815/php-thrift)，请戳 [Python链接](https://github.com/wbz93815/python-thrift)。
