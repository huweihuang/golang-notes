# 1. Goroutine

```go
//定义调用体
func Add(x,y int){
  z:=x+y
  fmt.Println(z)
}
//go关键字执行调用，即会产生一个goroutine并发执行
//当函数返回时，goroutine自动结束，如果有返回值,返回值会自动被丢弃
go Add(1,1)
//并发执行
func main(){
  for i:=0;i<10;i++{//主函数启动了10个goroutine，然后返回，程序退出，并不会等待其他goroutine结束
    go Add(i,i)     //所以需要通过channel通信来保证其他goroutine可以顺利执行
  }
}
```

# 2. sync.WaitGroup

sync.WaitGroup用来实现启动一组goroutine，并等待任务做完再结束goroutine。

使用方法是：

- wg.Add()：main协程通过调用 wg.Add(delta int) 设置worker协程的个数，然后创建worker协程；
- wg.Done()：worker协程执行结束以后，都要调用 wg.Done()，表示做完任务，goroutine减1；
- wg.Wait() ：main协程调用 wg.Wait() 且被block，直到所有worker协程全部执行结束后返回。
- 针对可能panic的goroutine，可以使用defer wg.Done()来结束goroutine。


示例：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	for i := 0; i <= 9; i++ {
		wg.Add(1)
		go func(i int) {
			fmt.Println(i)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

输出如下，随机输出0到9的数字

```
9
5
6
7
8
1
0
3
4
2
```

# 3. sync.Map

Go 语言原生 map 并不是线程安全的，对它进行并发读写操作的时候，需要加锁。sync.map 则是一种并发安全的 map，可以使用在并发读写map的场景中。

sync.Map常见操作：

- 写入：m.Store("1", 18)
- 读取：age, ok := m.Load("1")
- 删除：m.Delete("1")
- 遍历：m.Range(func(key, value interface{}) bool{}
- 存在则读取否则写入：	m.LoadOrStore("2", 100)

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var m sync.Map
	// 1. 写入
	m.Store("1", 18)
	m.Store("2", 20)

	// 2. 读取
	age, ok := m.Load("1")
	fmt.Println(age, ok)

	// 3. 遍历
	m.Range(func(key, value interface{}) bool {
		name := key.(string)
		age := value.(int)
		fmt.Println(name, age)
		return true
	})

	// 4. 删除
	m.Delete("1")
	age, ok = m.Load("1")
	fmt.Println(age, ok)

	// 5. 如果key存在则读取，否则写入给定的值
	m.LoadOrStore("2", 100)
	age, _ = m.Load("2")
	fmt.Println(age)
}
```

示例：

## 3.1. 不加锁的map并发读写

以下使用线程不安全的map，进行并发写入，就会出现并发报错。可以通过加锁来解决并发问题，但更推荐使用sync.Map来实现。

示例1：

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func main() {
	// 生成随机种子
	rand.Seed(time.Now().Unix())

	sm := make(map[int]int)
	var wg sync.WaitGroup
	for i := 0; i <= 9; i++ {
		wg.Add(1)
		go func(i int) {
			for j := 0; j <= 9; j++ {
				r := rand.Intn(100) // 生成0-99的随机数
				sm[j] = r  // 同时对map进行并发写入
			}

			wg.Done()
		}(i)
	}
	wg.Wait()

	// 打印map中的值
	fmt.Println(sm)
}
```

输出异常：

map不能并发写入。

```
fatal error: concurrent map writes
```

## 3.2. 加锁的并发读写

示例2：

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func main() {
	// 生成随机种子
	rand.Seed(time.Now().Unix())

	sm := &SafeMap{
		Map: make(map[int]int),
	}
	var wg sync.WaitGroup
	for i := 0; i <= 9; i++ {
		wg.Add(1)
		go func(i int) {
			for j := 0; j <= 9; j++ {
				r := rand.Intn(100) // 生成0-99的随机数
				sm.Set(i, r)        // 同时对map进行并发写入
			}

			wg.Done()
		}(i)
	}
	wg.Wait()

	// 打印map中的值
	fmt.Println(sm.Map)
}

// SafeMap
type SafeMap struct {
	Map  map[int]int
	lock sync.RWMutex // 加锁
}

// Set
func (m *SafeMap) Set(key, value int) {
	m.lock.Lock()
	defer m.lock.Unlock()
	m.Map[key] = value
}

// Get
func (m *SafeMap) Get(key int) int {
	return m.Map[key]
}

```

正常输出

```
map[0:52 1:16 2:86 3:50 4:97 5:38 6:54 7:75 8:26 9:32]
```


## 3.3. 使用sync.Map并发读写

示例3：

以下使用线程安全的sync.Map来实现对map的值进行并发的读写。

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func main() {
	// 生成随机种子
	rand.Seed(time.Now().Unix())
	// 使用线程安全的sync.Map
	var sm sync.Map
	var wg sync.WaitGroup
	for i := 0; i <= 9; i++ {
		wg.Add(1)
		go func(i int) {
			for j := 0; j <= 9; j++ {
				r := rand.Intn(100) // 生成0-99的随机数
				sm.Store(j, r)
			}

			wg.Done()
		}(i)
	}
	wg.Wait()

	// 打印map中的值
	sm.Range(func(k, v interface{}) bool {
		fmt.Println(k, v)
		return true
	})
}
```

正常输出：

```
2 92
4 5
8 48
9 6
0 50
1 64
6 27
7 86
3 59
5 57
```
