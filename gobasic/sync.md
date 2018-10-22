`sync`原理说的比较明确的文章[https://deepzz.com/post/golang-sync-package-usage.html](https://deepzz.com/post/golang-sync-package-usage.html)

程序中需要并发，需要创建多个goroutine，并且一定要等这些并发全部完成后才继续接下来的程序执行．WaitGroup的特点是Wait()可以用来阻塞直到队列中的所有任务都完成时才解除阻塞，而不需要sleep一个固定的时间来等待．但是其缺点是无法指定固定的goroutine数目．


关闭channel(通过close()函数实现，关闭之后无法再向channel中存数据，但是可以继续从channel中读取数据）
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
对于该段程序（只有单核cpu运行的程序）首先创建一个无缓冲channel  ch，然后遇到go Afuntion(ch)，查看此时无cpu可以用来运行该任务，则将该任务记下，等到有cpu时再运行该任务，然后执行ch<-1，此时主goroutine阻塞，查找是否有其他协程，查找到有Afuntion(ch)这一goroutine，则执行该goroutine内容，直到<-ch才从主goroutine获取数据１，解除主goroutine阻塞．（注：这种执行方式仅限于单核cpu）

如果指定多个cpu运行，则首先运行主goroutine创建无缓冲的channel，然后查看是否有空闲cpu可以运行另外一个goroutine，如果有，则运行协程Afuntion(ch)，对于多核cpu,主goroutine和另外一个goroutine的运行顺序是不确定的．
