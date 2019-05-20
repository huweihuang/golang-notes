# 1. bee工具

bee工具用来进行beego项目的创建、热编译、开发、测试、和部署。

安装:

```go
go get github.com/beego/bee
```

配置：

安装完之后，`bee`可执行文件默认存放在`$GOPATH/bin`里面，所以要把`$GOPATH/bin`添加到环境变量中。

# 2. bee命令

```bash
Bee is a tool for managing beego framework.
 
Usage:
 
    bee command [arguments]
 
The commands are:
 
    new         create an application base on beego framework
    run         run the app which can hot compile
    pack        compress an beego project
    api         create an api application base on beego framework
    bale        packs non-Go files to Go source files
    version     show the bee & beego version
    generate    source code generator
    migrate     run database migrations
```

说明：

## 2.1. new

在 `$GOPATH/src`的目录下执行`bee new <appname>`，会在当前目录下生成以下文件：

```go
myproject
├── conf
│   └── app.conf
├── controllers
│   └── default.go
├── main.go
├── models
├── routers
│   └── router.go
├── static
│   ├── css
│   ├── img
│   └── js
├── tests
│   └── default_test.go
└── views
    └── index.tpl
```

## 2.2. run

必须在`$GOPATH/src/appname`下执行bee run，默认监听8080端口：`http://localhost:8080/。`

## 2.3. api

`api` 命令就是用来创建 API 应用，生成以下文件：和 Web 项目相比，少了 static 和 views 目录，多了一个 test 模块，用来做单元测试。

```
apiproject
├── conf
│   └── app.conf
├── controllers
│   └── object.go
│   └── user.go
├── docs
│   └── doc.go
├── main.go
├── models
│   └── object.go
│   └── user.go
├── routers
│   └── router.go
└── tests
    └── default_test.go
```

## 2.4. pack

`pack` 目录用来发布应用的时候打包，会把项目打包成 zip 包(`apiproject.tar.gz`)，这样我们部署的时候直接把打包之后的项目上传，解压就可以部署了：

## 2.5. generate

用来自动化的生成代码的，包含了从数据库一键生成model，还包含了scaffold。

## 2.6. migrate

这个命令是应用的数据库迁移命令，主要是用来每次应用升级，降级的SQL管理。


参考：

- https://beego.me/docs/install/bee.md
