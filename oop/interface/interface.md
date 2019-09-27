# 1. 接口[多态]

​多态性（polymorphisn）是允许你将父对象设置成为和一个或更多的他的子对象相等的技术，赋值之后，父对象就可以根据当前赋值给它的子对象的特性以不同的方式运作。

 简而言之，就是允许将子类类型的指针赋值给父类类型的指针。

 即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。多态分为编译时多态（静态多态）和运行时多态（动态多态），编译时多态一般通过方法重载实现，运行时多态一般通过方法重写实现。

## 1.1 接口概念

>接口类型可以看作是类型系统中一种特殊的类型，而实例就是实现了该接口的具体结构体类型。
>
>接口类型与实现了该接口的结构体对象之间的关系好比变量类型与变量之间的关系。

​	**接口即一组方法定义的集合，定义了对象的一组行为**，由具体的类型实例实现具体的方法。换句话说，**一个接口就是定义（规范或约束），而方法就是实现**，接口的作用应该是**将定义与实现分离**，降低耦合度。习惯用“er”结尾来命名，例如“Reader”。**接口与对象的关系是多对多**，即一个对象可以实现多个接口，一个接口也可以被多个对象实现。

​	接口是Go语言整个类型系统的基石，其他语言的接口是不同组件之间的契约的存在，对契约的实现是强制性的，必须显式声明实现了该接口，这类接口称之为“侵入式接口”。而Go语言的接口是隐式存在，**只要实现了该接口的所有函数则代表已经实现了该接口，并不需要显式的接口声明**。

- **接口的比喻**

​     你的电脑上只有一个USB接口。这个USB接口可以接MP3，数码相机，摄像头，鼠标，键盘等。。。所有的上述硬件都可以公用这个接口，有很好的扩展性，**该USB接口定义了一种规范，只要实现了该规范，就可以将不同的设备接入电脑，而设备的改变并不会对电脑本身有什么影响（低耦合）**。

- **面向接口编程**

​      **接口表示调用者和设计者的一种约定**，在多人合作开发同一个项目时，事先定义好相互调用的接口可以大大提高开发的效率。接口是用类来实现的，实现接口的类必须严格按照接口的声明来实现接口提供的所有功能。有了接口，就可以在不影响现有接口声明的情况下，修改接口的内部实现，从而使兼容性问题最小化。

​	**面向接口编程可以分为三方面：制定者（或者叫协调者），实现者（或者叫生产者），调用者（或者叫消费者）**。 

​      当其他设计者调用了接口后，就不能再随意更改接口的定义，否则项目开发者事先的约定就失去了意义。但是可以在类中修改相应的代码，完成需要改动的内容。 

## 1.2 非侵入式接口

非侵入式接口：一个类只需要实现了接口要求的所有函数就表示实现了该接口，并不需要显式声明

```go
type File struct{
  //类的属性
}
//File类的方法
func (f *File) Read(buf []byte) (n int,err error)
func (f *File) Write(buf []byte) (n int,err error)
func (f *File) Seek(off int64,whence int) (pos int64,err error)
func (f *File) Close() error
//接口1：IFile
type IFile interface{
  Read(buf []byte) (n int,err error)
  Write(buf []byte) (n int,err error)
  Seek(off int64,whence int) (pos int64,err error)
  Close() error
}
//接口2：IReader
type IReader interface{
  Read(buf []byte) (n int,err error)
}
//接口赋值,File类实现了IFile和IReader接口，即接口所包含的所有方法
var file1 IFile = new(File)
var file2 IReader = new(File)
```

## 1.3 接口赋值

只要类实现了该接口的所有方法，即可将该类赋值给这个接口，接口主要用于多态化方法。即对接口定义的方法，不同的实现方式。

接口赋值：
**1）将对象实例赋值给接口**

```go
type IUSB interface{
    //定义IUSB的接口方法
}
//方法定义在类外，绑定该类，以下为方便，备注写在类中
type MP3 struct{
    //实现IUSB的接口，具体实现方式是MP3的方法
}
type Mouse struct{
    //实现IUSB的接口，具体实现方式是Mouse的方法
}
//接口赋值给具体的对象实例MP3
var usb IUSB =new(MP3)
usb.Connect()
usb.Close()
//接口赋值给具体的对象实例Mouse
var usb IUSB =new(Mouse)
usb.Connect()
usb.Close()
```

