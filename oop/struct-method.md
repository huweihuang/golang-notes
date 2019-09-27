# 1. 类型系统[类的声明]

类型系统：

- 一组基本类型构成的“基本类型集合”；
- “基本类型集合”上定义的一系列组合、运算、转换方法。

类型系统包括基础类型（byte、int、bool、float等）；复合类型（数组、结构体、指针等）；可以指向任何对象的类型（Any类型，类似Java的Object类型）；值语义和引用语义；面向对象类型；接口。Go大多数类型为值语义，可以给任何类型添加方法（包括内置类型，不包括指针类型）。Any类型是空接口即interface{}。

# 2. 结构体

**结构体[类属性的声明]**

struct的功能类似Java的class，可实现嵌套组合(类似继承的功能)
struct实际上就是一种复合类型，只是对类中的属性进行定义赋值，并没有对方法进行定义，方法可以随时定义绑定到该类的对象上，更具灵活性。可利用嵌套组合来实现类似继承的功能避免代码重复。

```go
type Rect struct{   //定义矩形类
  x,y float64       //类型只包含属性，并没有方法
  width,height float64
}
func (r *Rect) Area() float64{    //为Rect类型绑定Area的方法，*Rect为指针引用可以修改传入参数的值
  return r.width*r.height         //方法归属于类型，不归属于具体的对象，声明该类型的对象即可调用该类型的方法
}
```

# 3. 方法

1、为类型添加方法[类方法声明]，方法即为有接收者的函数
func (对象名 对象类型) 函数名(参数列表) (返回值列表)
可随时为某个对象添加方法即为某个方法添加归属对象（receiver），以方法为中心
在Go语言中没有隐藏的this指针，即显示传递，形参即为this，例如以下的形参为a。

```go
type Integer int
func (a Integer) Less(b Integer) bool{  //表示a这个对象定义了Less这个方法，a可以为任意类型
  return a<b                           
}
//类型基于值传递，如果要修改值需要传递指针
func (a *Integer) Add(b Integer){
  *a+=b    //通过指针传递来改变值
}
```

# 4. 值语义和引用语义

值类型：b的修改并不会影响a的值

引用类型：b的修改会影响a的值

Go大多类型为值语义，包括基本类型：byte，int，string等；复合类型：数组，结构体(struct)，指针等

```go
//2、值语义和引用语义
b=a
b.Modify()
 
//值类型
var a=[3]int{1,2,3}
b:=a
b[1]++
fmt.Println(a,b)   //a=[1,2,3]  b=[1,3,3]
//引用类型
a:=[3]int{1,2,3}
b:=&a              //b指向a,即为a的地址，对b指向的值改变实际上就是对a的改变（数组本身就是一种地址指向）
b[1]++
fmt.Println(a,*b)  //a=[1,3,3]  b=[1,3,3]   //*b,取地址指向的值
```

# 5. 初始化[实例化对象]

 数据初始化的内建函数new()与make()，二者都是用来分配空间。区别如下:

## 5.1. new()

1. func new(Type) *Type
2. 内置函数 `new` 分配空间。传递给`new` 函数的是一个**类型**，不是一个值。返回值是指向这个新分配的零值的**指针**

## 5.2. make()

1. func make(Type, size IntegerType) Type 
2. 内建函数 `make` 分配并且初始化 一个 slice, 或者 map 或者 chan 对象。 并且只能是这三种对象。 和 `new` 一样，第一个参数是 类型，不是一个值。 但是`make` 的返回值就是这个类型（即使一个引用类型），而不是指针。 具体的返回值，依赖具体传入的类型。

## 5.3. 示例

```go
//创建实例
rect1:=new(Rect)   //new一个对象
rect2:=&Rect{}     //为赋值默认值，bool默认值为false，int默认为零值0，string默认为空字符串
rect3:=&Rect{0,0,100,200}     //取地址并赋值,按声明的变量顺序依次赋值
rect4:=&Rect{width:100,height:200}    //按变量名赋值不按顺序赋值
//构造函数：没有构造参数的概念，通常由全局的创建函数NewXXX来实现构造函数的功能
func NewRect(x,y,width,height float64) *Rect{
  return &Rect{x,y,width,height}     //利用指针来改变传入参数的值达到类似构造参数的效果
}
//方法的重载,Go不支持方法的重载（函数同名，参数不同）
//v …interface{}表示参数不定的意思，其中v是slice类型，及声明不定参数，可以传入任意参数，实现类似方法的重载
func (poem *Poem) recite(v ...interface{}) {
    fmt.Println(v)
}
```

# 6. 匿名组合[继承]

​       组合，即方法代理，例如A包含B，即A通过消息传递的形式代理了B的方法，而不需要重复写B的方法。

​       继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。继承主要为了代码复用，继承也可以扩展已存在的代码模块（类）。

​       严格来讲，继承是“a kind of ”，即子类是父类的一种，例如student是person的一种；组合是“a part of”，即父类是子类中的一部分，例如眼睛是头部的一部分。

```go
//1、匿名组合的方式实现了类似Java继承的功能，可以实现多继承
type Base struct{
  Name string
}
func (base *Base) Foo(){...}    //Base的Foo()方法
func (base *Base) Bar(){...}    //Base的Bar()方法
type Foo struct{  
  Base                         //通过组合的方式声明了基类，即继承了基类
  ...
}
func (foo *Foo) Bar(){
  foo.Base.Bar()               //并改写了基类的方法，该方法实现时先调用基类的Bar()方法
  ...                          //如果没有改写即为继承，调用foo.Foo()和调用foo.Base.Foo()的作用的一样的
}
//修改内存布局
type Foo struct{
  ...   //其他成员信息
  Base
}
//以指针方式组合
type Foo struct{
  *Base   //以指针方式派生，创建Foo实例时，需要外部提供一个Base类实例的指针
  ...
}
//名字冲突问题,组合内外如果出现名字重复问题，只会访问到最外层，内层会被隐藏，不会报错，即类似java中方法覆盖/重写。
type X struct{
  Name string
}
type Y struct{
  X             //Y.X.Name会被隐藏，内层会被隐藏
  Name string   //只会访问到Y.Name，只会调用外层属性
}
```

# 7. 可见性[封装]

封装，也就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。

**封装的本质或目的其实程序对信息(数据)的控制力。封装分为两部分：该隐藏的隐藏，该暴露的暴露。封装可以隐藏实现细节，使得代码模块化。**

Go中用大写字母开头来表示public，可以包外访问；小写字母开头来表示private，只能包内访问；访问性是包级别非类型级别,如果可访问性是类型一致的，可以加friend关键字表示朋友关系可互相访问彼此的私有成员(属性和方法)

```go
type Rect struct{
  X,Y float64
  Width,Height float64           //字母大写开头表示该属性可以由包外访问到
}
func (r *Rect) area() float64{   //字母小写开头表示该方法只能包内调用
  return r.Width*r.Height
}
```
