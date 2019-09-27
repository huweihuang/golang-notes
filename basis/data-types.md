# 类型

# 1. 基础类型

## 1.1. 布尔类型

```go
//布尔类型的关键字为bool,值为true或false，不可写为0或1
var v1 bool
v1=true
//接受表达式判断赋值，不支持自动或强制类型转换
v2:=(1==2)
```

## 1.2. 整型

 ![这里写图片描述](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578665/article/golang/basics/int.png)

```go
//1、类型表示
//int和int32为不同类型，不会自动类型转换需要强制类型转换
//强制类型转换需注意精度损失（浮点数→整数），值溢出（大范围→小范围）
var v2 int32
v1:=64
v2=int32(v1)

//2、数值运算,支持“+,-,*,/和%”
5%3 //求余

//3、比较运算,“<,>,==,>=,<=,!=”
//不同类型不能进行比较例如int和int8，但可以与字面常量（literal）进行比较
var i int32
var j int64
i,j=1,2
if i==j  //编译错误，不同类型不能进行比较
if i==1 || j==2  //编译通过，可以与字面常量（literal）进行比较

//4、位运算
//Go(^x)取反与C语言(~x)不同，其他类似，具体见下表
```

![这里写图片描述](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578663/article/golang/basics/bit-operation.png)

## 1.3. 浮点型

```go
//1、浮点型分为float32(类似C中的float)，float64(类似C中的double)
var f1 float32
f1=12     //不加小数点，被推导为整型
f2:=12.0  //加小数点，被推导为float64
f1=float32(f2)  //需要执行强制转换
//2、浮点数的比较
//浮点数不是精确的表达方式，不能直接使用“==”来判断是否相等，可以借用math的包math.Fdim 
```

## 1.4. 复数类型

```go
//1、复数的表示
var v1 complex64
v1=3.2+12i
//v1 v2 v3 表示为同一个数
v2:=3.2+12i
v3:=complex(3.2,12)
//2、实部与虚部
//z=complex(x,y),通过内置函数实部x=real(z),虚部y=imag(z)
```

## 1.5. 字符串

```go
//声明与赋值
var str string
str="hello world"
```

![这里写图片描述](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578664/article/golang/basics/string.png)

## 1.6. 字符类型

```go
//1、byte，即uint8的别名
//2、rune，即Unicode
```

## 1.7. 错误类型（error）

# 2. 复合类型

## 2.1. 数组(array)

数组表示同一类型数据，数组长度定义后就不可更改，长度是数组内的一个内置常量，可通过len()来获取。

```go
//1、创建数组
var array1 [5]int    //声明：var 变量名 类型
var array2 [5]int=[5]int{1,2,3,4,5}   //初始化
array3：=[5]int{1,2,3,4,5}    //直接用“：=”赋值
[3][5]int  //二维数组
[3]*float  //指针数组

//2、元素访问
for i,v:=range array{
  //第一个返回值为数组下标，第二个为元素的值
}

//3、值类型
//数组在Go中作为一个值类型，值类型在赋值和函数参数传递时，只复制副本，因此在函数体中并不能改变数组的内容，需用指针来改变数组的值。
```

## 2.2. 切片(slice)

​	数组在定义了长度后无法改变，且作为值类型在传递时产生副本，并不能改变数组元素的值。因此切片的功能弥补了这个不足，切片类似指向数组的一个指针。可以抽象为三个变量：指向数组的指针；切片中元素的个数(len函数)；已分配的存储空间(cap函数)。

```go
//1、创建切片
//a)基于数组创建
var myArray [5]int=[5]{1,2,3,4,5}
var mySlice []int=myArray[first:last]
slice1=myArray[:]   //基于数组所有元素创建
slice2=myArray[:3]  //基于前三个元素创建
slice3=myArray[3:]  //基于第3个元素开始后的所有元素创建
//b)直接创建
slice1:=make([]int,5)       //元素初始值为0，初始个数为5
slice2:=make([]int,5,10)    //元素初始值为0，初始个数为5，预留个数为10
slice3:=[]int{1,2,3,4,5}    //初始化赋值
//c)基于切片创建
oldSlice:=[]int{1,2,3,4,5}
newSlice:=oldSlice[:3]   //基于切片创建，不能超过原切片的存储空间(cap函数的值)

//2、元素遍历
for i,v:=range slice{
  //与数组的方式一致，使用range来遍历
  //第一个返回值(i)为索引，第二个为元素的值(v)
}

//3、动态增减元素
//切片分存储空间(cap)和元素个数(len)，当存储空间小于实际的元素个数，会重新分配一块原空间2倍的内存块，并将原数据复制到该内存块中，合理的分配存储空间可以以空间换时间，降低系统开销。
//添加元素
newSlice:=append(oldSlice,1,2,3)   //直接将元素加进去，若存储空间不够会按上述方式扩容。
newSlice1:=append(oldSlice1,oldSlice2...)  //将oldSlice2的元素打散后加到oldSlice1中，三个点不可省略。

//4、内容复制
//copy()函数可以复制切片，如果切片大小不一样，按较小的切片元素个数进行复制
slice1:=[]int{1,2,3,4,5}
slice2:=[]int{6,7,8}
copy(slice2,slice1)   //只会复制slice1的前三个元素到slice2中
copy(slice1,slice1)   //只会复制slice2的三个元素到slice1中的前三个位置
```

## 2.3. 键值对(map)

map是一堆键值对的未排序集合。

```go
//1、先声明后创建再赋值
var map1 map[键类型] 值类型
//创建
map1=make(map[键类型] 值类型)
map1=make(map[键类型] 值类型 存储空间)
//赋值
map1[key]=value

// 直接创建
m2 := make(map[string]string)
// 然后赋值
m2["a"] = "aa"
m2["b"] = "bb"

// 初始化 + 赋值一体化
m3 := map[string]string{
	"a": "aa",
	"b": "bb",
}

//2、元素删除
//delete()函数删除对应key的键值对，如果key不存在，不会报错；如果value为nil，则会抛出异常(panic)。
delete(map1,key)  

//3、元素查找
value,ok:=myMap[key]
if ok{//如果找到
  //处理找到的value值
}

//遍历
for key,value:=range myMap{
	//处理key或value
}
```
map可以用来判断一个值是否在切片或数组中。

```go
// 判断某个类型（假如为myType）的值是否在切片或数组（假如为myList）中
// 构造一个map,key的类型为myType,value为bool型
myMap := make(map[myType]bool)
myList := []myType{value1, value2}
// 将切片中的值存为map中的key（因为key不能重复）,map的value都为true
for _, value := range myList {
	myMap[value] = true
}
// 判断valueX是否在myList中，即判断其是否在myMap的key中
if _, ok := myMap[valueX]; ok {
	// 如果valueX 在myList中，执行后续操作
}
```

## 2.4. 指针(pointer)
具体参考[Go语言指针详解](https://www.huweihuang.com/golang-notes/oop/pointer.html)

## 2.5. 结构体(struct)
具体参考[Go面向对象编程之结构体](https://www.huweihuang.com/golang-notes/oop/method.html)

## 2.6. 接口(interface)
具体参考[Go面向对象编程之接口](https://www.huweihuang.com/golang-notes/oop/interface/interface.html)

## 2.7. 通道(chan)
具体参考[Go并发编程之channel](https://www.huweihuang.com/golang-notes/concurrency/channel.html)
