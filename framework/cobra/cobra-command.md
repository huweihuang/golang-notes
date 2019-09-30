> 本文以cobra-demo为例介绍cobra添加命令的具体使用操作。

# 0. cobra-demo 

`cobra-demo`编译二进制执行的结果如下。具体代码参考：https://github.com/huweihuang/cobra-demo

```bash
 $ ./cobra-demo 
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra-demo [command]

Available Commands:
  config      A brief description of your command
  help        Help about any command
  server      A brief description of your command

Flags:
      --config string   config file (default is $HOME/.cobra-demo.yaml)
  -h, --help            help for cobra-demo
  -t, --toggle          Help message for toggle

Use "cobra-demo [command] --help" for more information about a command.
```

# 1. cobra init

> cobra init 的具体使用参考 [init](http://www.huweihuang.com/framework/cobra/cobra-usage.html#31-cobra-init)

## 1.1. cobra init --pkg-name

```bash
cobra init <cobra-demo> --pkg-name github.com/huweihuang/cobra-demo -a 'author name <email>'
```

执行以上命令，创建的文件目录结构如下：

```bash
./
├── LICENSE
├── cmd
│   ├── root.go
└── main.go
```

## 1.2. main.go

```go
package main

import "github.com/huweihuang/cobra-demo/cmd"

func main() {
	cmd.Execute()
}
```

## 1.3. cmd/root.go

```go
package cmd

import (
  "fmt"
  "os"
  "github.com/spf13/cobra"

  homedir "github.com/mitchellh/go-homedir"
  "github.com/spf13/viper"

)


var cfgFile string


// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
  Use:   "cobra-demo",
  Short: "A brief description of your application",
  Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
  // Uncomment the following line if your bare application
  // has an action associated with it:
  //	Run: func(cmd *cobra.Command, args []string) { },
}

// Execute adds all child commands to the root command and sets flags appropriately.
// This is called by main.main(). It only needs to happen once to the rootCmd.
func Execute() {
  if err := rootCmd.Execute(); err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
}

func init() {
  cobra.OnInitialize(initConfig)

  // Here you will define your flags and configuration settings.
  // Cobra supports persistent flags, which, if defined here,
  // will be global for your application.

  rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra-demo.yaml)")


  // Cobra also supports local flags, which will only run
  // when this action is called directly.
  rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}


// initConfig reads in config file and ENV variables if set.
func initConfig() {
  if cfgFile != "" {
    // Use config file from the flag.
    viper.SetConfigFile(cfgFile)
  } else {
    // Find home directory.
    home, err := homedir.Dir()
    if err != nil {
      fmt.Println(err)
      os.Exit(1)
    }

    // Search config in home directory with name ".cobra-demo" (without extension).
    viper.AddConfigPath(home)
    viper.SetConfigName(".cobra-demo")
  }

  viper.AutomaticEnv() // read in environment variables that match

  // If a config file is found, read it in.
  if err := viper.ReadInConfig(); err == nil {
    fmt.Println("Using config file:", viper.ConfigFileUsed())
  }
}
```


# 2. cobra add

> cobra add 的具体使用参考 [add](http://www.huweihuang.com/framework/cobra/cobra-usage.html#32-cobra-add)

## 2.1. cobra add command

```bash
cobra add serve -a 'author name <email>'
cobra add config -a 'author name <email>'
cobra add create -p 'configCmd' -a 'author name <email>'# 在父命令config命令下创建子命令create,若没有指定-p,默认的父命令为rootCmd。
```

执行以上命令，创建的文件目录结构如下：

```bash
./
├── LICENSE
├── cmd
│   ├── config.go  # rootCmd的子命令 
│   ├── create.go  # config的子命令
│   ├── root.go    # 默认父命令
│   └── server.go  # rootCmd的子命令
└── main.go
```

## 2.2. cmd/config.go

```go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

// configCmd represents the config command
var configCmd = &cobra.Command{
	Use:   "config",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("config called")
	},
}

func init() {
	rootCmd.AddCommand(configCmd)

	// Here you will define your flags and configuration settings.

	// Cobra supports Persistent Flags which will work for this command
	// and all subcommands, e.g.:
	// configCmd.PersistentFlags().String("foo", "", "A help for foo")

	// Cobra supports local flags which will only run when this command
	// is called directly, e.g.:
	// configCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}
```

## 2.3. cmd/create.go

create为config的子命令。

```go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

// createCmd represents the create command
var createCmd = &cobra.Command{
	Use:   "create",
	Short: "A brief description of your command",
	Long: `A longer description that spans multiple lines and likely contains examples
and usage of using your command. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("create called")
	},
}

func init() {
	configCmd.AddCommand(createCmd)

	// Here you will define your flags and configuration settings.

	// Cobra supports Persistent Flags which will work for this command
	// and all subcommands, e.g.:
	// createCmd.PersistentFlags().String("foo", "", "A help for foo")

	// Cobra supports local flags which will only run when this command
	// is called directly, e.g.:
	// createCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}
```

参考：

- https://github.com/spf13/cobra
- https://github.com/spf13/cobra/blob/master/cobra/README.md
