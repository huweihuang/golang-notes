# Goroutine

```go
//定义调用体
func Add(x,y int){
  z:=x+y
  fmt.Println(z)
}
//go关键字执行调用，即会产生一个goroutine并发执行
//当函数返回时，goroutine自动结束，如果有返回值,返回值会自动被丢弃
go Add(1,1)
//并发执行
func main(){
  for i:=0;i<10;i++{//主函数启动了10个goroutine，然后返回，程序退出，并不会等待其他goroutine结束
    go Add(i,i)     //所以需要通过channel通信来保证其他goroutine可以顺利执行
  }
}
```
