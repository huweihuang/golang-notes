## 错误处理
### 1、error接口
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
### 2、defer[延迟函数]
语法：**defer function_name()** 
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
