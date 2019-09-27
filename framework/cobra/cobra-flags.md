#  添加Flags

## 1. Persistent Flags

`Persistent Flags`表示该类参数可以被用于当前命令及其子命令。

例如，以下表示`verbose`参数可以被用于`rootCmd`及其子命令。

```go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```

## 2. Local Flags

`Local Flags`表示该类参数只能用于当前命令。

例如，以下表示`source`只能用于`localCmd`这个命令，不能用于其子命令。

```go
localCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

## 3. Local Flag on Parent Commands

cobra默认只解析当前命令的local flags，通过开启`Command.TraverseChildren`参数，可以解析每个命令的local flags。

```bash
command := cobra.Command{
  Use: "print [OPTIONS] [COMMANDS]",
  TraverseChildren: true,
}
```

## 4. Bind Flags with Config

可以通过 [viper](https://github.com/spf13/viper)来绑定flags。

```bash
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```

更多参考： [viper documentation](https://github.com/spf13/viper#working-with-flags)。

## 5. Required flags

默认添加的flags的`可选`参数，如果需要在二进制运行时添加`必要`参数，即当该参数没指定时会报错。可使用以下设置。

```go
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region")
```



参考：

- https://github.com/spf13/cobra
- https://github.com/spf13/cobra/blob/master/cobra/README.md
