## 1. govendor简介

golang工程的依赖包经常使用go get 的方式来获取，例如：go get github.com/kardianos/govendor ，会将依赖包下载到GOPATH的路径下。

常用的依赖包管理工具有godep，govendor等，在Golang1.5之后，Go提供了 `GO15VENDOREXPERIMENT` 环境变量，用于将go build时的应用路径搜索调整成为 `当前项目目录/vendor` 目录方式。通过这种形式，我们可以实现类似于 `godep` 方式的项目依赖管理。

## 2. 安装与使用

### 2.1. 安装

```go
go get -u -v github.com/kardianos/govendor
```

### 2.2. 使用

```bash
#进入到项目目录
cd /home/gopath/src/mytool
#初始化vendor目录
govendor init
#查看vendor目录
[root@CC54425A mytool]# ls
commands  main.go  vendor  mytool_test.sh
#进入vendor目录
cd vendor
#将GOPATH中本工程使用到的依赖包自动移动到vendor目录中
#说明：如果本地GOPATH没有依赖包，先go get相应的依赖包
govendor add +external
#通过设置环境变量 GO15VENDOREXPERIMENT=1 使用vendor文件夹构建文件。
#可以选择 export GO15VENDOREXPERIMENT=1 或 GO15VENDOREXPERIMENT=1 go build 执行编译
export GO15VENDOREXPERIMENT=1
```

### 2.3. 说明

govendor只是用来管理项目的依赖包，如果GOPATH中本身没有项目的依赖包，则需要通过go get先下载到GOPATH中，再通过govendor add +external 拷贝到vendor目录中。

## 3. govendor的配置文件

`vendor.json`记录依赖包列表。

```json
{
    "comment": "",
    "ignore": "test",
    "package": [
        {
            "checksumSHA1": "uGalSICR4r7354vvKuNnh7Y/R/4=",
            "path": "github.com/urfave/cli",
            "revision": "b99aa811b4c1dd84cc6bccb8499c82c72098085a",
            "revisionTime": "2017-08-04T09:34:15Z"
        }
    ],
    "rootPath": "mytool"
}
```

## 4. govendor使用命令

```bash
[root@CC54425A mytool]# govendor
govendor (v1.0.8): record dependencies and copy into vendor folder
    -govendor-licenses    Show govendor's licenses.
    -version              Show govendor version
    -cpuprofile 'file'    Writes a CPU profile to 'file' for debugging.
    -memprofile 'file'    Writes a heap profile to 'file' for debugging.
Sub-Commands
    init     Create the "vendor" folder and the "vendor.json" file.
    list     List and filter existing dependencies and packages.
    add      Add packages from $GOPATH.
    update   Update packages from $GOPATH.
    remove   Remove packages from the vendor folder.
    status   Lists any packages missing, out-of-date, or modified locally.
    fetch    Add new or update vendor folder packages from remote repository.
    sync     Pull packages into vendor folder from remote repository with revisions
                 from vendor.json file.
    migrate  Move packages from a legacy tool to the vendor folder with metadata.
    get      Like "go get" but copies dependencies into a "vendor" folder.
    license  List discovered licenses for the given status or import paths.
    shell    Run a "shell" to make multiple sub-commands more efficient for large
                 projects.
    go tool commands that are wrapped:
      "+status" package selection may be used with them
    fmt, build, install, clean, test, vet, generate, tool
Status Types
    +local    (l) packages in your project
    +external (e) referenced packages in GOPATH but not in current project
    +vendor   (v) packages in the vendor folder
    +std      (s) packages in the standard library
    +excluded (x) external packages explicitly excluded from vendoring
    +unused   (u) packages in the vendor folder, but unused
    +missing  (m) referenced packages but not found
    +program  (p) package is a main package
    +outside  +external +missing
    +all      +all packages
    Status can be referenced by their initial letters.
Package specifier
    <path>[::<origin>][{/...|/^}][@[<version-spec>]]
Ignoring files with build tags, or excluding packages from being vendored:
    The "vendor.json" file contains a string field named "ignore".
    It may contain a space separated list of build tags to ignore when
    listing and copying files.
    This list may also contain package prefixes (containing a "/", possibly
    as last character) to exclude when copying files in the vendor folder.
    If "foo/" appears in this field, then package "foo" and all its sub-packages
    ("foo/bar", …) will be excluded (but package "bar/foo" will not).
    By default the init command adds the "test" tag to the ignore list.
If using go1.5, ensure GO15VENDOREXPERIMENT=1 is set.
```
