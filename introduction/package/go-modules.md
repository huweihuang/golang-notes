# 1. Go modules简介

`Go 1.11`版本开始支持`Go modules`方式的依赖包管理功能，官网参考：https://github.com/golang/go/wiki/Modules 。


# 2. [go mod的使用](https://github.com/golang/go/wiki/Modules#how-to-use-modules)

项目文件如下：

hello.go

```go
package main

import (
    "fmt"
    "rsc.io/quote"
)

func main() {
    fmt.Println(quote.Hello())
}
```

操作记录：

```bash
# 安装GO 1.11及以上版本
go version
go version go1.12.5 darwin/amd64

# 开启module功能
export GO111MODULE=on

# 进入到项目目录
cd /home/gopath/src/hello

# 初始化
go mod init
go: creating new go.mod: module hello

# 编译
go build
go: finding rsc.io/quote v1.5.2
go: downloading rsc.io/quote v1.5.2
go: extracting rsc.io/quote v1.5.2
go: finding rsc.io/sampler v1.3.0
go: finding golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: downloading rsc.io/sampler v1.3.0
go: extracting rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
go: extracting golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c

# 不添加vendor目录
go mod tidy -v

# 如果添加vendor目录，则执行vendor参数
go mod vendor -v

# 命令输出如下:
# golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
golang.org/x/text/language
golang.org/x/text/internal/tag
# rsc.io/quote v1.5.2
rsc.io/quote
# rsc.io/sampler v1.3.0
rsc.io/sampler

# 文件目录结构
./
├── go.mod
├── go.sum
├── hello  # 二进制文件
├── hello.go
└── vendor
    ├── golang.org
    ├── modules.txt
    └── rsc.io
```

# 3. go mod的相关文件

## 3.1. go.mod

文件路径：项目根目录下

```bash
module hello

go 1.12

require rsc.io/quote v1.5.2
```

## 3.2. go.sum

文件路径：项目根目录下

```bash
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZOaTkIIMiVjBQcw93ERBE4m30iBm00nkL0i8=
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3Y=
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPXsUe+TKr0=
rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/QiW4=
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9JXDnKaTXpA=
```

## 3.3. modules.txt

文件路径：/{project}/vendor/modules.txt

```bash
# golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
golang.org/x/text/language
golang.org/x/text/internal/tag
# rsc.io/quote v1.5.2
rsc.io/quote
# rsc.io/sampler v1.3.0
rsc.io/sampler
```

# 4. go mod的帮助信息

```bash
go help mod
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

	go mod <command> [arguments]

The commands are:

	download    download modules to local cache
	edit        edit go.mod from tools or scripts
	graph       print module requirement graph
	init        initialize new module in current directory
	tidy        add missing and remove unused modules
	vendor      make vendored copy of dependencies
	verify      verify dependencies have expected content
	why         explain why packages or modules are needed

Use "go help mod <command>" for more information about a command.
```

## 4.1. go mod init

```bash
go help mod init
usage: go mod init [module]

Init initializes and writes a new go.mod to the current directory,
in effect creating a new module rooted at the current directory.
The file go.mod must not already exist.
If possible, init will guess the module path from import comments
(see 'go help importpath') or from version control configuration.
To override this guess, supply the module path as an argument.
```

## 4.2. go mod tidy

```
usage: go mod tidy [-v]

Tidy makes sure go.mod matches the source code in the module.
It adds any missing modules necessary to build the current module's
packages and dependencies, and it removes unused modules that
don't provide any relevant packages. It also adds any missing entries
to go.sum and removes any unnecessary ones.

The -v flag causes tidy to print information about removed modules
to standard error.
```

## 4.3. go mod vendor

```bash
go help mod vendor
usage: go mod vendor [-v]

Vendor resets the main module's vendor directory to include all packages
needed to build and test all the main module's packages.
It does not include test code for vendored packages.

The -v flag causes vendor to print the names of vendored
modules and packages to standard error.
```


参考：

- https://github.com/golang/go/wiki/Modules
- https://blog.golang.org/modules2019
- https://blog.golang.org/using-go-modules 
