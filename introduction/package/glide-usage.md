# 1. glide简介

glide是一个golang项目的包管理工具，非常方便快捷，一般只需要2-3个命令就可以将go依赖包自动下载并归档到vendor的目录中。

# 2. glide安装

```bash
go get github.com/Masterminds/glide
```

# 3. glide使用

```bash
#进入到项目目录
cd /home/gopath/src/demo
#glide初始化，初始化配置文件glide.yaml
glide init
#glide加载依赖包，自动归档到vendor目录
glide up -v
```

# 4. glide的配置文件

`glide.yaml`记录依赖包列表。

```bash
package: demo
import:
- package: github.com/astaxie/beego
  version: v1.9.2
testImport:
- package: github.com/smartystreets/goconvey
  version: 1.6.3
  subpackages:
  - convey
```

# 5. glide-help

更多glide的命令帮助参考`glide —help`。

```bash
➜  demo glide --help
NAME:
   glide - Vendor Package Management for your Go projects.
 
   Each project should have a 'glide.yaml' file in the project directory. Files
   look something like this:
 
       package: github.com/Masterminds/glide
       imports:
       - package: github.com/Masterminds/cookoo
         version: 1.1.0
       - package: github.com/kylelemons/go-gypsy
         subpackages:
         - yaml
 
   For more details on the 'glide.yaml' files see the documentation at
   https://glide.sh/docs/glide.yaml
 
 
USAGE:
   glide [global options] command [command options] [arguments...]
 
VERSION:
   0.13.2-dev
 
COMMANDS:
     create, init       Initialize a new project, creating a glide.yaml file
     config-wizard, cw  Wizard that makes optional suggestions to improve config in a glide.yaml file.
     get                Install one or more packages into `vendor/` and add dependency to glide.yaml.
     remove, rm         Remove a package from the glide.yaml file, and regenerate the lock file.
     import             Import files from other dependency management systems.
     name               Print the name of this project.
     novendor, nv       List all non-vendor paths in a directory.
     rebuild            Rebuild ('go build') the dependencies
     install, i         Install a project's dependencies
     update, up         Update a project's dependencies
     tree               (Deprecated) Tree prints the dependencies of this project as a tree.
     list               List prints all dependencies that the present code references.
     info               Info prints information about this project
     cache-clear, cc    Clears the Glide cache.
     about              Learn about Glide
     mirror             Manage mirrors
     help, h            Shows a list of commands or help for one command
 
GLOBAL OPTIONS:
   --yaml value, -y value  Set a YAML configuration file. (default: "glide.yaml")
   --quiet, -q             Quiet (no info or debug messages)
   --debug                 Print debug verbose informational messages
   --home value            The location of Glide files (default: "/Users/meitu/.glide") [$GLIDE_HOME]
   --tmp value             The temp directory to use. Defaults to systems temp [$GLIDE_TMP]
   --no-color              Turn off colored output for log messages
   --help, -h              show help
   --version, -v           print the version
```

