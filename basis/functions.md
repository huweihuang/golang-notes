# 函数

## 1. 函数定义与调用

```go
//1、函数组成：关键字func ,函数名，参数列表，返回值，函数体，返回语句
//先名称后类型
func 函数名(参数列表)(返回值列表){  //参数列表和返回值列表以变量声明的形式，如果单返回值可以直接加类型
  函数体
  return    //返回语句
}
//例子
func Add(a,b int)(ret int,err error){
  //函数体 
  return   //return语句
}

//2、函数调用
//先导入函数所在的包，直接调用函数
import "mymath"
sum,err:=mymath.Add(1,2)   //多返回值和错误处理机制
//可见性，包括函数、类型、变量
//本包内可见(private)：小写字母开头
//包外可见(public)：大写字母开头
```

## 2. 不定参数

```go
//1、不定参数的类型
func myfunc(args ...int){   //...type不定参数的类型，必须是最后一个参数，本质是切片
  for _,arg:=range args{
    fmt.Println(arg)
  }
}
//函数调用,传参可以选择多个，个数不定
myfunc(1,2,3)
myfunc(1,2,3,4,5)

//2、不定参数的传递，假如有个变参函数myfunc2(args ...int)
func myfunc1(args ...int){
  //按原样传递
  myfunc2(args...)
  //传递切片
  myfunc2(args[1:]...)
}

//3、任意类型的不定参数，使用interface{}作为指定类型
func Printf(format string,args ...interface{}){   //此为标准库中fmt.Printf()函数的原型
  //函数体
}
```

## 3. 多返回值

```go
//多返回值
func (file *File) Read(b []byte) (n int,err error)
//使用下划线"_"来丢弃返回值
n,_:=f.Read(buf)
```

## 4. 匿名函数

匿名函数：不带函数名的函数，可以像变量一样被传递。

```go
func(a,b int,z float32) bool{  //没有函数名
  return a*b<int(z)
}
f:=func(x,y int) int{
  return x+y
}
```

## 5. 闭包

### 5.1. 闭包的概念

闭包是可以包含自由变量（未绑定到特定的对象）的代码块，这些变量不在代码块内或全局上下文中定义，而在定义代码块的环境中定义。要执行的代码块为自由变量提供绑定的计算环境（作用域）。

### 5.2. 闭包的价值

闭包的价值在于可以作为一个变量对象来进行传递和返回。即可以把函数本身看作是一个变量。

### 5.3. Go中的闭包

Go闭包是指引用了函数外的变量的一种函数，这样该函数就被绑定在某个变量上，只要闭包还被使用则引用的变量会一直存在。

Go的匿名函数是一个闭包，Go闭包常用在go和defer关键字中。

### 5.4. 闭包的坑

在for range中goroutine的方式使用闭包，如果没有给匿名函数传入一个变量，或新建一个变量存储迭代的变量，那么goroutine执行的结果会是最后一个迭代变量的结果，而不是每个迭代变量的结果。这是因为如果没有通过一个变量来拷贝迭代变量，那么闭包因为绑定了变量，当每个groutine运行时，迭代变量可能被更改。

示例如下：

```go
// false, print 3 3 3
values := []int{1,2,3}
for _, val := range values {
	go func() {
		fmt.Println(val)
	}()
}
// true, print 1 2 3
for _, val := range values {
	go func(val interface{}) {
		fmt.Println(val)
	}(val)
}
```

### 5.5 闭包的示例

> closure.go

```go
package main

import (
	"fmt"
)

func main() {
	var j int = 5
	a := func() func() {
		var i int = 10
		return func() {
			fmt.Printf("i, j: %d, %d\n", i, j)
		}
	}()
	a()
	j *= 2
	a()
}
```

### 5.6 闭包的参考链接

- https://tour.golang.org/moretypes/25
- https://golang.org/doc/faq#closures_and_goroutines
- https://github.com/golang/go/wiki/CommonMistakes

参考：
 
- 《Go语言编程》 
