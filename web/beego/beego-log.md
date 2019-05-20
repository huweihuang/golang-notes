# 1. 使用入门

beego 的日志处理是基于 logs 模块搭建的，内置了一个变量 `BeeLogger`，默认已经是 `logs.BeeLogger` 类型，初始化了 console，也就是默认输出到 `console。`

```go
beego.Emergency("this is emergency")
beego.Alert("this is alert")
beego.Critical("this is critical")
beego.Error("this is error")
beego.Warning("this is warning")
beego.Notice("this is notice")
beego.Informational("this is informational")
beego.Debug("this is debug")
```

# 2. 设置输出

我们的程序往往期望把信息输出到 log 中，现在设置输出到文件很方便，如下所示：

```go
beego.SetLogger("file", `{"filename":"logs/test.log"}`)
```

这个默认情况就会同时输出到两个地方，一个 console，一个 file，如果只想输出到文件，就需要调用删除操作：

```go
beego.BeeLogger.DelLogger("console")
```

# 3. 设置级别

```go
LevelEmergency
LevelAlert
LevelCritical
LevelError
LevelWarning
LevelNotice
LevelInformational
LevelDebug
```

级别依次降低，默认全部打印，但是一般我们在部署环境，可以通过设置级别设置日志级别：

```go
beego.SetLevel(beego.LevelInformational)
```

# 4. 输出文件名和行号

日志默认不输出调用的文件名和文件行号,如果你期望输出调用的文件名和文件行号,可以如下设置

```go
beego.SetLogFuncCall(true)
```

开启传入参数 true, 关闭传入参数 false, 默认是关闭的。

# 5. beego/logs模块的使用

是一个用来处理日志的库，目前支持的引擎有 file、console、net、smtp，可以通过如下方式进行安装：

```go
go get github.com/astaxie/beego/logs
```

## 5.1. 通用方式

首先引入包：

```go
import ( "github.com/astaxie/beego/logs" )
```

然后添加输出引擎（log 支持同时输出到多个引擎），这里我们以 console 为例，第一个参数是引擎名（包括：console、file、conn、smtp、es、multifile）

```go
logs.SetLogger("console")
```

添加输出引擎也支持第二个参数,用来表示配置信息，详细的配置请看下面介绍：

```go
logs.SetLogger(logs.AdapterFile,`{"filename":"project.log","level":7,"maxlines":0,"maxsize":0,"daily":true,"maxdays":10}`)
```

示例：

```go
package main
 
import (
    "github.com/astaxie/beego/logs"
)
 
func main() {
    //an official log.Logger
    l := logs.GetLogger()
    l.Println("this is a message of http")
    //an official log.Logger with prefix ORM
    logs.GetLogger("ORM").Println("this is a message of orm")
 
    logs.Debug("my book is bought in the year of ", 2016)
    logs.Info("this %s cat is %v years old", "yellow", 3)
    logs.Warn("json is a type of kv like", map[string]int{"key": 2016})
    logs.Error(1024, "is a very", "good game")
    logs.Critical("oh,crash")
}
```

## 5.2. 输出文件名和行号

日志默认不输出调用的文件名和文件行号,如果你期望输出调用的文件名和文件行号,可以如下设置

```go
logs.EnableFuncCallDepth(true)
```

开启传入参数 true,关闭传入参数 false,默认是关闭的.

如果你的应用自己封装了调用 log 包,那么需要设置 SetLogFuncCallDepth,默认是 2,也就是直接调用的层级,如果你封装了多层,那么需要根据自己的需求进行调整.

```go
logs.SetLogFuncCallDepth(3)
```

## 5.3. 异步输出日志

为了提升性能, 可以设置异步输出:

```go
logs.Async()
```

异步输出允许设置缓冲 chan 的大小

```go
logs.Async(1e3)
```

## 5.4. 引擎配置

- console

  可以设置输出的级别，或者不设置保持默认，默认输出到 `os.Stdout`：

  ```go
  logs.SetLogger(logs.AdapterConsole, `{"level":1}`)
  ```

- file

  设置的例子如下所示：

  ```go
  logs.SetLogger(logs.AdapterFile, `{"filename":"test.log"}`)
  ```

  主要的参数如下说明：

  - filename 保存的文件名
  - maxlines 每个文件保存的最大行数，默认值 1000000
  - maxsize 每个文件保存的最大尺寸，默认值是 1 << 28, //256 MB
  - daily 是否按照每天 logrotate，默认是 true
  - maxdays 文件最多保存多少天，默认保存 7 天
  - rotate 是否开启 logrotate，默认是 true
  - level 日志保存的时候的级别，默认是 Trace 级别
  - perm 日志文件权限

- multifile

  设置的例子如下所示：

  ```go
  logs.SetLogger(logs.AdapterMultiFile, ``{"filename":"test.log","separate":["emergency", "alert", "critical", "error", "warning", "notice", "info", "debug"]}``)
  ```

  主要的参数如下说明(除 separate 外,均与file相同)：

  - filename 保存的文件名
  - maxlines 每个文件保存的最大行数，默认值 1000000
  - maxsize 每个文件保存的最大尺寸，默认值是 1 << 28, //256 MB
  - daily 是否按照每天 logrotate，默认是 true
  - maxdays 文件最多保存多少天，默认保存 7 天
  - rotate 是否开启 logrotate，默认是 true
  - level 日志保存的时候的级别，默认是 Trace 级别
  - perm 日志文件权限
  - separate 需要单独写入文件的日志级别,设置后命名类似 test.error.log

- conn

  网络输出，设置的例子如下所示：

  ```go
  logs.SetLogger(logs.AdapterConn, `{"net":"tcp","addr":":7020"}`)
  ```

  主要的参数说明如下：

  - reconnectOnMsg 是否每次链接都重新打开链接，默认是 false
  - reconnect 是否自动重新链接地址，默认是 false
  - net 发开网络链接的方式，可以使用 tcp、unix、udp 等
  - addr 网络链接的地址
  - level 日志保存的时候的级别，默认是 Trace 级别

- smtp

  邮件发送，设置的例子如下所示：

  ```go
  logs.SetLogger(logs.AdapterMail, `{"username":"beegotest@gmail.com","password":"xxxxxxxx","host":"smtp.gmail.com:587","sendTos":["xiemengjun@gmail.com"]}`)
  ```

  主要的参数说明如下：

  - username smtp 验证的用户名
  - password smtp 验证密码
  - host 发送的邮箱地址
  - sendTos 邮件需要发送的人，支持多个
  - subject 发送邮件的标题，默认是 `Diagnostic message from server`
  - level 日志发送的级别，默认是 Trace 级别

- ElasticSearch

  输出到 ElasticSearch:

  ```go
  logs.SetLogger(logs.AdapterEs, `{"dsn":"http://localhost:9200/","level":1}`
  ```

参考文章：

- https://beego.me/docs/mvc/controller/logs.md
- https://beego.me/docs/module/logs.md
