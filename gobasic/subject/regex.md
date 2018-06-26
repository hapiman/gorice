## 常见的正则规范

`.` 匹配除换行符以外的任意字符
`*`重复零次或更多次
`+`重复一次或更多次
`?`	重复零次或一次
`{n}`	重复n次
`{n,}`	重复n次或更多次
`{n,m}`	重复n到m次

`\d` 数字 (相当于 [0-9])
`\D` 非数字 (相当于 [^0-9])
`\s` 空白 (相当于 [\t\n\f\r ])
`\S` 非空白 (相当于[^\t\n\f\r ])
`\w` 单词字符 (相当于 [0-9A-Za-z_])
`\W` 非单词字符 (相当于 [^0-9A-Za-z_])
`[^x]` 匹配除了x以外的任意字符
`[^aeiou]`匹配除了aeiou这几个字母以外的任意字符

### 关于数字
只能输入1个数字：/^\d$/
只能输入n个数字: /^\d{n}$/
至少输入n个数字：/^\d{n,}$/
输入m到n个数字: /^\d{m,n}$/
只能输入数字: /^[0-9]*$/
只能输入某个区间数字: /[12-15$]/
只能输入0和非0打头的数字: /^(0|[1-9][0-9]*)$/

### 关于字母
只能输入英文字符：/^[a-zA-Z]+$/
只能输入数字和英文字符：/^[a-zA-Z0-9]+$/
只能输入英文字符/数字/下划线: /^\w+$/
只能以字母开始，并且还有m到n个英文字符/数字/下划线: /^[a-zA-Z]\w{m,n}$/

关于 `.`在正则中代表任意字符
`a.[0-9]` a后面跟着任意字符和一个数字
`ab{2}` 表示一个字符串有一个a跟着2个b（"abb"）；
`ab{2,}` 表示一个字符串有一个a跟着至少2个b；
`ab{3,5}` 表示一个字符串有一个a跟着3到5个b。

- 分支，使用符号`|`，其实就是`或`的关系
`\d{5}-\d{4}|\d{5}`

- 分组，使用`()`将多个字符组合起来
`(\d{1,3}\.){3}\d{1,3}`代表``(\d{1,3}\.)`作为一个整体重复3次


### 查找满足正则条件的字符串
使用`regexp.MustCompile`和`FindAllString`
```go
text := `Hello 世界！123 Go.`
// 查找连续的小写字母
reg := regexp.MustCompile(`[a-z]+`)
fmt.Printf("%q\n", reg.FindAllString(text, -1))
// ["ello" "o"]

//这个测试一个字符串是否符合一个表达式。
match, _ := regexp.MatchString("p([a-z]+)ch", "peach")
fmt.Println(match)
//上面我们是直接使用字符串，但是对于一些其他的正则任务，你需要使用 Compile 一个优化的 Regexp 结构体。
r, _ := regexp.Compile("p([a-z]+)ch")
//这个结构体有很多方法。这里是类似我们前面看到的一个匹配测试。
fmt.Println(r.MatchString("peach"))
//这是查找匹配字符串的。
fmt.Println(r.FindString("peach punch"))
//这个也是查找第一次匹配的字符串的，但是返回的匹配开始和结束位置索引，而不是匹配的内容。
fmt.Println(r.FindStringIndex("peach punch"))
//Submatch 返回完全匹配和局部匹配的字符串。例如，这里会返回 p([a-z]+)ch 和 `([a-z]+) 的信息。
fmt.Println(r.FindStringSubmatch("peach punch"))
//类似的，这个会返回完全匹配和局部匹配的索引位置。
fmt.Println(r.FindStringSubmatchIndex("peach punch"))
// //带 All 的这个函数返回所有的匹配项，而不仅仅是首次匹配项。例如查找匹配表达式的所有项。
fmt.Println(r.FindAllString("peach punch pinch", -1))
// //All 同样可以对应到上面的所有函数。
fmt.Println(r.FindAllStringSubmatchIndex("peach punch pinch", -1))
// //这个函数提供一个正整数来限制匹配次数。
fmt.Println(r.FindAllString("peach punch pinch", 2))
// //上面的例子中，我们使用了字符串作为参数，并使用了如 MatchString 这样的方法。我们也可以提供 []byte参数并将 String 从函数命中去掉。
fmt.Println(r.Match([]byte("peach")))
// //创建正则表示式常量时，可以使用 Compile 的变体MustCompile 。因为 Compile 返回两个值，不能用语常量。
r = regexp.MustCompile("p([a-z]+)ch")
fmt.Println(r)
// //regexp 包也可以用来替换部分字符串为其他值。
fmt.Println(r.ReplaceAllString("a peach", "<fruit>"))
// //Func 变量允许传递匹配内容到一个给定的函数中，
in := []byte("a peach")
out := r.ReplaceAllFunc(in, bytes.ToUpper)
fmt.Println(string(out))
```


