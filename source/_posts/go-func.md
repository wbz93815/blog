---
title: 快速理解Go语言中的方法
date: 2019-03-31 18:30:12
categories:
- 技术
tags:
- 方法
- go
- 2019
---

在go语言中，我个人针对类、方法的理解是这样的：结构体就是面向对象中的类，方法就是面向对象中类的方法。

### 方法的申明

#### 方法的定义

```
func (recv receiver_type) methodName(parameter_list) [return_type] { ... }
```

针对上面格式中的关键部分，我的理解是这样的：

**recv** 接收者的一个实例；也就是要通过 recv.methodName() 才能调用。
**receiver_type** 接收者类型。
**methodName** 接收者类型的方法名。
**parameter_list** 方法的入参。
**return_type** 方法的出参类型。

<!-- more -->

#### 方法的重载

方法的重载，有个至关重要的条件：**一个接收类型只能有一个同名的方法**。下面这样的重载是合法的：

```
func (a A) open(a1 AA) { ... }
func (b B) open(a1 AA) { ... }
```

### 方法的使用

#### 函数与方法的使用方式

在go中，函数与方法都支持，但是两者的区别在于：**函数是将变量作为参数**、**方法是在变量上被调用**。

* 函数（基于面向过程），使用方式如下：

```
type Integer int 

func Less(a Integer, b Integer) bool {
    return a < b 
}

func main() {
    var a Integer = 1 

    if Less(a, 2) {
        fmt.Println("less then 2")
    }   
}
```

* 方法（基于面向对象）的使用方式如下：

```
type Integer int 

func (a Integer) Less(b Integer) bool {
    return a < b 
}

func main() {
    var a Integer = 1 

    if a.Less(2) {
        fmt.Println("less then 2")
    }   
} 
```

#### 初始化方法（构造函数）的使用方式

在go语言中，要使用构造函数，那么**函数名必须以New开头**、结构体的名字首字母要大写。
具体示例：
```
// File开头大写
type File struct {
	 fd int
	 name string
}

// New开头
func NewFile(fd int, name string) *File {
	if fd > 0 {
		return nil
	}

	return &File{fd, name}
}

func main() {
    f := NewFile(10, "../test.txt")
}
```

#### 类似于面向对象中Setter、Getter方法

在go中，也有类似面向对象的get、set方法。但是在使用上有一些限制，针对**setter方法，要使用Set前缀**；针对**getter方法，使用成员名**（也就是结构体中的字段）。

示例代码：

```
type Person struct {
	firstName string
}

func (p *Person) FirstName() string {
	return p.firstName
}

func (p *Person) SetFirstName(firstName string) string {
	p.firstName = firstName
}
```

#### 类似于面向对象的继承特性

Go还支持面向对象的继承特性，而且比其它语言更牛逼一下。很多语言只能继承一个类，而在Go支持多个（使用的也就是嵌入类型的方式）。

具体看代码：

```
type Point struct {
	x, y float64
}

func (p *Point) Abs() float64 {
	return math.Sqrt(p.x * p.y + p.y * p.y)
}

type NamePoint struct {
	Point
	name string
}

func main() {
    n := &NamePoint{Point{3,4}, "abcName"}
    fmt.Println(n.Abs())
}
```