### chan 入门文档
- https://blog.csdn.net/skh2015java/article/details/60330785
- https://blog.csdn.net/netdxy/article/details/54564436
- [Go Channel 详解](http://colobu.com/2016/04/14/Golang-Channels/)
- [https://www.jianshu.com/p/42e89de33065](https://www.jianshu.com/p/42e89de33065)
### 注意事项
`for range`，如果没有关闭通道`chan`，会一直阻塞

如何一直处理某个case
```go
for {
  select {
    case c <- x:
      x,y = y, x+y
    case <-quit:
      fmt.Println("Quit")
      return
    case  <- time.After(time.Second * 1):
      fmt.Println("xxx")
  }
}
```

```go
// 使用chan控制并发的数量，问题是如果设置任务数量过大，不设置处理时间，会出现死锁问题
package main

import (
    "fmt"
    "time"
)

type Demo struct {
    input         chan string
    output        chan string
    goroutine_cnt chan int
}

func NewDemo() *Demo {
    d := new(Demo)
    d.input = make(chan string, 8192)
    d.output = make(chan string, 8192)
    d.goroutine_cnt = make(chan int, 10)
    return d
}

func (this *Demo) Goroutine() {
    this.input <- time.Now().Format("2006-01-02 15:04:05")
    time.Sleep(time.Millisecond * 500)
    <-this.goroutine_cnt
}

func (this *Demo) Handle() {
    for t := range this.input {
        fmt.Println("datatime is :", t, "goroutine count is :", len(this.goroutine_cnt))
        this.output <- t + "handle"
    }
}

func main() {
    demo := NewDemo()
    go demo.Handle()
    for i := 0; i < 10000; i++ {
        demo.goroutine_cnt <- 1
        go demo.Goroutine()
    }
    close(demo.input)
}
```
```go
package gpool

import (
	"sync"
)

type pool struct {
	queue chan int
	wg    *sync.WaitGroup
}

func New(size int) *pool {
	if size <= 0 {
		size = 1
	}
	return &pool{
		queue: make(chan int, size),
		wg:    &sync.WaitGroup{},
	}
}

func (p *pool) Add(delta int) {
	for i := 0; i < delta; i++ {
		p.queue <- 1
	}
	for i := 0; i > delta; i-- {
		<-p.queue
	}
	p.wg.Add(delta)
}

func (p *pool) Done() {
	<-p.queue
	p.wg.Done()
}

func (p *pool) Wait() {
	p.wg.Wait()
}
// 测试
package gpool_test

import (
	"runtime"
	"testing"
	"time"
	"gpool"
)

func Test_Example(t *testing.T) {
	pool := gpool.New(100)
	println(runtime.NumGoroutine())
	for i := 0; i < 1000; i++ {
		pool.Add(1)
		go func() {
			time.Sleep(time.Second)
			println(runtime.NumGoroutine())
			pool.Done()
		}()
	}
	pool.Wait()
	println(runtime.NumGoroutine())
}
```
