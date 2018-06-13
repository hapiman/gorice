## json处理

使用 gjson处理从数据接口读取的数据
```go
  robots, errBody := ioutil.ReadAll(resp.Body)
  // 数组
  va := gjson.GetBytes(robots, "data.ret_array")
  // value是gjson.Result类型，直接连接处理
  va.ForEach(func(key, value gjson.Result) bool {
    displayName := value.Get("display_name.0").String()
  })
```

### JSON转化数据类型对比
`bool`, for JSON `booleans`
`float64`, for JSON `numbers`
`string`, for JSON `strings`
`[]interface{}`, for JSON `arrays`
`map[string]interface{}`, for JSON `objects`
`nil` for JSON null

数据格式
```sh
// 序列化
func Marshal(v interface{}) ([]byte, error)

// 反序列化
func Unmarshal(data []byte, v interface{}) error
```

### Unmarshal使用
```go
package main

import (
    "encoding/json"
    "fmt"
)
type Animal struct {
    Name  string
    Order string
}
func main() {
    var jsonBlob = []byte(`[
        {"Name": "Platypus", "Order": "Monotremata"},
        {"Name": "Quoll",    "Order": "Dasyuromorphia"}
    ]`)

    var animals []Animal
    err := json.Unmarshal(jsonBlob, &animals)
    if err != nil {
        fmt.Println("error:", err)
    }
    fmt.Printf("%+v", animals)
}
```
输出结果 `[{Name:Platypus Order:Monotremata} {Name:Quoll Order:Dasyuromorphia}]`
可以看出，结构体字段名与 JSON 里的 KEY 一一对应.
例如 JSON 中的 KEY 是 Name，那么怎么找对应的字段呢？
- 首先查找 tag 含有 Name 的可导出的 struct 字段(首字母大写)
- 其次查找字段名是 Name 的导出字段
- 最后查找类似 NAME 或者 NAmE 等这样的除了首字母之外其他大小写不敏感的导出字段

### Marshal使用
```go
package main

import (
    "encoding/json"
    "fmt"
)
type Animal struct {
    Name  string `json:"name"`
    Order string `json:"order"`
}
func main() {
    var animals []Animal
    animals = append(animals, Animal{Name: "Platypus", Order: "Monotremata"})
    animals = append(animals, Animal{Name: "Quoll", Order: "Dasyuromorphia"})

    jsonStr, err := json.Marshal(animals)
    if err != nil {
        fmt.Println("error:", err)
    }

    fmt.Println(string(jsonStr))
}
// 输出结果
// [{"name":"Platypus","order":"Monotremata"},{"name":"Quoll","order":"Dasyuromorphia"}]
```
序列化规则
- tag 是 "-"，表示该字段不会输出到 JSON.
- tag 中带有自定义名称，那么这个自定义名称会出现在 JSON 的字段名中，比如上面小写字母开头的 name.
- tag 中带有`omitempty`选项，那么如果该字段值为空，就不会输出到JSON串中.
- 如果字段类型是 `bool`, `string`, `int`, `int64` 等，而tag中带有"`,string`" 选项，那么该字段在输出到JSON时，会把该字段对应的值转换成JSON字符串.


