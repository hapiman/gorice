golang中实现并发非常简单，只需在需要并发的函数前面添加关键字＂Go"，但是如何处理go并发机制中不同goroutine之间的同步与通信，golang 中提供了`sync`包和`channel`机制来解决这一问题．

## sync
sync 包提供了互斥锁这类的基本的同步原语.

除`Once`和`WaitGroup`之外的类型大多用于底层库的例程。更高级的同步操作通过信道与通信进行。

```go
type Cond
    func NewCond(l Locker) *Cond
    func (c *Cond) Broadcast()
    func (c *Cond) Signal()
    func (c *Cond) Wait()
type Locker
type Mutex
    func (m *Mutex) Lock()
    func (m *Mutex) Unlock()
type Once
    func (o *Once) Do(f func())
type Pool
    func (p *Pool) Get() interface{}
    func (p *Pool) Put(x interface{})
type RWMutex
    func (rw *RWMutex) Lock()
    func (rw *RWMutex) RLock()
    func (rw *RWMutex) RLocker() Locker
    func (rw *RWMutex) RUnlock()
    func (rw *RWMutex) Unlock()
type WaitGroup
    func (wg *WaitGroup) Add(delta int)
    func (wg *WaitGroup) Done()
    func (wg *WaitGroup) Wait()
```


而golang中的同步是通过`sync.WaitGroup`来实现的．

WaitGroup的功能：它实现了一个类似队列的结构，可以一直向队列中添加任务，当任务完成后便从队列中删除，如果队列中的任务没有完全完成，可以通过`Wait()`函数来出发阻塞，防止程序继续进行，直到所有的队列任务都完成为止．

WaitGroup总共有三个方法：`Add(delta int)`,　`Done()`,　`Wait()`。

Add:添加或者减少等待goroutine的数量

Done:相当于`Add(-1)`

Wait:执行阻塞，直到所有的`WaitGroup`数量变成0

具体例子如下：
```go
package main

import (
    "fmt"
    "sync"
)

var waitgroup sync.WaitGroup

func Afunction(shownum int) {
    fmt.Println(shownum)
    waitgroup.Done() //任务完成，将任务队列中的任务数量-1，其实.Done就是.Add(-1)
}

func main() {
    for i := 0; i < 10; i++ {
        waitgroup.Add(1) //每创建一个goroutine，就把任务队列中任务的数量+1
        go Afunction(i)
    }
    waitgroup.Wait() //.Wait()这里会发生阻塞，直到队列中所有的任务结束就会解除阻塞
}
```

使用场景：

程序中需要并发，需要创建多个goroutine，并且一定要等这些并发全部完成后才继续接下来的程序执行．

WaitGroup的特点是Wait()可以用来阻塞直到队列中的所有任务都完成时才解除阻塞，而不需要sleep一个固定的时间来等待．

但是其**缺点**是无法指定固定的goroutine数目．


## channel

相对sync.WaitGroup而言，golang中利用channel实现同步则简单的多．

channel自身可以实现阻塞，其通过`<-`进行数据传递，channel是golang中一种内置基本类型，对于channel操作只有４种方式：

1. 创建channel(通过make()函数实现，包括无缓存channel和有缓存channel);

2. 向channel中添加数据（channel<-data）;

3. 从channel中读取数据（data<-channel）;

