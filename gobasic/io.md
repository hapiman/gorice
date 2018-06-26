# 1.1 io — 基本的 IO 接口 #

io 包为 I/O 原语提供了基本的接口。它主要包装了这些原语的已有实现。

由于这些接口和原语以不同的实现包装了低级操作，因此除非另行通知，否则客户端不应假定它们对于并行执行是安全的。

在 io 包中最重要的是两个接口：Reader 和 Writer 接口。本章所提到的各种 IO 包，都跟这两个接口有关，也就是说，只要实现了这两个接口，它就有了 IO 的功能。

## Reader 接口 ##

Reader 接口的定义如下：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

官方文档中关于该接口方法的说明：

> Read 将 len(p) 个字节读取到 p 中。它返回读取的字节数 n（0 <= n <= len(p)） 以及任何遇到的错误。即使 Read 返回的 n < len(p)，它也会在调用过程中使用 p 的全部作为暂存空间。若一些数据可用但不到 len(p) 个字节，Read 会照例返回可用的数据，而不是等待更多数据。

> 当 Read 在成功读取 n > 0 个字节后遇到一个错误或 EOF (end-of-file)，它就会返回读取的字节数。它会从相同的调用中返回（非nil的）错误或从随后的调用中返回错误（同时 n == 0）。 一般情况的一个例子就是 Reader 在输入流结束时会返回一个非零的字节数，同时返回的 err 不是 EOF 就是 nil。无论如何，下一个 Read 都应当返回 0, EOF。

> 调用者应当总在考虑到错误 err 前处理 n > 0 的字节。这样做可以在读取一些字节，以及允许的 EOF 行为后正确地处理 I/O 错误。

也就是说，当 Read 方法返回错误时，不代表没有读取到任何数据。调用者应该处理返回的任何数据，之后才处理可能的错误。

