# Channel

channel就像管道的形式，是goroutine之间的通信方式，是进程内的通信方式，跨进程通信建议用分布式系统的方法来解决，例如Socket或http等通信协议。channel是类型相关，即一个channel只能传递一种类型的值，在声明时指定。

# 1. 基本语法

## 1.1. channel的声明

```go
//1、channel声明，声明一个管道chanName，该管道可以传递的类型是ElementType
//管道是一种复合类型，[chan ElementType],表示可以传递ElementType类型的管道[类似定语从句的修饰方法]
var chanName chan ElementType
var ch chan int                  //声明一个可以传递int类型的管道
var m map[string] chan bool      //声明一个map，值的类型为可以传递bool类型的管道
```

## 1.2. 初始化

```go
//2、初始化
ch:=make(chan int)   //make一般用来声明一个复合类型，参数为复合类型的属性
```

## 1.3. 管道读写

```go
//3、管道写入,把值想象成一个球，"<-"的方向，表示球的流向，ch即为管道
//写入时，当管道已满（管道有缓冲长度）则会导致程序堵塞，直到有goroutine从中读取出值
ch <- value
//管道读取，"<-"表示从管道把球倒出来赋值给一个变量
//当管道为空，读取数据会导致程序阻塞，直到有goroutine写入值
value:= <-ch 
```

## 1.4. select

```go
//4、每个case必须是一个IO操作，面向channel的操作，只执行其中的一个case操作，一旦满足则结束select过程
//面向channel的操作无非三种情况：成功读出；成功写入；即没有读出也没有写入
select{
  case <-chan1:
  //如果chan1读到数据，则进行该case处理语句
  case chan2<-1:
  //如果成功向chan2写入数据，则进入该case处理语句
  default:
  //如果上面都没有成功，则进入default处理流程
}
```

# 2. 缓冲和超时机制

## 2.1. 缓冲机制

```go
//1、缓冲机制：为管道指定空间长度，达到类似消息队列的效果
c:=make(chan int,1024)  //第二个参数为缓冲区大小，与切片的空间大小类似
//通过range关键字来实现依次读取管道的数据，与数组或切片的range使用方法类似
for i :=range c{
  fmt.Println("Received:",i)
}
```

## 2.2. 超时机制

```go
//2、超时机制：利用select只要一个case满足，程序就继续执行而不考虑其他case的情况的特性实现超时机制
timeout:=make(chan bool,1)    //设置一个超时管道
go func(){
  time.Sleep(1e9)      //设置超时时间，等待一分钟
  timeout<-true        //一分钟后往管道放一个true的值
}()
//
select {
  case <-ch:           //如果读到数据，则会结束select过程
  //从ch中读取数据
  case <-timeout:      //如果前面的case没有调用到，必定会读到true值，结束select，避免永久等待
  //一直没有从ch中读取到数据，但从timeout中读取到了数据
}
```

## 2.3. channel的传递

```go
//1、channel的传递，来实现Linux系统中管道的功能，以插件的方式增加数据处理的流程
type PipeData struct{
  value int
  handler func(int) int   //handler是属性？
  next chan int   //可以把[chan int]看成一个整体，表示放int类型的管道
}
func handler(queue chan *PipeData){ //queue是一个存放*PipeDate类型的管道，可改变管道里的数据块内容
  for data:=range queue{     //data的类型就是管道存放定义的类型，即PipeData
    data.next <- data.handler(data.value)    //该方法实现将PipeData的value值存放到next的管道中
  }
}
```

## 2.4. 单向channel

```go
//2、单向channel：只能用于接收或发送数据，是对channel的一种使用限制
//单向channel的声明
var ch1 chan int    //正常channel，可读写
var ch2 chan<- int  //单向只写channel  [chan<- int]看成一个整体，表示流入管道
var ch3 <-chan int  //单向只读channel  [<-chan int]看成一个整体，表示流出管道
//管道类型强制转换
ch4:=make(chan int)     //ch4为双向管道
ch5:=<-chan int(ch4)    //把[<-chan int]看成单向只读管道类型，对ch4进行强制类型转换
ch6:=chan<- int(ch4)    //把[chan<- int]看成单向只写管道类型，对ch4进行强制类型转换
func Parse(ch <-chan int){    //最小权限原则
  for value:=range ch{
    fmt.Println("Parsing value",value)
  }
}
```

## 2.5. 关闭channel

```go
//3、关闭channel，使用内置函数close()函数即可
close(ch)
//判断channel是否关闭
x,ok:=<-ch //ok==false表示channel已经关闭
if !ok {   //如果channel关闭，ok==false，!ok==true
  //执行体
}
```
