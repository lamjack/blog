---
title: GO Interface
date: 2018-04-10 00:13:19
categories:
  - 编程语言
  - GO
tags:
  - interface
  - go
---
![duck](https://jack-images.wilead.net/blog/f4exl.jpg)
<!--more-->

## 各种语言的duck typing实现比较

### Python中的duck typing

```python
def download(retriever):
    return retriever.get("lamjack.github.io")
```

- **运行时**才知道传入的 retriever 有没有get方法（写代码时不知道）
- 需要注释来说明接口

### C++中的duck typing

```c++
template <class R>
string download(const R& retriever) {
    return retriever.get("lamjack.github.io");	
}
```

- **编译时**才知道传入的retriever有没有get方法（写代码时不知道）
- 需要注释来说明接口

### Java中的类似代码

```java
<R extends Retriever>
String download(R r) {
	return r.get("lamjack.github.io");
}
```

- 传入的参数必须实现Retriever接口

- 不是duck typing（必须实现某个接口 = 大黄鸭必须属于某个物种定义才是鸭的传统定义一样）

### go语言的duck typing

- 可以同时组装多个接口
- 同时具有python,c++的duck typing的灵活性
- 具有java的类型检查

**Go的接口由<span style="color: red;">使用者</span>定义**

**在Go语言中，一个类<span style="color: red;">只需要实现了接口所需要的所有函数</span>，我们就说这个类实现了该接口**，即接口的实现是**隐式的**。

假设我们定义了下面几个接口，

```go
type IFile interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
    Seek(off int64, whence int) (pos int64, err error)
    Close() error
}

type IReader interface {
    Read(buf []byte) (n int, err error)
}

type IWriter interface {
    Write(buf []byte) (n int, err error)
}

type ICloser interface {
    Close() error
}
```

然后我们有一个类，

```go
type File struct {
    // ...
}

func (f *File) Read(buf []byte) (n int, err error) {...}
func (f *File) Write(buf []byte) (n int, err error) {...}
func (f *File) Seek(off int64, whence int) (pos int64, err error) {...}
func (f *File) Close() error {...}
```

这个时候我们就可以说这个类实现了IFile、IReader、IWriter、ICloser多个接口，

```go
var file1 IFile = new(File)
var file2 IReader = new(File)
var file3 IWriter = new(File)
var file4 ICloser = new(File)
```

另外，接口可以组合，比如这样写，

```go
type IFile interface {
    IReader
    IWriter
    ICloser
}
```

## 接口赋值

### 将对象实例赋值给接口

首先对象赋值要遵循一个原则，**赋值符号的左值和右值是同种类型。**PHP等存在隐性类型转换的语言实际上也是先做右边的类型转换再做赋值对吧？

假设有如下代码，

```go
type Integer int

func (a Integer) Less(b Integer) bool {
    return a < b
}

func (a *Integer) Add(b Integer) {
    *a += b
}

type LessAdder interface {
    Less(b Integer) bool
    Add(b Integer)
}
```

然后进行赋值，

```go
var a Integer = 1
var b LessAdder = &a // 1
var b LessAdder = a  // 2
```

先说结论，上面两个赋值语句，只有(1)能做正确赋值。原因在于下面的函数，

```go
func (a Integer) Less(b Integer) bool
```

Go语言可以根据这个函数，自动生成一个新的Less()方法，

```go
func (a *Integer) Less(b Integer) bool
```

这样*Integer就既存在Less()方法，也存在Add方法，

### 将一个接口赋值给另外一个接口

## 接口查询

## Any类型

由于Go语言任何对象实例都满足空接口interface{}，所以interface{}看起来像是可以指向任何对象的Any类型，

``` go
var v1 interface{} = 1
var v2 interface{} = "abc"
var v3 interface{} = $v2
var v4 interface{} = struct{ X int }{1}
var v5 interface{} = &struct{ X int }{1}
```

