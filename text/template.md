# 基本语法

go 统一使用了 `{{` 和 `}}` 作为左右标签，没有其他的标签符号。

使用 `.` 来访问当前位置的上下文

使用 `$` 来引用当前模板根级的上下文

使用 `$var` 来访问创建的变量

**模板中支持的 go 语言符号**

```go
{{"string"}} // 一般 string
{{`raw string`}} // 原始 string
{{'c'}} // byte
{{print nil}} // nil 也被支持
```

**模板中的 pipeline**

可以是上下文的变量输出，也可以是函数通过管道传递的返回值

```go
{{. | FuncA | FuncB | FuncC}}
```

当 pipeline 的值等于:

- false 或 0
- nil 的指针或 interface
- 长度为 0 的 array, slice, map, string

那么这个 pipeline 被认为是空

## 1. if … else … end

```go
{{if pipeline}}{{end}}
```

if 判断时，pipeline 为空时，相当于判断为 False

支持嵌套的循环

```go
{{if .IsHome}}
{{else}}
    {{if .IsAbout}}{{end}}
{{end}}
```

也可以使用 else if 进行

```go
{{if .IsHome}}
{{else if .IsAbout}}
{{else}}
{{end}}
```

## 2. range … end

```go
{{range pipeline}}{{.}}{{end}}
```

pipeline 支持的类型为 array, slice, map, channel

range 循环内部的 `.` 改变为以上类型的子元素

对应的值长度为 0 时，range 不会执行，`.` 不会改变。

```go
pages := []struct {
    Num int
}{{10}, {20}, {30}}
 
this.Data["Total"] = 100
this.Data["Pages"] = pages
```

使用 `.Num` 输出子元素的 Num 属性，使用 `$.` 引用模板中的根级上下文

```go
{{range .Pages}}
    {{.Num}} of {{$.Total}}
{{end}}
```

使用创建的变量，在这里和 go 中的 range 用法是相同的。

```go
{{range $index, $elem := .Pages}}
    {{$index}} - {{$elem.Num}} - {{.Num}} of {{$.Total}}
{{end}}
```

range 也支持 else

```go
{{range .Pages}}
{{else}}
    {{/* 当 .Pages 为空 或者 长度为 0 时会执行这里 */}}
{{end}}
```

## 3. with … end

```go
{{with pipeline}}{{end}}
```

with 用于重定向 pipeline

```go
{{with .Field.NestField.SubField}}
    {{.Var}}
{{end}}
```

也可以对变量赋值操作

```go
{{with $value := "My name is %s"}}
    {{printf . "slene"}}
{{end}}
```

with 也支持 else

```go
{{with pipeline}}
{{else}}
    {{/* 当 pipeline 为空时会执行这里 */}}
{{end}}
```

## 4. define

define 可以用来定义自模板，可用于模块定义和模板嵌套

```go
{{define "loop"}}
    <li>{{.Name}}</li>
{{end}}
```

使用 template 调用模板

```go
<ul>
    {{range .Items}}
        {{template "loop" .}}
    {{end}}
</ul>
```

## 5. template

```go
{{template "模板名" pipeline}}
```

将对应的上下文 pipeline 传给模板，才可以在模板中调用

**Beego 中支持直接载入文件模板**

```go
{{template "path/to/head.html" .}}
```

Beego 会依据你设置的模板路径读取 head.html

在模板中可以接着载入其他模板，对于模板的分模块处理很有用处

## 6. 注释

允许多行文本注释，不允许嵌套

 ```go
{{/* comment content
support new line */
 ```
