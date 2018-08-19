## 1. 指针的概念

| 概念   | 说明                                |
| ---- | --------------------------------- |
| 变量   | 是一种占位符，用于引用计算机的内存地址。可理解为内存地址的标签   |
| 指针   | 表示内存地址，表示地址的指向。指针是一个指向另一个变量内存地址的值 |
| &    | 取地址符，例如：{指针}:=&{变量}               |
| *    | 取值符，例如：{变量}:=*{指针}                |

## 2. 内存地址说明

### 2.1. 内存定义
计算机的内存 RAM 可以把它想象成一些有序的盒子，一个接一个的排成一排，每一个盒子或者单元格都被一个唯一的数字标记依次递增，这个数字就是该单元格的地址，也就是内存的地址。
![什么是内存](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578751/article/golang/pointer/memory.png)

**硬件角度**：内存是CPU沟通的桥梁，程序运行在内存中。

**逻辑角度**：内存是一块具备随机访问能力，支持读写操作，用来存放程序及程序运行中产生的数据的区域。

| 概念   | 比喻                            |
| ---- | ----------------------------- |
| 内存   | 一层楼层                          |
| 内存块  | 楼层中的一个房间                      |
| 变量名  | 房间的标签，例如：总经理室                 |
| 指针   | 房间的具体地址（门牌号），例如：总经理室地址是2楼201室 |
| 变量值  | 房间里的具体存储物                     |
| 指针地址 | 指针的地址：存储指针内存块的地址              |

### 2.2. 内存单位和编址

#### 2.2.1. 内存单位

| 单位       | 说明                                |
| -------- | --------------------------------- |
| 位（bit）   | 计算机中最小的数据单位，每一位的状态只能是0或1          |
| 字节（Byte） | 1Byte=8bit，是内存基本的计量单位             |
| 字        | “字”由若干个字节构成，字的位数叫字长，不同档次的机器有不同的字长 |
| KB       | 1KB=1024Byte，即1024个字节             |
| MB       | 1MB=1024KB                        |
| GB       | 1GB=1024MB                        |

#### 2.2.2. 内存编址

计算机中的内存按字节编址，每个地址的存储单元可以存放一个字节的数据，CPU通过内存地址获取指令和数据，并不关心这个地址所代表的空间在什么位置，内存地址和地址指向的空间共同构成了一个内存单元。

#### 2.2.3. 内存地址

内存地址通常用16进制的数据表示，例如0x0ffc1。
## 3.变量与指针运算理解
编写一段程序，检索出值并存储在地址为 200 的一个块内存中，将其乘以 3，并将结果存储在地址为 201 的另一块内存中
### 3.1.本质
1. 检索出内存地址为 200 的值，并将其存储在 CPU 中
2. 将存储在 CPU 中的值乘以 3
3. 将 CPU 中存储的结果，写入地址为 201 的内存块中

![什么是变量](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578752/article/golang/pointer/var1.png)

### 3.2.基于变量的理解
1. 获取变量 a 中存储的值，并将其存储在 CPU 中
2. 将其乘以 3
3. 将结果保存在变量 b 中

![什么是变量2](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578752/article/golang/pointer/var2.png)

```
var a = 6 
var b = a * 3
```
### 3.3.基于指针的理解
```go
func main() {
    a := 200
    b := &a
    *b++
    fmt.Println(a)
}
```
以上函数对a进行+1操作，具体理解如下：

**1.a:=200**

![什么是指针](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578751/article/golang/pointer/pointer1.png)

**2.  b := &a**

![什么是指针2](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578751/article/golang/pointer/pointer2.png)

**3. *b++**

![什么是指针3](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578752/article/golang/pointer/pointer3.png)

![什么是指针4](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578752/article/golang/pointer/pointer4.png)

## 4. 指针的使用

### 4.1. 方法中的指针

方法即为有接受者的函数，接受者可以是类型的实例变量或者是类型的实例指针变量。但两种效果不同。

**1、类型的实例变量**

