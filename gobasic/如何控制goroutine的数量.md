1. 为什么要控制`goroutine`数量?

goroutine固然好，但是数量太多了，往往会带来很多麻烦，比如耗尽系统资源导致程序崩溃，或者CPU使用率过高导致系统忙不过来。比如：
```go
for i:=0; i < 10000; i++ {
    go work()
}
```

2 使用通道限制`goroutine`的数量
```go
var ch chan int

func work() {
    //do something
    <-ch
}

func main() {
    // 开启10个goroutine执行任务
    // 当chan中任务为10时, 直接阻塞,不能继续执行任务
    ch = make(chan int, 10)
    for i:=0; i < 10000; i++ {
       ch <- 1
       go work()
    }
}
```
使用上面这种方式, 主程序直接退出, 然而`work`任务没有执行完, 转而使用`sync.WaitGroup`, `wg.Done`, `wg.Add`,`wg.Wait`方式
即: `wg := &sync.WaitGroup{}`, `wg.Add(1)`, `defer wg.Done()`, `wg.Wait()`
```go
var wg *sync.WaitGroup

func work() {
    defer wg.Done()
    //do something
}

func main() {
    wg = &sync.WaitGroup{}
    for i:=0; i < 10000; i++ {
       wg.Add(1)
       go work()
    }
    wg.Wait()//等待所有goroutine退出
}
```
