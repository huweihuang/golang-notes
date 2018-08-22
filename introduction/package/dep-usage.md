## 1. dep简介

`dep`是一个golang项目的包管理工具，一般只需要2-3个命令就可以将go依赖包自动下载并归档到`vendor`的目录中。dep官网参考：https://github.com/golang/dep

## 2. dep安装

```bash
go get -u github.com/golang/dep/cmd/dep
```

## 3. dep使用

```bash
#进入到项目目录
cd /home/gopath/src/demo
#dep初始化，初始化配置文件Gopkg.toml
dep init
#dep加载依赖包，自动归档到vendor目录
dep ensure 
# 最终会生成vendor目录，Gopkg.toml和Gopkg.lock的文件
```

## 4. dep的配置文件

`Gopkg.toml`记录依赖包列表。

```bash
# Gopkg.toml example
#
# Refer to https://golang.github.io/dep/docs/Gopkg.toml.html
# for detailed Gopkg.toml documentation.
#
# required = ["github.com/user/thing/cmd/thing"]
# ignored = ["github.com/user/project/pkgX", "bitbucket.org/user/project/pkgA/pkgY"]
#
# [[constraint]]
#   name = "github.com/user/project"
#   version = "1.0.0"
#
# [[constraint]]
#   name = "github.com/user/project2"
#   branch = "dev"
#   source = "github.com/myfork/project2"
#
# [[override]]
#   name = "github.com/x/y"
#   version = "2.4.0"
#
# [prune]
#   non-go = false
#   go-tests = true
#   unused-packages = true

ignored = ["demo"]

[[constraint]]
  name = "github.com/BurntSushi/toml"
  version = "0.3.0"
  
[prune]
  go-tests = true
  unused-packages = true
```

## 5. dep-help

更多dep的命令帮助参考`dep`。

```bash
$ dep
Dep is a tool for managing dependencies for Go projects

Usage: "dep [command]"

Commands:

  init     Set up a new Go project, or migrate an existing one
  status   Report the status of the project's dependencies
  ensure   Ensure a dependency is safely vendored in the project
  prune    Pruning is now performed automatically by dep ensure.
  version  Show the dep version information

Examples:
  dep init                               set up a new project
  dep ensure                             install the project's dependencies
  dep ensure -update                     update the locked versions of all dependencies
  dep ensure -add github.com/pkg/errors  add a dependency to the project

Use "dep help [command]" for more information about a command.
```

