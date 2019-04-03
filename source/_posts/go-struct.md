---
title: 快速理解Go语言中的结构体
date: 2019-03-29 21:09:25
categories:
- 技术
tags:
- 结构体
- go
- 2019
---

#### 结构体

```
type MyStruct struct {
	Id int "编号"
	f1 float32 "价格"
	str string "介绍"
}
```

1. **结构体中的字段名首字母要大写**。因为小写开头，只能在包内引用。
2. 上面的结构体中字段的最后有汉字标签，这个一般用在文档说明，和字段注释的作用；这个标签的内容，可以通过reflect包来获取。
3. 结构体中的字段名不要重名。

<!-- more -->

#### 创建结构体的方式

```
// 方式1：
var s3 *MyStruct
s3 = new(MyStruct)

// 方式2：
var s5 MyStruct
```

针对上面两种方式，理解了下：
方式1：创建指向结构体（MyStruct）类型变量的指针s3，给s3分配内存，初始化s3中字段的值为所属类型的零值。
方式2：创建一个结构体类型的变量s5，给s5分配内存，初始化s5中字段的值为所属类型的零值。

#### 初始化结构体

结构体的初始化方式，也有三种方式： **new(结构体类型)** 、 **&结构体类型{}** 、**结构体类型{}**。
示例代码如下：

```
// 方式1：
s := new(MyStruct)
s.Id = 100
s.f1 = 28.8
s.str = "这是一段字符串"

// 方式2：
s1 := &MyStruct{11, 12.21, "字符串"} // 顺序不能乱
s1 := &MyStruct{Id:11, f1:12.21} // 指定字段名，初始化；

// 方式3：
var s5 MyStruct
s2 = MyStruct{11, 12.21, "字符串"} // 顺序不能乱
```

#### 使用结构体

在函数间传递结构体参数，也有两方式 **形式参数传输** 和 **指针参数传输** 。
**形式参数传输** 也就是传递一个变量的副本到另一个函数中。
**指针参数传输** 也就是传递一个指针的地址到另一个函数中。

具体看下代码：

```
// 形式传参
func sell(ms MyStruct) {
    // todo
}

// 指针传参
func sell(ms *MyStruct) {
    // todo
}
```

#### 有坑

```
type One struct {
	a int
}

type Two struct {
	a, b int
}

type Three struct {
	One
	Two
}

func main() {
	var three Three
	three.a = 10
	fmt.Println(three.a)
}

输出结果：
# command-line-arguments
./hello.go:37:19: ambiguous selector three.a
```

以上是嵌套结构体。之所以说它有坑，主要体现在字段命名冲突的时候。
具体有以下两种场景：
1. 外层结构体中字段的名字，会覆盖内层字段的名字；两者都会在内存空间都会保留。
2. 重名的字段，如果被程序调用了，则会提示错误；如果没有被调用，则不提示错误。