根据 Go 语言中关于接口和实现了接口的类型的定义（[Interface_types](http://golang.org/ref/spec#Interface_types)），我们知道 Reader 接口的方法集（[Method_sets](http://golang.org/ref/spec#Method_sets)）只包含一个 Read 方法，因此，所有实现了 Read 方法的类型都实现了 io.Reader 接口，也就是说，在所有需要 io.Reader 的地方，可以传递实现了 Read() 方法的类型的实例。

下面，我们通过具体例子来谈谈该接口的用法。

```go
func ReadFrom(reader io.Reader, num int) ([]byte, error) {
	p := make([]byte, num)
	n, err := reader.Read(p)
	if n > 0 {
		return p[:n], nil
	}
	return p, err
}
```

ReadFrom 函数将 io.Reader 作为参数，也就是说，ReadFrom 可以从任意的地方读取数据，只要来源实现了 io.Reader 接口。比如，我们可以从标准输入、文件、字符串等读取数据，示例代码如下：

```go
// 从标准输入读取
data, err = ReadFrom(os.Stdin, 11)

// 从普通文件读取，其中 file 是 os.File 的实例
data, err = ReadFrom(file, 9)

// 从字符串读取
data, err = ReadFrom(strings.NewReader("from string"), 12)
```

完整的演示例子源码见 [code/src/chapter01/io/reader.go](/code/src/chapter01/io/reader.go)

**小贴士**

io.EOF 变量的定义：`var EOF = errors.New("EOF")`，是 error 类型。根据 reader 接口的说明，在 n > 0 且数据被读完了的情况下，返回的 error 有可能是 EOF 也有可能是 nil。

## Writer 接口 ##

Writer 接口的定义如下：

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

官方文档中关于该接口方法的说明：

> Write 将 len(p) 个字节从 p 中写入到基本数据流中。它返回从 p 中被写入的字节数 n（0 <= n <= len(p)）以及任何遇到的引起写入提前停止的错误。若 Write 返回的 n < len(p)，它就必须返回一个 非nil 的错误。

同样的，所有实现了Write方法的类型都实现了 io.Writer 接口。

在上个例子中，我们是自己实现一个函数接收一个 io.Reader 类型的参数。这里，我们通过标准库的例子来学习。

在fmt标准库中，有一组函数：Fprint/Fprintf/Fprintln，它们接收一个 io.Wrtier 类型参数（第一个参数），也就是说它们将数据格式化输出到 io.Writer 中。那么，调用这组函数时，该如何传递这个参数呢？

我们以 fmt.Fprintln 为例，同时看一下 fmt.Println 函数的源码。

```go
func Println(a ...interface{}) (n int, err error) {
	return Fprintln(os.Stdout, a...)
}
```

很显然，fmt.Println会将内容输出到标准输出中。下一节我们将详细介绍fmt包。

关于 io.Writer 的更多说明，可以查看笔者之前写的博文[《以io.Writer为例看go中的interface{}》](http://blog.studygolang.com/2013/02/%e4%bb%a5io-writer%e4%b8%ba%e4%be%8b%e7%9c%8bgo%e4%b8%ad%e7%9a%84interface/)。

## 实现了 io.Reader 接口或 io.Writer 接口的类型 ##

初学者看到函数参数是一个接口类型，很多时候有些束手无策，不知道该怎么传递参数。还有人问：标准库中有哪些类型实现了 io.Reader 或 io.Writer 接口？

通过本节上面的例子，我们可以知道，os.File 同时实现了这两个接口。我们还看到 os.Stdin/Stdout 这样的代码，它们似乎分别实现了 io.Reader/io.Writer 接口。没错，实际上在 os 包中有这样的代码：

```go
var (
    Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
    Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
    Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```

也就是说，Stdin/Stdout/Stderr 只是三个特殊的文件（即都是 os.File 的实例），自然也实现了 io.Reader 和 io.Writer。

目前，Go 文档中还没法直接列出实现了某个接口的所有类型。不过，我们可以通过查看标准库文档，列出实现了 io.Reader 或 io.Writer 接口的类型（导出的类型）：（注：godoc 命令支持额外参数 -analysis ，能列出都有哪些类型实现了某个接口，相关参考 `godoc -h` 或 [Static analysis features of godoc](https://golang.org/lib/godoc/analysis/help.html)。另外，我做了一个官网镜像，能查看接口所有的实现类型，地址：http://docs.studygolang.com。

- os.File 同时实现了 io.Reader 和 io.Writer
- strings.Reader 实现了 io.Reader
- bufio.Reader/Writer 分别实现了 io.Reader 和 io.Writer
- bytes.Buffer 同时实现了 io.Reader 和 io.Writer
- bytes.Reader 实现了 io.Reader
- compress/gzip.Reader/Writer 分别实现了 io.Reader 和 io.Writer
- crypto/cipher.StreamReader/StreamWriter 分别实现了 io.Reader 和 io.Writer
- crypto/tls.Conn 同时实现了 io.Reader 和 io.Writer
- encoding/csv.Reader/Writer 分别实现了 io.Reader 和 io.Writer
- mime/multipart.Part 实现了 io.Reader
- net/conn 分别实现了 io.Reader 和 io.Writer(Conn接口定义了Read/Write)

除此之外，io 包本身也有这两个接口的实现类型。如：

	实现了 Reader 的类型：LimitedReader、PipeReader、SectionReader
	实现了 Writer 的类型：PipeWriter

以上类型中，常用的类型有：os.File、strings.Reader、bufio.Reader/Writer、bytes.Buffer、bytes.Reader

**小贴士**

从接口名称很容易猜到，一般地， Go 中接口的命名约定：接口名以 er 结尾。注意，这里并非强行要求，你完全可以不以 er 结尾。标准库中有些接口也不是以 er 结尾的。

## ReaderAt 和 WriterAt 接口 ##

**ReaderAt 接口**的定义如下：

```go
type ReaderAt interface {
    ReadAt(p []byte, off int64) (n int, err error)
}
```

官方文档中关于该接口方法的说明：

> ReadAt 从基本输入源的偏移量 off 处开始，将 len(p) 个字节读取到 p 中。它返回读取的字节数 n（0 <= n <= len(p)）以及任何遇到的错误。

> 当 ReadAt 返回的 n < len(p) 时，它就会返回一个 非nil 的错误来解释 为什么没有返回更多的字节。在这一点上，ReadAt 比 Read 更严格。

> 即使 ReadAt 返回的 n < len(p)，它也会在调用过程中使用 p 的全部作为暂存空间。若一些数据可用但不到 len(p) 字节，ReadAt 就会阻塞直到所有数据都可用或产生一个错误。 在这一点上 ReadAt 不同于 Read。

> 若 n = len(p) 个字节在输入源的的结尾处由 ReadAt 返回，那么这时 err == EOF 或者 err == nil。

> 若 ReadAt 按查找偏移量从输入源读取，ReadAt 应当既不影响基本查找偏移量也不被它所影响。

> ReadAt 的客户端可对相同的输入源并行执行 ReadAt 调用。

可见，ReaderAt 接口使得可以从指定偏移量处开始读取数据。

简单示例代码如下：

```go
reader := strings.NewReader("Go语言中文网")
p := make([]byte, 6)
n, err := reader.ReadAt(p, 2)
if err != nil {
    panic(err)
}
fmt.Printf("%s, %d\n", p, n)
```

输出：

	语言, 6

**WriterAt 接口**的定义如下：

```go
type WriterAt interface {
    WriteAt(p []byte, off int64) (n int, err error)
}
```

官方文档中关于该接口方法的说明：

> WriteAt 从 p 中将 len(p) 个字节写入到偏移量 off 处的基本数据流中。它返回从 p 中被写入的字节数 n（0 <= n <= len(p)）以及任何遇到的引起写入提前停止的错误。若 WriteAt 返回的 n < len(p)，它就必须返回一个 非nil 的错误。

> 若 WriteAt 按查找偏移量写入到目标中，WriteAt 应当既不影响基本查找偏移量也不被它所影响。

> 若区域没有重叠，WriteAt 的客户端可对相同的目标并行执行 WriteAt 调用。

我们可以通过该接口将数据写入数据流的特定偏移量之后。

通过简单示例来演示 WriteAt 方法的使用（os.File 实现了 WriterAt 接口）：

```go
file, err := os.Create("writeAt.txt")
if err != nil {
    panic(err)
}
defer file.Close()
file.WriteString("Golang中文社区——这里是多余")
n, err := file.WriteAt([]byte("Go语言中文网"), 24)
if err != nil {
    panic(err)
}
fmt.Println(n)
```

打开文件 WriteAt.txt，内容是：`Golang中文社区——Go语言中文网`。

分析：

`file.WriteString("Golang中文社区——这里是多余")` 往文件中写入 `Golang中文社区——这里是多余`，之后 `file.WriteAt([]byte("Go语言中文网"), 24)` 在文件流的 offset=24 处写入 `Go语言中文网`（会覆盖该位置的内容）。

## ReaderFrom 和 WriterTo 接口 ##

**ReaderFrom** 的定义如下：

```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```

官方文档中关于该接口方法的说明：

> ReadFrom 从 r 中读取数据，直到 EOF 或发生错误。其返回值 n 为读取的字节数。除 io.EOF 之外，在读取过程中遇到的任何错误也将被返回。

> 如果 ReaderFrom 可用，Copy 函数就会使用它。

注意：ReadFrom 方法不会返回 err == EOF。

下面的例子简单的实现将文件中的数据全部读取（显示在标准输出）：

```go
file, err := os.Open("writeAt.txt")
if err != nil {
    panic(err)
}
defer file.Close()
writer := bufio.NewWriter(os.Stdout)
writer.ReadFrom(file)
writer.Flush()
```

当然，我们可以通过 ioutil 包的 ReadFile 函数获取文件全部内容。其实，跟踪一下 ioutil.ReadFile 的源码，会发现其实也是通过 ReadFrom 方法实现（用的是 bytes.Buffer，它实现了 ReaderFrom 接口）。

如果不通过 ReadFrom 接口来做这件事，而是使用 io.Reader 接口，我们有两种思路：

1. 先获取文件的大小（File 的 Stat 方法），之后定义一个该大小的 []byte，通过 Read 一次性读取
2. 定义一个小的 []byte，不断的调用 Read 方法直到遇到 EOF，将所有读取到的 []byte 连接到一起

```go
// 方法１：
package main

import (
	"fmt"
	"os"
)

func main() {

	file, err := os.Open("writeAt.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()
	stat, err := file.Stat()
	if err != nil {
		fmt.Println(err)
	}
	size := stat.Size()

	a := make([]byte, size)
	file.Read(a)
	fmt.Println(string(a))
}
// 运行结果： Golang中文社区——Go语言学习园地

// 方法２：
package main

import (
    "fmt"
    "os"
)

func main() {

    file, err := os.Open("writeAt.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()
    a := make([]byte, 5)
    var b []byte
    for n, err := file.Read(a); err == nil; n, err = file.Read(a) {
        b = append(b, a[:n]...)
    }
    fmt.Println(string(b))
}
// 运行结果： Golang中文社区——Go语言学习园地
```

**提示**

通过查看 bufio.Writer 或 strings.Buffer 类型的 ReadFrom 方法实现，会发现，其实它们的实现和上面说的第 2 种思路类似。

**WriterTo**的定义如下：

```go
type WriterTo interface {
    WriteTo(w Writer) (n int64, err error)
}
```

官方文档中关于该接口方法的说明：

> WriteTo 将数据写入 w 中，直到没有数据可写或发生错误。其返回值 n 为写入的字节数。 在写入过程中遇到的任何错误也将被返回。

> 如果 WriterTo 可用，Copy 函数就会使用它。

读者是否发现，其实 ReaderFrom 和 WriterTo 接口的方法接收的参数是 io.Reader 和 io.Writer 类型。根据 io.Reader 和 io.Writer 接口的讲解，对该接口的使用应该可以很好的掌握。

这里只提供简单的一个示例代码：将一段文本输出到标准输出

```go
reader := bytes.NewReader([]byte("Go语言中文网"))
reader.WriteTo(os.Stdout)
```

通过 io.ReaderFrom 和 io.WriterTo 的学习，我们知道，如果这样的需求，可以考虑使用这两个接口：“一次性从某个地方读或写到某个地方去。”

## Seeker 接口 ##

接口定义如下：

```go
type Seeker interface {
    Seek(offset int64, whence int) (ret int64, err error)
}
```

官方文档中关于该接口方法的说明：

> Seek 设置下一次 Read 或 Write 的偏移量为 offset，它的解释取决于 whence： 0 表示相对于文件的起始处，1 表示相对于当前的偏移，而 2 表示相对于其结尾处。 Seek 返回新的偏移量和一个错误，如果有的话。

也就是说，Seek 方法用于设置偏移量的，这样可以从某个特定位置开始操作数据流。听起来和 ReaderAt/WriteAt 接口有些类似，不过 Seeker 接口更灵活，可以更好的控制读写数据流的位置。

简单的示例代码：获取倒数第二个字符（需要考虑 UTF-8 编码，这里的代码只是一个示例）

```go
reader := strings.NewReader("Go语言中文网")
reader.Seek(-6, io.SEEK_END)
r, _, _ := reader.ReadRune()
fmt.Printf("%c\n", r)
```

**小贴士**

whence 的值，在 io 包中定义了相应的常量，应该使用这些常量

```go
const (
  SeekStart   = 0 // seek relative to the origin of the file
  SeekCurrent = 1 // seek relative to the current offset
  SeekEnd     = 2 // seek relative to the end
)
```

而原先 os 包中的常量已经被标注为Deprecated

```go
// Deprecated: Use io.SeekStart, io.SeekCurrent, and io.SeekEnd.
const (
  SEEK_SET int = 0 // seek relative to the origin of the file
  SEEK_CUR int = 1 // seek relative to the current offset
  SEEK_END int = 2 // seek relative to the end
)
```

## Closer接口 ##

接口定义如下：

```go
type Closer interface {
    Close() error
}
```

该接口比较简单，只有一个 Close() 方法，用于关闭数据流。

文件 (os.File)、归档（压缩包）、数据库连接、Socket 等需要手动关闭的资源都实现了 Closer 接口。

实际编程中，经常将 Close 方法的调用放在 defer 语句中。

**小提示**

初学者容易写出这样的代码：

```go
file, err := os.Open("studygolang.txt")
defer file.Close()
if err != nil {
	...
}
```

当文件 studygolang.txt 不存在或找不到时，file.Close() 会 panic，因为 file 是 nil。因此，应该将 defer file.Close() 放在错误检查之后。

经过 [issue40](https://github.com/polaris1119/The-Golang-Standard-Library-by-Example/issues/40) 提醒，查看源码：

```go
func (f *File) Close() error {
	if f == nil {
		return ErrInvalid
	}
	return f.file.close()
}
```
可见并不会 panic，但在 Close 之前校验错误是个好习惯！


# 1.2 ioutil — 方便的IO操作函数集 #

虽然 io 包提供了不少类型、方法和函数，但有时候使用起来不是那么方便。比如读取一个文件中的所有内容。为此，标准库中提供了一些常用、方便的IO操作函数。

说明：这些函数使用都相对简单，一般就不举例子了。

## NopCloser 函数 ##

有时候我们需要传递一个io.ReadCloser的实例，而我们现在有一个io.Reader的实例，比如：strings.Reader，这个时候NopCloser就派上用场了。它包装一个io.Reader，返回一个io.ReadCloser，而相应的Close方法啥也不做，只是返回nil。

比如，在标准库net/http包中的NewRequest，接收一个io.Reader的body，而实际上，Request的Body的类型是io.ReadCloser，因此，代码内部进行了判断，如果传递的io.Reader也实现了io.ReadCloser接口，则转换，否则通过ioutil.NopCloser包装转换一下。相关代码如下：

	rc, ok := body.(io.ReadCloser)
	if !ok && body != nil {
		rc = ioutil.NopCloser(body)
	}

如果没有这个函数，我们得自己实现一个。当然，实现起来很简单，读者可以看看NopCloser的实现。

## ReadAll 函数 ##

很多时候，我们需要一次性读取io.Reader中的数据，通过上一节的讲解，我们知道有很多种实现方式。考虑到读取所有数据的需求比较多，Go提供了ReadAll这个函数，用来从io.Reader中一次读取所有数据。

	func ReadAll(r io.Reader) ([]byte, error)

阅读该函数的源码发现，它是通过bytes.Buffer中的ReadFrom来实现读取所有数据的。

## ReadDir 函数 ##

笔试题：编写程序输出某目录下的所有文件（包括子目录）

是否见过这样的笔试题？

在Go中如何输出目录下的所有文件呢？首先，我们会想到查os包，看File类型是否提供了相关方法（关于os包，后面会讲解）。

其实在ioutil中提供了一个方便的函数：ReadDir，它读取目录并返回排好序的文件和子目录名（[]os.FileInfo）。通过这个方法，我们可以很容易的实现“面试题”。

下面的例子实现了类似Unix中的tree命令，不过它在windows下也运行的很好哦。

	// 未实现-L参数功能
	func main() {
		if len(os.Args) > 1 {
			Tree(os.Args[1], 1, map[int]bool{1:true})
		}
	}

	// 列出dirname目录中的目录树，实现类似Unix中的tree命令
	// curHier 当前层级（dirname为第一层）
	// hierMap 当前层级的上几层是否需要'|'的映射
	func Tree(dirname string, curHier int, hierMap map[int]bool) error {
		dirAbs, err := filepath.Abs(dirname)
		if err != nil {
			return err
		}
		fileInfos, err := ioutil.ReadDir(dirAbs)
		if err != nil {
			return err
		}

		fileNum := len(fileInfos)
		for i, fileInfo := range fileInfos {
			for j := 1; j < curHier; j++ {
				if hierMap[j] {
				    fmt.Print("|")
				} else {
				    fmt.Print(" ")
				}
				fmt.Print(strings.Repeat(" ", 3))
			}

			// map是引用类型，所以新建一个map
			tmpMap := map[int]bool{}
			for k, v := range hierMap {
			    tmpMap[k] = v
			}
			if i+1 == fileNum {
				fmt.Print("`")
				delete(tmpMap, curHier)
			} else {
				fmt.Print("|")
				tmpMap[curHier] = true
			}
			fmt.Print("-- ")
			fmt.Println(fileInfo.Name())
			if fileInfo.IsDir() {
				Tree(filepath.Join(dirAbs, fileInfo.Name()), curHier+1, tmpMap)
			}
		}
		return nil
	}

## ReadFile 和 WriteFile 函数 ##

ReadFile 读取整个文件的内容，在上一节我们自己实现了一个函数读取文件整个内容，由于这种需求很常见，因此Go提供了ReadFile函数，方便使用。ReadFile的是实现和ReadAll类似，不过，ReadFile会先判断文件的大小，给bytes.Buffer一个预定义容量，避免额外分配内存。

WriteFile 函数的签名如下：

	func WriteFile(filename string, data []byte, perm os.FileMode) error

它将data写入filename文件中，当文件不存在时会创建一个（文件权限由perm指定）；否则会先清空文件内容。对于perm参数，我们一般可以指定为：0666，具体含义os包中讲解。

**小提示**

ReadFile 源码中先获取了文件的大小，当大小 < 1e9时，才会用到文件的大小。按源码中注释的说法是FileInfo不会很精确地得到文件大小。
