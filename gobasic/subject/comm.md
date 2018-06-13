1. 对 defer 延迟执行的函数，它的参数会在声明时候就会求出具体值，而不是在执行时才求值；
2. 对 defer 延迟执行的函数，会在调用它的函数结束时执行，而不是在调用它的语句块结束时执行，注意区分开。
```go
func main() {
  var i = 1
  // 在 defer 函数中参数会提前求值
  defer fmt.Println("result: ", func() int { return i * 2 }())
  i++
}
```


```go
// for 语句中的迭代变量在每次迭代中都会重用，
// 即 for 中创建的闭包函数接收到的参数始终是同一个变量，在 goroutine 开始执行时都会得到同一个迭代值：
// 该值都是最后的值
func main() {
  data := []string{"one", "two", "three"}

  for _, v := range data {
    go func() {
      fmt.Println(v)
    }()
  }

  time.Sleep(3 * time.Second)
  // 输出 three three three
}


func main() {
  data := []string{"one", "two", "three"}

  for _, v := range data {
    go func(in string) {
      fmt.Println(in)
    }(v)
  }

  time.Sleep(3 * time.Second)
  // 输出 one two three
}
```
