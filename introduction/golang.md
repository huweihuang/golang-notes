# 初识Go语言

## 1. 概述

一个在语言层面实现了并发机制的类C通用型编程语言。

## 2. Go关键字（25个）

| 类别     | 关键字                                     | 说明                         |
| ------ | --------------------------------------- | -------------------------- |
| 程序声明   | package，import                          | 包的声明和导入                    |
| 声明与定义  | var，const                               | 变量和常量的声明                   |
|        | type                                    | 用于定义类型                     |
| 复合数据类型 | struct                                  | 定义结构体，类似java中的class        |
|        | interface                               | 定义接口                       |
|        | map                                     | 定义键值对                      |
|        | func                                    | 定义函数和方法                    |
|        | chan                                    | 定义管道，并发中channel通信          |
| 并发编程   | go                                      | 并发编程                       |
|        | select                                  | 用于选择不同类型通信                 |
| 流程控制   | for；if，else；switch，case                 | 循环语句；条件语句；选择语句             |
|        | break，continue，fallthrough，default，goto | 跳转语句等                      |
|        | return                                  | 函数返回值                      |
|        | defer                                   | 延迟函数，用于return前释放资源         |
|        | range                                   | 用于读取slice，map，channel容器类数据 |

## 3. Go语言命令

 Usage：go command [arguments]

| 分类   | 命令       | 说明                                       |
| ---- | -------- | ---------------------------------------- |
|      | build    | compile packages and dependencies        |
|      | clean    | remove object files                      |
|      | doc      | show documentation for package or symbol |
|      | env      | print Go environment information         |
|      | fix      | run go tool fix on packages              |
|      | fmt      | run gofmt on package sources             |
|      | generate | generate Go files by processing source   |
|      | get      | download and install packages and dependencies |
|      | install  | compile and install packages and dependencies |
|      | list     | list packages                            |
|      | run      | compile and run Go program               |
|      | test     | test packages                            |
|      | tool     | run specified go tool                    |
|      | version  | print Go version                         |
|      | vet      | run go tool vet on packages              |
