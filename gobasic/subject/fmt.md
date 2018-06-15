## golang中打印字符串

```go
type point struct {
    x, y int
}

func main() {

    // Go offers several printing "verbs" designed to
    // format general Go values. For example, this prints
    // an instance of our `point` struct.
    p := point{1, 2}
    fmt.Printf("%v\n", p) // {1 2}

    // If the value is a struct, the `%+v` variant will
    // include the struct's field names.
    fmt.Printf("%+v\n", p) // {x:1 y:2}

    // The `%#v` variant prints a Go syntax representation
    // of the value, i.e. the source code snippet that
    // would produce that value.
    fmt.Printf("%#v\n", p) // main.point{x:1, y:2}

    // To print the type of a value, use `%T`.
    fmt.Printf("%T\n", p) // main.point

    // Formatting booleans is straight-forward.
    fmt.Printf("%t\n", true) // true

    // There are many options for formatting integers.
    // Use `%d` for standard, base-10 formatting.
    fmt.Printf("%d\n", 123) // 1233

    // This prints a binary representation.
    fmt.Printf("%b\n", 14) // 转化为二进制 123

    // This prints the character corresponding to the
    // given integer.
    fmt.Printf("%c\n", 33) // 转化为Assic码

    // `%x` provides hex encoding.
    fmt.Printf("%x\n", 456) // 转化为16进制


    // For basic string printing use `%s`.
    fmt.Printf("%s\n", "\"string\"")

    // To double-quote strings as in Go source, use `%q`.
    fmt.Printf("%q\n", "\"string\"")
}
```
