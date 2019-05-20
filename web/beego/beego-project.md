# beego项目逻辑

# 1. 路由设置

## 1.1. beego.Router

入口文件main.go

```go
package main
 
import (
    _ "quickstart/routers"
    "github.com/astaxie/beego"
)
 
func main() {
    beego.Run()
}
```

go中导入包中init函数的执行逻辑

![init](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578706/article/golang/beego/init.png)

`_ "quickstart/routers"`,包只引入执行了里面的init函数

```go
package routers
 
import (
    "quickstart/controllers"
    "github.com/astaxie/beego"
)
 
func init() {
    beego.Router("/", &controllers.MainController{})
}
```

路由包里执行了路由注册`beego.Router`, 这个函数的功能是映射URL到controller，第一个参数是URL(用户请求的地址)，这里是 `/`，也就是访问的不带任何参数的URL，第二个参数是对应的 Controller，就是把请求分发到那个控制器来执行相应的逻辑。

## 1.2. beego.Run

- 解析配置文件

  beego 会自动解析在 conf 目录下面的配置文件 `app.conf`，通过修改配置文件相关的属性，我们可以定义：开启的端口，是否开启 session，应用名称等信息。

- 执行用户的hookfunc

  beego会执行用户注册的hookfunc，默认的已经存在了注册mime，用户可以通过函数`AddAPPStartHook`注册自己的启动函数。

- 是否开启 session

  会根据上面配置文件的分析之后判断是否开启 session，如果开启的话就初始化全局的 session。

- 是否编译模板

  beego 会在启动的时候根据配置把 views 目录下的所有模板进行预编译，然后存在 map 里面，这样可以有效的提高模板运行的效率，无需进行多次编译。

- 是否开启文档功能

  根据EnableDocs配置判断是否开启内置的文档路由功能

- 是否启动管理模块

  beego 目前做了一个很酷的模块，应用内监控模块，会在 8088 端口做一个内部监听，我们可以通过这个端口查询到 QPS、CPU、内存、GC、goroutine、thread 等统计信息。

- 监听服务端口

  这是最后一步也就是我们看到的访问 8080 看到的网页端口，内部其实调用了 `ListenAndServe`，充分利用了 goroutine 的优势，一旦 run 起来之后，我们的服务就监听在两个端口了，一个服务端口 8080 作为对外服务，另一个 8088 端口实行对内监控。

# 2. controller 逻辑

```go
package controllers
 
import (
        "github.com/astaxie/beego"
)
 
type MainController struct {
        beego.Controller
}
 
func (this *MainController) Get() {
        this.Data["Website"] = "beego.me"
        this.Data["Email"] = "astaxie@gmail.com"
        this.TplName = "index.tpl"
}
```

1、声明了一个控制器 `MainController`，这个控制器里面内嵌了 `beego.Controller`，即Go 的嵌入方式，也就是 `MainController` 自动拥有了所有 `beego.Controller` 的方法。而 `beego.Controller` 拥有很多方法，其中包括 `Init`、`Prepare`、`Post`、`Get`、`Delete`、`Head`等方法。可以通过重写的方式来实现这些方法，以上例子重写了 `Get` 方法。

2、beego 是一个 RESTful 的框架，请求默认是执行对应 `req.Method` 的方法。例如浏览器的是 `GET` 请求，那么默认就会执行 `MainController` 下的 `Get` 方法。（用户可以改变这个行为，通过注册自定义的函数名）。

3、获取数据，赋值到 `this.Data` 中，这是一个用来存储输出数据的 map。

4、渲染模板，`this.TplName` 就是需要渲染的模板，这里指定了 `index.tpl`，如果用户不设置该参数，那么默认会去到模板目录的 `Controller/<方法名>.tpl` 查找，例如上面的方法会去 `maincontroller/get.tpl`(文件、文件夹必须小写)。用户设置了模板之后系统会自动的调用 `Render` 函数（这个函数是在 beego.Controller 中实现的），所以无需用户自己来调用渲染。

5、如果不使用模板可以直接输出：

```go
func (this *MainController) Get() {
        this.Ctx.WriteString("hello")
}
```

# 3. model逻辑

model一般用来处理数据库操作，如果逻辑中存在可以复用的部分就可以抽象成一个model。

```go
package models
 
import (
    "loggo/utils"
    "path/filepath"
    "strconv"
    "strings"
)
 
var (
    NotPV []string = []string{"css", "js", "class", "gif", "jpg", "jpeg", "png", "bmp", "ico", "rss", "xml", "swf"}
)
 
const big = 0xFFFFFF
 
func LogPV(urls string) bool {
    ext := filepath.Ext(urls)
    if ext == "" {
        return true
    }
    for _, v := range NotPV {
        if v == strings.ToLower(ext) {
            return false
        }
    }
    return true
}
```

# 4. view逻辑

`Controller中的this.TplName = "index.tpl"`，设置显示的模板文件，默认支持 `tpl` 和 `html` 的后缀名，如果想设置其他后缀你可以调用 `beego.AddTemplateExt` 接口设置。beego 采用了 Go 语言默认的模板引擎，和 Go 的模板语法一样。

```html
<!DOCTYPE html>
 
<html>
    <head>
        <title>Beego</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    </head>
     
    <body>
        <header class="hero-unit" style="background-color:#A9F16C">
            <div class="container">
            <div class="row">
              <div class="hero-text">
                <h1>Welcome to Beego!</h1>
                <p class="description">
                    Beego is a simple & powerful Go web framework which is inspired by tornado and sinatra.
                <br />
                    Official website: <a href="http://{{.Website}}">{{.Website}}</a>
                <br />
                    Contact me: {{.Email}}
                </p>
              </div>
            </div>
            </div>
        </header>
    </body>
</html>
```

Controller 里面把数据赋值给了 data（map 类型），然后在模板中就直接通过 key 访问 `.Website` 和 `.Email` 。这样就做到了数据的输出。

# 5. 静态文件

网页往往包含了很多的静态文件，包括图片、JS、CSS 等

```bash
├── static
    │   ├── css
    │   ├── img
    │   └── js
```

beego 默认注册了 static 目录为静态处理的目录，注册样式：URL 前缀和映射的目录（在/main.go文件中beego.Run()之前加入）：

```go
StaticDir["/static"] = "static"
```

用户可以设置多个静态文件处理目录，例如你有多个文件下载目录 download1、download2，你可以这样映射（在/main.go文件中beego.Run()之前加入）：

```go
beego.SetStaticPath("/down1", "download1") beego.SetStaticPath("/down2", "download2")
```

这样用户访问 URL `http://localhost:8080/down1/123.txt` 则会请求 download1 目录下的 123.txt 文件。


参考：

- https://beego.me/docs/quickstart/router.md
- https://beego.me/docs/quickstart/controller.md
- https://beego.me/docs/quickstart/model.md
- https://beego.me/docs/quickstart/view.md
- https://beego.me/docs/quickstart/static.md
