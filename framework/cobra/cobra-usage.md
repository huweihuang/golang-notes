# 1. Cobra简介

`Cobra`是一个cli接口模式的应用程序框架，同时也是生成该框架的命令行工具。用户可以通过`help`方式快速查看该二进制的使用方式。

`Cobra`主要包括以下部分

- `Command`:一般表示action，即运行的二进制命令服务。同时可以拥有子命令（children commands）。
- `Args`:命令执行相关参数。
- `Flags`:二进制命令的配置参数，可对应配置文件。参数可分为全局参数和子命令参数。参考：[pflag library](https://github.com/spf13/pflag)。

# 2. 安装

通过以下操作可以在`$GOPATH/bin`安装cobra的二进制命令。

```bash
go get -u github.com/spf13/cobra/cobra
```

# 3. 使用

`cobra`命令行帮助信息如下：

```bash
# cobra
Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra [command]

Available Commands:
  add         Add a command to a Cobra Application
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)

Use "cobra [command] --help" for more information about a command.
```

## 3.1. cobra init

```bash
# cobra init --help
Initialize (cobra init) will create a new application, with a license
and the appropriate structure for a Cobra-based CLI application.

  * If a name is provided, it will be created in the current directory;
  * If no name is provided, the current directory will be assumed;
  * If a relative path is provided, it will be created inside $GOPATH
    (e.g. github.com/spf13/hugo);
  * If an absolute path is provided, it will be created;
  * If the directory already exists but is empty, it will be used.

Init will not use an existing directory with contents.

Usage:
  cobra init [name] [flags]

Aliases:
  init, initialize, initialise, create

Flags:
  -h, --help              help for init
      --pkg-name string   fully qualified pkg name

Global Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)
```


## 3.2. cobra add

```bash
# cobra add --help
Add (cobra add) will create a new command, with a license and
the appropriate structure for a Cobra-based CLI application,
and register it to its parent (default rootCmd).

If you want your command to be public, pass in the command name
with an initial uppercase letter.

Example: cobra add server -> resulting in a new cmd/server.go

Usage:
  cobra add [command name] [flags]

Aliases:
  add, command

Flags:
  -h, --help            help for add
  -p, --parent string   variable name of parent command for this command (default "rootCmd")

Global Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)
```



参考：

- https://github.com/spf13/cobra
- https://github.com/spf13/cobra/blob/master/cobra/README.md
