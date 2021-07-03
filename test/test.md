# 1. Go中的测试框架

Go语言中自带有一个轻量级的测试框架`testing`和自带的`go test`命令来实现单元测试和性能测试，`testing`框架和其他语言中的测试框架类似，你可以基于这个框架写针对相应函数的测试用例，也可以基于该框架写相应的压力测试用例。

# 2. 单元测试原则

- 文件名必须是`_test.go`结尾的，这样在执行`go test`的时候才会执行到相应的代码
- 你必须import `testing`这个包
- 所有的测试用例函数必须是`Test`开头
- 测试用例会按照源代码中写的顺序依次执行
- 测试函数`TestXxx()`的参数是`testing.T`，我们可以使用该类型来记录错误或者是测试状态
- 测试格式：`func TestXxx (t *testing.T)`,`Xxx`部分可以为任意的字母数字的组合，但是首字母不能是小写字母[a-z]，例如`Testintdiv`是错误的函数名。
- 函数中通过调用`testing.T`的`Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf`方法，说明测试不通过，调用`Log`方法用来记录测试的信息。

# 3 测试常用命令

```
# 测试整个目录
go test -v ./pkg/... ./cmd/... -coverprofile cover.out

# 测试某个文件
go test -v  file_test.go file.go 

# 测试某个函数
go test -v -test.run TestFunction
```


# 4. 示例

## 4.1. 源文件getest.go

```go
package gotest
import (
    "errors"
)
func Division(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return a / b, nil
}
```

## 4.2. 测试文件gotest_test.go

```go
func Test_Division_2(t *testing.T) {
    if _, e := Division(6, 0); e == nil { //try a unit test on function
        t.Error("Division did not work as expected.") // 如果不是如预期的那么就报错
    } else {
        t.Log("one test passed.", e) //记录一些你期望记录的信息
    }
}
```

# 5. 压力测试

压力测试用来检测函数(方法）的性能，和编写单元功能测试的方法类似。

- 压力测试用例必须遵循如下格式，其中XXX可以是任意字母数字的组合，但是首字母不能是小写字母

```
		func BenchmarkXXX(b *testing.B) { ... }
```

- `go test`不会默认执行压力测试的函数，如果要执行压力测试需要带上参数`-test.bench`，语法:`-test.bench="test_name_regex"`,例如`go test -test.bench=".*"`表示测试全部的压力测试函数
- 在压力测试用例中,请记得在循环体内使用`testing.B.N`,以使测试可以正常的运行
- 文件名也必须以`_test.go`结尾

## 5.1. 示例

```go
package gotest
     
import (
    "testing"
)
func Benchmark_Division(b *testing.B) {
    for i := 0; i < b.N; i++ { //use b.N for looping
        Division(4, 5)
    }
}
func Benchmark_TimeConsumingFunction(b *testing.B) {
    b.StopTimer() //调用该函数停止压力测试的时间计数
    //做一些初始化的工作,例如读取文件数据,数据库连接之类的,
    //这样这些时间不影响我们测试函数本身的性能
    b.StartTimer() //重新开始时间
    for i := 0; i < b.N; i++ {
        Division(4, 5)
    }
}
```

执行测试命令

```go
go test -file webbench_test.go -test.bench=".*"
```