**2）将接口赋值给另一个接口**

1. 只要两个接口拥有相同的方法列表（与次序无关），即是两个相同的接口，可以相互赋值
2. 接口赋值只需要接口A的方法列表是接口B的子集（即假设接口A中定义的所有方法，都在接口B中有定义），那么B接口的实例可以赋值给A的对象。反之不成立，即子接口B包含了父接口A，因此可以将子接口的实例赋值给父接口。
3. 即子接口实例实现了子接口的所有方法，而父接口的方法列表是子接口的子集，则子接口实例自然实现了父接口的所有方法，因此可以将子接口实例赋值给父接口。

```go
type Writer interface{    //父接口
    Write(buf []byte) (n int,err error)
}
type ReadWriter interface{    //子接口
    Read(buf []byte) (n int,err error)
    Write(buf []byte) (n int,err error)
}
var file1 ReadWriter=new(File)   //子接口实例
var file2 Writer=file1           //子接口实例赋值给父接口
```

## 1.4 接口查询

若要在 switch 外判断一个接口类型是否实现了某个接口，可以使用“逗号 ok ”。

value, ok := Interfacevariable.(implementType)

其中 Interfacevariable 是接口变量（接口值），implementType 为实现此接口的类型，value 返回接口变量实际类型变量的值，如果该类型实现了此接口返回 true。

```go
//判断file1接口指向的对象实例是否是File类型
var file1 Writer=...
if file5,ok:=file1.(File);ok{  
  ...
}
```

## 1.5 接口类型查询

在 Go 中，要判断传递给接口值的变量类型，可以在使用 type switch 得到。(type)只能在 switch 中使用。

```go
// 另一个实现了 I 接口的 R 类型
type R struct { i int }
func (p *R) Get() int { return p.i }
func (p *R) Put(v int) { p.i = v }
 
func f(p I) {
    switch t := p.(type) { // 判断传递给 p 的实际类型
        case *S: // 指向 S 的指针类型
        case *R: // 指向 R 的指针类型
        case S:  // S 类型
        case R:  // R 类型
        default: //实现了 I 接口的其他类型
    }
}
```

## 1.6 接口组合

```go
//接口组合类似类型组合，只不过只包含方法，不包含成员变量
type ReadWriter interface{  //接口组合，避免代码重复
  Reader      //接口Reader
  Writer      //接口Writer
}
```

## 1.7 Any类型[空接口]

每种类型都能匹配到空接口：interface{}。空接口类型对方法没有任何约束（因为没有方法），它能包含任意类型，也可以实现到其他接口类型的转换。如果传递给该接口的类型变量实现了转换后的接口则可以正常运行，否则出现运行时错误。

```go
//interface{}即为可以指向任何对象的Any类型，类似Java中的Object类
var v1 interface{}=struct{X int}{1}
var v2 interface{}="abc"
 
func DoSomething(v interface{}) {   //该函数可以接收任何类型的参数，因为任何类型都实现了空接口
   // ...
}
```

## 1.8 接口的代码示例

```go
//接口animal
type Animal interface {
    Speak() string
}
//Dog类实现animal接口
type Dog struct {
}
 
func (d Dog) Speak() string {
    return "Woof!"
}
//Cat类实现animal接口
type Cat struct {
}
 
func (c Cat) Speak() string {
    return "Meow!"
}
//Llama实现animal接口
type Llama struct {
}
 
func (l Llama) Speak() string {
    return "?????"
}
//JavaProgrammer实现animal接口
type JavaProgrammer struct {
}
 
func (j JavaProgrammer) Speak() string {
    return "Design patterns!"
}
//主函数
func main() {
    animals := []Animal{Dog{}, Cat{}, Llama{}, JavaProgrammer{}}  //利用接口实现多态
    for _, animal := range animals {
        fmt.Println(animal.Speak())  //打印不同实现该接口的类的方法返回值
    }
}
```

