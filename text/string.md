## 字符串处理

字符串操作涉及的标准库有**strings**和**strconv**两个包

### 1. 字符串操作

| 函数                                       | 说明                                       |
| ---------------------------------------- | ---------------------------------------- |
| func Contains(s, substr string) bool     | 字符串 s 中是否包含 substr，返回 bool 值             |
| func Join(a []string, sep string) string | 字符串链接，把 slice a 通过 sep 链接起来              |
| func Index(s, sep string) int            | 在字符串 s 中查找 sep 所在的位置，返回位置值，找不到返回-1       |
| func Repeat(s string, count int) string  | 重复 s 字符串 count 次，最后返回重复的字符串              |
| func Replace(s, old, new string, n int) string | 在 s 字符串中，把 old 字符串替换为 new 字符串，n 表示替换的次数，小于 0 表示全部替换 |
| func Split(s, sep string) []string       | 把 s 字符串按照 sep 分割，返回 slice                |
| func Trim(s string, cutset string) string | 在 s 字符串中去除 cutset 指定的字符串                 |
| func Fields(s string) []string           | 去除 s 字符串的空格符，并且按照空格分割返回 slice            |

### 2. 字符串转换

1、Append 系列函数将整数等转换为字符串后，添加到现有的字节数组中

```go
package main
import (
    "fmt"
    "strconv"
)
func main() {
    str := make([]byte, 0, 100)
    str = strconv.AppendInt(str, 4567, 10)
    str = strconv.AppendBool(str, false)
    str = strconv.AppendQuote(str, "abcdefg")
    str = strconv.AppendQuoteRune(str, '单')
    fmt.Println(string(str))
}
```

2、Format 系列函数把其他类型的转换为字符串

```go
package main
import (
    "fmt"
    "strconv"
)
func main() {
    a := strconv.FormatBool(false)
    b := strconv.FormatFloat(123.23, 'g', 12, 64)
    c := strconv.FormatInt(1234, 10)
    d := strconv.FormatUint(12345, 10)
    e := strconv.Itoa(1023)
    fmt.Println(a, b, c, d, e)
}
```

3、Parse 系列函数把字符串转换为其他类型

```go
package main
import (
    "fmt"
    "strconv"
)
func main() {
     
    a, err := strconv.ParseBool("false")
    if err != nil {
        fmt.Println(err)
    }
    b, err := strconv.ParseFloat("123.23", 64)
    if err != nil {
        fmt.Println(err)
    }
    c, err := strconv.ParseInt("1234", 10, 64)
    if err != nil {
        fmt.Println(err)
    }
    d, err := strconv.ParseUint("12345", 10, 64)
    if err != nil {
        fmt.Println(err)
    }
    e, err := strconv.Itoa("1023")
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(a, b, c, d, e)
}
```
