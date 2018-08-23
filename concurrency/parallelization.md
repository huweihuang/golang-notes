# 多核并行化与同步锁

## 1. 多核并行化

```go
//多核并行化
runtime.GOMAXPROCS(16) //设置环境变量GOMAXPROCS的值来控制使用多少个CPU核心
runtime.NumCPU() //来获取核心数
//出让时间片
runtime.Gosched() //在每个goroutine中控制何时出让时间片给其他goroutine
```

## 2. 同步锁

```go
//同步锁
sync.Mutex //单读单写：占用Mutex后，其他goroutine只能等到其释放该Mutex
sync.RWMutex //单写多读：会阻止写，不会阻止读
RLock() //读锁
Lock() //写锁
RUnlock() //解锁（读锁）
Unlock() //解锁（写锁）
//全局唯一性操作
//once的Do方法保证全局只调用指定函数(setup)一次，其他goroutine在调用到此函数是会阻塞，直到once调用结束才继续
once.Do(setup)
```
