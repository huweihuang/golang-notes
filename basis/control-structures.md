## 流程控制

### 1. 条件语句

```go
//在if之后条件语句之前可以添加变量初始化语句，用;号隔离
if <条件语句> {    //条件语句不需要用括号括起来，花括号必须存在
  //语句体
}else{
  //语句体
}

//在有返回值的函数中，不允许将最后的return语句放在if...else...的结构中，否则会编译失败
//例如以下为错误范例
func example(x int) int{
  if x==0{
    return 5
  }else{
    return x  //最后的return语句放在if-else结构中，所以编译失败
  }
}
```

### 2. 选择语句

```go
//1、根据条件不同，对应不同的执行体
switch i{
  case 0:
  	fmt.Printf("0")
  case 1:                //满足条件就会退出，只有添加fallthrough才会继续执行下一个case语句
  	fmt.Prinntf("1")
  case 2,3,1:            //单个case可以出现多个选项
  	fmt.Printf("2,3,1")
  default:               //当都不满足以上条件时，执行default语句
  	fmt.Printf("Default")
}

//2、该模式等价于多个if-else的功能
switch {
  case <条件表达式1>:
  	语句体1
  case <条件表达式2>:
  	语句体2
}
```

### 3. 循环语句

```go
//1、Go只支持for关键字，不支持while，do-while结构
for i,j:=0,1;i<10;i++{    //支持多个赋值
  //语句体
}

//2、无限循环
sum:=1
for{  //不接条件表达式表示无限循环
  sum++
  if sum > 100{
    break   //满足条件跳出循环
  }
}

//3、支持continue和break，break可以指定中断哪个循环，break JLoop(标签)
for j:=0;j<5;j++{
  for i:=0;i<10;i++{
    if i>5{
      break JLoop   //终止JLoop标签处的外层循环
  }
  fmt.Println(i)
}
JLoop:    //标签处
...
```

### 4. 跳转语句

```go
//关键字goto支持跳转
func myfunc(){
  i:=0
  HERE:           //定义标签处
  fmt.Println(i)
  i++
  if i<10{
    goto HERE     //跳转到标签处
  }
}
```
