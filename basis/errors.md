# 错误处理

## 1. error接口
```go
//定义error接口
type error interface{
  Error() string
}
//调用error接口
func Foo(param int) (n int,err error){
  //...
}
n,err:=Foo(0)
if err!=nil{
  //错误处理
}else{
  //使用返回值
}
```

## 2. defer[延迟函数]

语法：

```go
defer function_name()
```

1）defer在声明时不会执行，而是推迟执行，在return执行前，倒序执行defer[先进后出]，一般用于释放资源，清理数据，记录日志，异常处理等。

2）defer有一个特性：即使函数抛出异常，defer仍会被执行，这样不会出现程序错误导致资源不被释放，或者因为第三方包的异常导致程序崩溃。

3）一般用于打开文件后释放资源的操作，比如打开一个文件，最后总是要关闭的。而在打开和关闭之间，会有诸多的处理，可能会有诸多的if-else、根据不同的情况需要提前返回

```go
f, = os.open(filename)
defer f.close()
do_something()
if (condition_a) {return}
do_something_again() 
if (condition_b) {return}
do_further_things()
```

4）defer示例

```go
package main
import "fmt"

func deferTest(number int) int {
	defer func() {
		number++
		fmt.Println("three:", number)
	}()

	defer func() {
		number++
		fmt.Println("two:", number)
	}()

	defer func() {
		number++
		fmt.Println("one:", number)
	}()

	return number
}

func main() {
	fmt.Println("函数返回值：", deferTest(0))
}
 
/*
one: 1
two: 2
three: 3
函数返回值： 0
*/
```

## 3. panic()和recover()

Go中使用内置函数`panic()`和`recover()`来处理程序中的错误。

```go
func panic(interface{})
func recover() interface{}
```

### 3.1. panic()

当函数执行触发了panic()函数时，如果没有使用到defer关键字，函数执行流程会被立即终止；如果使用了defer关键字，则会逐层执行defer语句，直到所有的函数被终止。

错误信息，包括panic函数传入的参数，将会被报告出来。

示例如下：

```go
panic(404)
panic("network broken")
panic(Error("file not exists"))
```

### 3.2. recover()

recover()函数用于终止错误处理流程。一般使用在defer关键字后以有效截取错误处理流程。如果没有在发生异常的goroutine中明确调用恢复过程（使用recover关键字） ，会导致该goroutine所属的进程打印异常信息后直接退出。

示例如下：

```go
defer func() {
	if r := recover(); r != nil {
		log.Printf("Runtime error caught: %v", r)
	}
}()
foo()
```

无论foo()中是否触发了错误处理流程，该匿名defer函数都将在函数退出时得到执行。假如foo()中触发了错误处理流程， recover()函数执行将使得该错误处理过程终止。如果错误处理流程被触发时，程序传给panic函数的参数不为nil，则该函数还会打印详细的错误信息。

参考：

- 《Go语言编程》
