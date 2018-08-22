## 文件操作

更多文件操作见Go的os包。

### 1. 目录操作

- **func Mkdir(name string, perm FileMode) error**  
  创建名称为 name 的目录，权限设置是 perm，例如 0777


- **func MkdirAll(path string, perm FileMode) error**
  根据 path 创建多级子目录，例如 astaxie/test1/test2。
- **func Remove(name string) error** 
  删除名称为 name 的目录，当目录下有文件或者其他目录是会出错
- **func RemoveAll(path string) error** 
  根据 path 删除多级子目录，如果 path 是单个名称，那么该目录不删除

 

### 2. 文件操作

#### 2.1. 建立与打开文件

 **新建文件：** 

- **func Create(name string) (file \*File, err Error)**
  根据提供的文件名创建新的文件，返回一个文件对象，默认权限是 0666 的文件，返回的文件对象是可读写的。
- ** func NewFile(fd uintptr, name string) \*File** 
  根据文件描述符创建相应的文件，返回一个文件对象

**打开文件：** 

-  **func Open(name string) (file \*File, err Error)**
  该方法打开一个名称为 name 的文件，但是是只读方式，内部实现其实调用了 OpenFile。
-  **func OpenFile(name string, flag int, perm uint32) (file \*File, err Error)**
  打开名称为 name 的文件，flag 是打开的方式，只读、读写等，perm 是权限

#### 2.2. 写文件

**写文件函数：**

- **func (file \*File) Write(b []byte) (n int, err Error)** 
  写入 byte 类型的信息到文件 
- **func (file \*File) WriteAt(b []byte, off int64) (n int, err Error)**
  在指定位置开始写入 byte 类型的信息 
- **func (file \*File) WriteString(s string) (ret int, err Error)**
  写入 string 信息到文件

```go
package main
import (
    "fmt"
    "os"
)
func main() {
    userFile := "test.txt"
    fout, err := os.Create(userFile)
    defer fout.Close()
    if err != nil {
        fmt.Println(userFile, err)
        return
    }
    for i := 0; i < 10; i++ {
        fout.WriteString("Just a test!\r\n")
        fout.Write([]byte("Just a test!\r\n"))
    }
}
```

#### 2.3. 读文件  

**读文件函数：**

- func (file *File) Read(b []byte) (n int, err Error) 
  读取数据到 b 中 


- func (file *File) ReadAt(b []byte, off int64) (n int, err Error)
   从 off 开始读取数据到 b 中

```go
package main
import (
    "fmt"
    "os"
)
func main() {
    userFile := "text.txt"
    fl, err := os.Open(userFile)
    defer fl.Close()
    if err != nil {
        fmt.Println(userFile, err)
        return
    }
    buf := make([]byte, 1024)
    for {
        n, _ := fl.Read(buf)
        if 0 == n {
            break
        }
        os.Stdout.Write(buf[:n])
    }
}
```

#### 2.4. 删除文件

Go 语言里面删除文件和删除文件夹是同一个函数

- func Remove(name string) Error
  调用该函数就可以删除文件名为 name 的文件
