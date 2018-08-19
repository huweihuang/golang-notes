## 函数

### 1. 函数定义与调用

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

### 2. 不定参数

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

### 3. 多返回值

```go
//多返回值
func (file *File) Read(b []byte) (n int,err error)
//使用下划线"_"来丢弃返回值
n,_:=f.Read(buf)
```

### 4. 匿名函数与闭包

```go
//1、匿名函数：不带函数名的函数，可以像变量一样被传递
func(a,b int,z float32) bool{  //没有函数名
  return a*b<int(z)
}
f:=func(x,y int) int{
  return x+y
}

//2、闭包
```
