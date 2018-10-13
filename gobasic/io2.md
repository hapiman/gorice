以下三个文件都可以追根到`Reader`
`strings.NewReader("string")`
`os.Stdin`
`file, _ := os.Open("main.go")`中的`file`文件

```go
// 使用io.Reader参数
func ReadFrom(reader io.Reader, num int) ([]byte, error) {
	p := make([]byte, num)
	n, err := reader.Read(p)
	if n > 0 {
		return p[:n], nil
	}
	return p, err
}

func sampleReadFromString() {

	// 从字符串读取
	data, _ := ReadFrom(strings.NewReader("from string"), 12)

	fmt.Println(data)
}

func sampleReadStdin() {

	fmt.Println("please input from stdin:")

	// 从标准输入读取
	data, _ := ReadFrom(os.Stdin, 11)

	fmt.Println(data)
}

func sampleReadFile() {

	file, _ := os.Open("main.go")
	defer file.Close()

	// 从普通文件读取，其中file是os.File的实例
	data, _ := ReadFrom(file, 9)

	fmt.Println(data)
}
```

```go
	// 使用bufio.NewReader将数据置于缓冲区中, 然后从缓冲区读数据
	strReader := strings.NewReader("hello world")
	bufReader := bufio.NewReader(strReader)
	// 只读拾取, Peek只是看一看
	data, _ := bufReader.Peek(5)

	// bufReader.Buffered() 当前缓冲的数据总量
	fmt.Println(data, bufReader.Buffered())

	// 读出hello ， 查看剩余,
	// bufReader.ReadString 读到什么为止, 包括字段, 从缓存区重化工取出
	str, _ := bufReader.ReadString(' ')

	// 取出
	fmt.Println(str, bufReader.Buffered())

	// 写入
	// os.Stdout表示屏幕, 将数据输出到屏幕中, 注意使用Flush()方法
	w := bufio.NewWriter(os.Stdout)

	fmt.Fprint(w, "Hello, ")
	fmt.Fprint(w, "world!")
	w.Flush() // Don't forget to flush!
```
