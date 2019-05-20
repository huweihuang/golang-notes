# 1. beego的使用

## 1.1. beego的安装

```go
go get github.com/astaxie/beego
```

## 1.2. beego的升级

1、直接升级

```go
go get -u github.com/astaxie/beego
```

2、源码下载升级

用户访问 `https://github.com/astaxie/beego` ,下载源码，然后覆盖到 `$GOPATH/src/github.com/astaxie/beego` 目录，然后通过本地执行安装就可以升级了：

```go
go install  github.com/astaxie/beego
```

# 2. beego的架构

beego 是一个快速开发 Go 应用的 HTTP 框架，他可以用来快速开发 API、Web 及后端服务等各种应用，是一个 RESTful 的框架。

## 2.1. beego架构图

![architecture](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578706/article/golang/beego/architecture.png)

beego 是基于八大独立的模块构建的，是一个高度解耦的框架。

可以使用 cache 模块来做你的缓存逻辑；使用日志模块来记录你的操作信息；使用 config 模块来解析你各种格式的文件。

## 2.2. beego执行逻辑

![flow](https://res.cloudinary.com/dqxtn0ick/image/upload/v1510578706/article/golang/beego/flow.png)

 

参考：

- https://beego.me/docs/intro/
- https://beego.me/docs/install/