4. 关闭channel(通过close()函数实现，关闭之后无法再向channel中存数据，但是可以继续从channel中读取数据）

channel分为`有缓冲channel`和`无缓冲channel`,两种channel的创建方法如下:

`var ch = make(chan int)` //无缓冲channel,等同于make(chan int ,0)

`var ch = make(chan int,10)` //有缓冲channel,缓冲大小是５

其中无缓冲channel在读和写是都会阻塞，而有缓冲channel在向channel中存入数据没有达到channel缓存总数时，可以一直向里面存，直到缓存已满才阻塞．

由于阻塞的存在，所以使用channel时特别注意使用方法，防止**死锁**的产生．例子如下：

无缓存channel
```go
package main

import "fmt"

func Afuntion(ch chan int) {
    fmt.Println("finish")
    <-ch
}

func main() {
    ch := make(chan int) //无缓冲的channel
    go Afuntion(ch)
    ch <- 1

    // 输出结果：
    // finish
}
```

代码分析：

对于该段程序（只有单核cpu运行的程序）首先创建一个无缓冲`channel  ch`，然后遇到`go Afuntion(ch)`，查看此时无cpu可以用来运行该任务，则将该任务记下，等到有cpu时再运行该任务，然后执行ch<-1，此时主goroutine阻塞，查找是否有其他协程，查找到有Afuntion(ch)这一goroutine，则执行该goroutine内容，直到<-ch才从主goroutine获取数据１，解除主goroutine阻塞．（注：这种执行方式仅限于单核cpu）

如果指定多个cpu运行，则首先运行主goroutine创建无缓冲的channel，然后查看是否有空闲cpu可以运行另外一个goroutine，如果有，则运行协程Afuntion(ch)，对于多核cpu,主goroutine和另外一个goroutine的运行顺序是不确定的．


``` go
package main

import "fmt"
import "runtime"
import "time"

func Afuntion(ch chan int) {
    fmt.Println("finish")
    <-ch
}

func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())
    ch := make(chan int) //无缓冲的channel
    go Afuntion(ch)
    time.Sleep(time.Nanosecond * 1000)
    fmt.Println("main goroutine")
    ch <- 1
}
```

运行结果：

finishmain goroutine

或者  main goroutine

finish

主goroutine和另外一个goroutine的`执行顺序`是不确定的（对于多核cpu）


```go
package main

import "fmt"

func Afuntion(ch chan int) {
    fmt.Println("finish")
    <-ch
}

func main() {
    ch := make(chan int) //无缓冲的channel
    //只是把这两行的代码顺序对调一下
    ch <- 1
    go Afuntion(ch)

    // 输出结果：
    // 死锁，无结果
}
```



代码分析：

首先创建一个无缓冲的channel,　然后在主协程里面向channel　ch 中通过ch<-1命令写入数据，则此时主协程阻塞，就无法执行下面的go Afuntions(ch),自然也就无法解除主协程的阻塞状态，则系统死锁

总结：
对于无缓存的channel,放入channel和从channel中向外面取数据这两个操作不能放在同一个协程中，防止死锁的发生；同时应该先利用go 开一个协程对channel进行操作，此时阻塞该go 协程，然后再在主协程中进行channel的相反操作（与go 协程对channel进行相反的操作），实现go 协程解锁．即必须go协程在前，解锁协程在后．

带缓存channel:
对于带缓存channel，只要channel中缓存不满，则可以一直向 channel中存入数据，直到缓存已满；同理只要channel中缓存不为０，便可以一直从channel中向外取数据，直到channel缓存变为０才会阻塞．

由此可见，相对于不带缓存channel，带缓存channel不易造成死锁，可以同时在一个goroutine中放心使用，



### close():

close主要用来关闭channel通道其用法为close(channel)，并且实在生产者的地方关闭channel，而不是在消费者的地方关闭．

并且关闭channel后，便不可再向channel中继续存入数据，但是可以继续从channel中读取数据．例子如下：

```go
package main

import "fmt"

func main() {
    var ch = make(chan int, 20)
    for i := 0; i < 10; i++ {
        ch <- i
    }
    close(ch)
    //ch <- 11 //panic: runtime error: send on closed channel
    for i := range ch {
        fmt.Println(i)　//输出０　１　２　３　４　５　６　７　８　９
    }
}
```



### channel阻塞超时处理：
goroutine有时候会进入阻塞情况，那么如何避免由于channel阻塞导致整个程序阻塞的发生那？解决方案：通过select设置超时处理，具体程序如下：

```go
package main

 import (
    "fmt"
    "time"
)

func main() {
    c := make(chan int)
    o := make(chan bool)
    go func() {
        for {
            select {
            case i := <-c:
                fmt.Println(i)
            case <-time.After(time.Duration(3) * time.Second):    //设置超时时间为３ｓ，如果channel　3s钟没有响应，一直阻塞，则报告超时，进行超时处理．
                fmt.Println("timeout")
                o <- true
                break
            }
        }
    }()
    <-o
}
```

## golang 并发总结：

并发两种方式：

sync.WaitGroup，该方法最大优点是Wait()可以阻塞到队列中的所有任务都执行完才解除阻塞，但是它的缺点是不能够指定并发协程数量．

channel优点：能够利用带缓存的channel指定并发协程goroutine，比较灵活．但是它的缺点是如果使用不当容易造成死锁；并且他还需要自己判定并发goroutine是否执行完．

但是相对而言，channel更加灵活，使用更加方便，同时通过超时处理机制可以很好的避免channel造成的程序死锁，因此利用channel实现程序并发，更加方便，更加易用．
