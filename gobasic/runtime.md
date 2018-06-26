- NumCPU CPU数量
- GOROOT 当前GO的Root
- GOOS 查看目标操作系统
```go
func main() {
    fmt.Println("cpus:", runtime.NumCPU())
    fmt.Println("goroot:", runtime.GOROOT())
    fmt.Println("os/platform:", runtime.GOOS)
}
```
- GOMAXPROCS 设置使用CPU核数
- Gosched 设置让出CPU的使用权

```go
func main() {
    runtime.GOMAXPROCS(1)
    exit := make(chan int)
    go func() {
        defer close(exit)
        go func() {
            fmt.Println("b")
        }()
    }()

    for i := 0; i < 10; i++ {
        fmt.Println("a:", i)

        if i == 4 {
            runtime.Gosched() //切换任务
        }
    }
    <-exit
}
/*
a: 0
a: 1
a: 2
a: 3
a: 4
b
a: 5
a: 6
a: 7
a: 8
a: 9
*/

func main() {
    runtime.GOMAXPROCS(2)
    exit := make(chan int)
    go func() {
        defer close(exit)
        go func() {
            fmt.Println("b")
        }()
    }()

    for i := 0; i < 10; i++ {
        fmt.Println("a:", i)

        if i == 4 {
            runtime.Gosched() //切换任务
        }
    }
    <-exit
}
/*
b
a: 0
a: 1
a: 2
a: 3
a: 4
a: 5
a: 6
a: 7
a: 8
a: 9
*/

// 使用GOMAXPROCS和sync包处理逻辑
func main() {
    //runtime.GOMAXPROCS(1)
    runtime.GOMAXPROCS(2)

    var wg sync.WaitGroup
    wg.Add(2)

    fmt.Println("Starting Go Routines")
    go func() {
        defer wg.Done()

        for char := 'a'; char < 'a'+26; char++ {
            fmt.Printf("%c ", char)
        }
    }()

    go func() {
        defer wg.Done()

        for number := 1; number < 27; number++ {
            fmt.Printf("%d ", number)
        }
    }()

    fmt.Println("Waiting To Finish")
    wg.Wait()

    fmt.Println("\nTerminating Program")
}
```
