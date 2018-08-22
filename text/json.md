## JSON处理

JSON是一种轻量级的数据交换语言。

### 1. 解析JSON[Unmarshal(data []byte, v interface{})]

#### 1.1. Unmarshal源码

**/src/encoding/json/decode.go**

```go
func Unmarshal(data []byte, v interface{}) error {
    // Check for well-formedness.
    // Avoids filling out half a data structure
    // before discovering a JSON syntax error.
    var d decodeState
    err := checkValid(data, &d.scan)
    if err != nil {
        return err
    }
    d.init(data)
    return d.unmarshal(v)
}
...
func (d *decodeState) unmarshal(v interface{}) (err error) {
    defer func() {
        if r := recover(); r != nil {
            if _, ok := r.(runtime.Error); ok {
                panic(r)
            }
            err = r.(error)
        }
    }()
    rv := reflect.ValueOf(v)
    if rv.Kind() != reflect.Ptr || rv.IsNil() {
        return &InvalidUnmarshalError{reflect.TypeOf(v)}
    }
    d.scan.reset()
    // We decode rv not rv.Elem because the Unmarshaler interface
    // test must be applied at the top level of the value.
    d.value(rv)
    return d.savedError
}
```

#### 1.2. 解析到结构体

```go
package main
import (
    "encoding/json"
    "fmt"
)
type Server struct {
    ServerName string
    ServerIP   string
}
type Serverslice struct {
    Servers []Server
}
func main() {
    var s Serverslice
    str := `{"servers":
    [{"serverName":"Shanghai_VPN","serverIP":"127.0.0.1"},
    {"serverName":"Beijing_VPN","serverIP":"127.0.0.2"}]}`
    err:=json.Unmarshal([]byte(str), &s)
    if err!=nil{
        fmt.Println(err)
    }
    fmt.Println(s)
}
```

**说明**

JSON格式与结构体一一对应，Unmarshal方法即将JSON文本转换成结构体。只会匹配结构体中的可导出字段，即首字母大写字段（类似java的public），匹配规则如下：json的key为Foo为例

1. 先查找struct tag中含有Foo的可导出的struct字段（首字母大写）
2. 其次查找字段名为Foo的可导出字段。
3. 最后查找类似FOO或者FoO这类除首字母外，其他大小写不敏感的可导出字段。

#### 1.3. 解析到interface

 

### 2. 生成JSON[Marshal(v interface{})]

#### 2.1. Marshal源码

**/src/encoding/json/encode.go**

```go
func Marshal(v interface{}) ([]byte, error) {
    e := &encodeState{}
    err := e.marshal(v)
    if err != nil {
        return nil, err
    }
    return e.Bytes(), nil
}
...
func (e *encodeState) marshal(v interface{}) (err error) {
    defer func() {
        if r := recover(); r != nil {
            if _, ok := r.(runtime.Error); ok {
                panic(r)
            }
            if s, ok := r.(string); ok {
                panic(s)
            }
            err = r.(error)
        }
    }()
    e.reflectValue(reflect.ValueOf(v))
    return nil
}
```

#### 2.2. 使用方法

```go
package main
import (
    "encoding/json"
    "fmt"
)
type Server struct {
    ServerName string `json:"serverName,string"`
    ServerIP   string `json:"serverIP,omitempty"`
}
type Serverslice struct {
    Servers []Server `json:"servers"`
}
func main() {
    var s Serverslice
    s.Servers = append(s.Servers, Server{ServerName: "Shanghai_VPN", ServerIP: "127.0.0.1"})
    s.Servers = append(s.Servers, Server{ServerName: "Beijing_VPN", ServerIP: "127.0.02"})
    b, err := json.Marshal(s)
    if err != nil {
        fmt.Println("JSON ERR:", err)
    }
    fmt.Println(string(b))
}
```

#### 2.3. 说明

Marshal方法将结构体转换成json文本，匹配规则如下：

1. 如果字段的tag是“-”，那么该字段不会输出到JSON。
2. 如果tag中带有自定义名称，那么该自定义名称会出现在JSON字段名中。例如例子中的“serverName”
3. 如果tag中带有“omitempty”选项，那么如果该字段值为空，就不会输出到JSON中。
4. 如果字段类型是bool,string,int,int64等，而tag中带有“，string”选项，那么这个字段在输出到JSON的时候会把该字段对应的值转换成JSON字符串。

注意事项：

1. Marshal只有在转换成功的时候才会返回数据，JSON对象只支持string作为key，如果要编码一个map,那么必须是map[string]T这种类型。（T为任意类型）
2. Channel,complex和function不能被编码成JSON。
3. 嵌套的数据不能编码，会进入死循环。
4. 指针在编码时会输出指针指向的内容，而空指针会输出null。