```go
func main(){
    person := Person{"vanyar", 21}
    fmt.Printf("person<%s:%d>\n", person.name, person.age)
    person.sayHi()
    person.ModifyAge(210)
    person.sayHi()
}
type Person struct {
    name string
    age int
}
func (p Person) sayHi() {
    fmt.Printf("SayHi -- This is %s, my age is %d\n",p.name, p.age)
}
func (p Person) ModifyAge(age int) {
    fmt.Printf("ModifyAge")
    p.age = age
}
 
 
//输出结果
person<vanyar:21>
SayHi -- This is vanyar, my age is 21
ModifyAgeSayHi -- This is vanyar, my age is 21
```

尽管 ModifyAge 方法修改了其age字段，可是方法里的p是person变量的一个副本，修改的只是副本的值。下一次调用sayHi方法的时候，还是person的副本，因此修改方法并不会生效。

即实例变量的方式并不会改变接受者本身的值。

**2、类型的实例指针变量**

```go
func (p *Person) ChangeAge(age int)  {
    fmt.Printf("ModifyAge")
    p.age = age
}
```

Go会根据Person的示例类型，转换成指针类型再拷贝，即 person.ChangeAge会变成 (&person).ChangeAge。

指针类型的接受者，如果实例对象是值，那么go会转换成指针，然后再拷贝，如果本身就是指针对象，那么就直接拷贝指针实例。因为指针都指向一处值，就能修改对象了。

## 5. 零值与nil(空指针)

变量声明而没有赋值，默认为零值，不同类型零值不同，例如字符串零值为空字符串；

指针声明而没有赋值，默认为nil，即该指针没有任何指向。当指针没有指向的时候，不能对(*point)进行操作包括读取，否则会报空指针异常。

```go
func main(){
    // 声明一个指针变量 aPot 其类型也是 string
    var aPot *string
    fmt.Printf("aPot: %p %#v\n", &aPot, aPot) // 输出 aPot: 0xc42000c030 (*string)(nil)
    *aPot = "This is a Pointer"  // 报错： panic: runtime error: invalid memory address or nil pointer dereference
}
```

解决方法即给该指针分配一个指向,即初始化一个内存，并把该内存地址赋予指针变量，例如：

```go
// 声明一个指针变量 aPot 其类型也是 string
    var aPot *string
    fmt.Printf("aPot: %p %#v\n", &aPot, aPot) // 输出 aPot: 0xc42000c030 (*string)(nil)
 
    aPot = &aVar
    *aPot = "This is a Pointer"
    fmt.Printf("aVar: %p %#v \n", &aVar, aVar) // 输出 aVar: 0xc42000e240 "This is a Pointer"
    fmt.Printf("aPot: %p %#v %#v \n", &aPot, aPot, *aPot) // 输出 aPot: 0xc42000c030 (*string)(0xc42000e240) "This is a Pointer"
```

或者通过new开辟一个内存，并返回这个内存的地址。

```go
var aNewPot *int
 
aNewPot = new(int)
*aNewPot = 217
fmt.Printf("aNewPot: %p %#v %#v \n", &aNewPot, aNewPot, *aNewPot) // 输出 aNewPot: 0xc42007a028 (*int)(0xc42006e1f0) 217
```

## 6. 总结

- Golang提供了指针用于操作数据内存，并通过引用来修改变量。
- 只声明未赋值的变量，golang都会自动为其初始化为零值，基础数据类型的零值比较简单，引用类型和指针的零值都为nil，nil类型不能直接赋值，因此需要通过new开辟一个内存，或者通过make初始化数据类型，或者两者配合，然后才能赋值。
- 指针也是一种类型，不同于一般类型，指针的值是地址，这个地址指向其他的内存，通过指针可以读取其所指向的地址所存储的值。
- 函数方法的接受者，也可以是指针变量。无论普通接受者还是指针接受者都会被拷贝传入方法中，不同在于拷贝的指针，其指向的地方都一样，只是其自身的地址不一样。



参考：

- [http://www.jianshu.com/p/d23f78a3922b](http://www.jianshu.com/p/d23f78a3922b)
  
- [http://www.jianshu.com/p/44b9429d7bef](http://www.jianshu.com/p/44b9429d7bef)
