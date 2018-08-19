## 1.变量

### 1.1变量声明

```go
//1、单变量声明,类型放在变量名之后，可以为任意类型
var 变量名 类型
var v1,v2,v3 string //多变量同类型声明
//2、多变量声明
var {
    v1 int
    v2 []int
}
```

### 1.2变量初始化

```go
//1、使用关键字var，声明变量类型并赋值
var v1 int=10
//2、使用关键字var，直接对变量赋值，go可以自动推导出变量类型
var v2=10
//3、直接使用“：=”对变量赋值，不使用var，两者同时使用会语法冲突，推荐使用
v3:=10
```

### 1.3变量赋值

```go
//1、声明后再变量赋值
var v int
v=10
//2、多重赋值，经常使用在函数的多返回值中，err,v=func(arg)
i，j=j,i  //两者互换，并不需要引入中间变量
```

### 1.4匿名变量

```go
//Go中所有声明后的变量都需要调用到，当出现函数多返回值，并且部分返回值不需要使用时，可以使用匿名变量丢弃该返回值
func GetName()(firstName,lastName,nickName string){
  return "May","Chan","Make"
}
_,_,nickName:=GetName()  //使用匿名变量丢弃部分返回值
```

## 2.常量

​	Go语言中，常量是编译时期就已知且不可变的值，常量可以是数值类型（整型、浮点型、复数类型）、布尔类型、字符串类型。

### 2.1字面常量

```go
//字面常量(literal)指程序中硬编码的常量
3.14
“foo”
true
```

### 2.2常量定义

```go
//1、可以限定常量类型，但非必需
const Pi float64 = 3.14
//2、无类型常量和字面常量一样
const zero=0.0
//3、多常量赋值
const(
  size int64=1024
  eof=-1
)
//4、常量的多重赋值，类似变量的多重赋值
const u,v float32=0,3
const a,b,c=3,4,"foo"    //无类型常量的多重赋值
//5、常量赋值是编译期行为，可以赋值为一个编译期运算的常量表达式
const mask=1<<3
```

### 2.3预定义常量

```go
//预定义常量：true、false、iota
//iota：可修改常量，在每次const出现时被重置为0，在下一个const出现前，每出现一次iota，其代表的值自动增1。
const(          //iota重置为0
  c0=iota       //c0==0
  c1=iota       //c1==1
  c2=iota       //c2==2
)
//两个const赋值语句一样可以省略后一个
const(          //iota重置为0
  c0=iota       //c0==0
  c1            //c1==1
  c2            //c2==2
)
```

### 2.4枚举

枚举指一系列相关常量。

```go
const(
  Sunday=iota    //Sunday==0,以此类推
  Monday
  Tuesday
  Wednesday
  Thursday
  Friday
  Saturday       //大写字母开头表示包外可见
  numberOfDays   //小写字母开头表示包内私有
)
```
