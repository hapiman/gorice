### os.Args()
`os.Args`获取所有的启动参数，返回一个数组
```sh
./jhub aa bb cc dd => [jhub, aa, bb, cc, dd]
```

### os.GetEnv()
`os.GetEnv()` 去获取命令行前传的参数 `ENV=beta  go run main.go`
```sh
os.GetEnv(“ENV”) = “beta”
```

### flag
```go
	wordPtr := flag.String("word", "foo", "a string")
	numbPtr := flag.Int("numb", 42, "an int")
  boolPtr := flag.Bool("fork", false, "a bool")

  // 使用已知的变量作为flag
	var svar string
  flag.StringVar(&svar, "svar", "bar", "a string var")

  // 都需要执行Parse()函数
	flag.Parse()
	fmt.Println("word:", *wordPtr) // 取指针的值
	fmt.Println("numb:", *numbPtr) // 取指针的值
	fmt.Println("fork:", *boolPtr) // 取指针的值
	fmt.Println("svar:", svar) // 直接取值
	fmt.Println("tail:", flag.Args())
```
```sh
# 测试1 "1 2 3 4 --word ss"皆被当作参数
go git:(master) ✗ go run main.go  1 2 3 4 --word ss
word: foo
numb: 42
fork: false
svar: bar
tail: [1 2 3 4 --word ss]

# 测试2 如果跟了参数，且使用了--不会报错，因为直接当作参数使用，默认不会有flag变量
go git:(master) ✗ go run main.go  1 2 3 4 --wor
word: foo
numb: 42
fork: false
svar: bar
tail: [1 2 3 4 --wor]

# 测试3，如果后面没有接参数名，而直接使用--会直接报错
go run main.go  --wor
flag provided but not defined: -wor
Usage of /var/folders/5q/6z9y429d1n16mjft4t85_1w40000gn/T/go-build919420242/b001/exe/main:
  -fork
        a bool
  -numb int
        an int (default 42)
  -svar string
        a string var (default "bar")
  -word string
        a string (default "foo")
exit status 2

# 测试4 一般是先接flag变量--，然后接其他的变量
go run main.go  --word ss 11 22 33
word: ss
numb: 42
fork: false
svar: bar
tail: [11 22 33]
```

### fmt.Scan 从控制台读取消息
```go
fmt.Println("请输入要查询的作品名称：")
var bookName string
fmt.Scan(&bookName)

fmt.Println("请输入作者：")
var author string
fmt.Scan(&author)

fmt.Println("要我默写哪句话？（格式 - （末尾要有标点）：___,则物与我皆无尽也，___！）")
var findStr string
fmt.Scan(&findStr)

ans, errs := StartSearch(bookName, author, findStr)
```
