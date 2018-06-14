> [Golang 处理 Json（一）：编码](http://www.cnblogs.com/52php/p/6512581.html)
> [Golang 处理 Json（二）：解码](http://www.cnblogs.com/52php/p/6518728.html)

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
`bool`, for JSON `bool`
`float64`, for JSON `number`
`int64`, for JSON `number`
`string`, for JSON `string`
`[]interface{}`, for JSON `array`
`map[string]interface{}`, for JSON `object`
`nil` for JSON `null`



### 数据格式
```sh
// 序列化
func Marshal(v interface{}) ([]byte, error)

// 反序列化
func Unmarshal(data []byte, v interface{}) error
```

### Encode
```go
func NewEncoder(w io.Writer) *Encoder

file, _ := os.Create("json.txt")
enc := json.NewEncoder(file)
err := enc.Encode(&v)
// 数据结构v会以json格式写入json.txt文件。
```

### Decode
```go
func NewDecoder(r io.Reader) *Decoder

fp, _ os.Open("json.txt")
dec := json.NewDecoder(fp)
for {
    var V v
    err := dec.Decode(&v)
    if err != nil {
        break
    }
    //use v
}
// v是一个数据结构空间，decoder会将文件中的json格式按照v的定义转化，存在v中。
```

> 1. encoder与decoder像是在writer外面封装了一层。会根据指定的数据结构的格式进行读写。如果文件中的json格式与指定的数据结构的格式不一致会出现error。
> 2. 在decoder的过程中，用一个for｛｝不停的读文件，直到出现error，代表文件结束。在for｛｝中，每次都要申请一个新的空间，存放从文件中读取一行数据。


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

针对第四点
```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
    Money    float64 `json:"money,string"` // money字段必须已字符串的格式
}

var jsonString string = `{
    "email": "phpgo@163.com",
    "password" : "123456",
    "money" : "100.5"
}` // money不能没有 双引号，否则会报错

func main() {

    account := Account{}
    err := json.Unmarshal([]byte(jsonString), &account)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("%+v\n", account)
}
```

### 如何动态的解析JSON
理想的情况往往都存在理想的愿望之中，很多 json 非但格式不确定，有的还可能是动态数据类型。
例如通常登录的时候，往往既可以使用手机号做用户名，也可以使用邮件做用户名，客户端传的 json 可以是字串，也可以是数字。此时服务端解析就需要技巧了。

#### Decode
前面我们使用了简单的方法 Unmarshal 直接解析 json 字串，下面我们使用更底层的方法 NewDecode 和 Decode 方法。
```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "log"
    "strings"
)

type User struct {
    UserName string `json:"username"`
    Password string `json:"password"`
}

var jsonString string = `{
    "username": "phpgo@163.com",
    "password": "123"
}`

// 传如Reader的对象
func Decode(r io.Reader) (u *User, err error) {
    u = new(User)
    err = json.NewDecoder(r).Decode(u)
    if err != nil {
        return
    }
    return
}

func main() {
    // 对于字串，可是使用 strings.NewReader 方法，让字串变成一个 Stream 对象。
    user, err := Decode(strings.NewReader(jsonString))
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%#v\n", user)
}
```

如果对于某些字段类型可能出现动态变化的情况,需要使用`switch t := u.(type)`的方式断言改变
```go
type User struct {
    UserName interface{} `json:"username"`
    Password string `json:"password"`
}

var jsonString string = `{
    "username": 15899758289,
    "password": "123"
}`

func Decode(r io.Reader) (u *User, err error) {
    u = new(User)
    if err = json.NewDecoder(r).Decode(u); err != nil {
        return
    }

    switch t := u.UserName.(type) {
    case string:
        u.UserName = t
    case float64:
        u.UserName = int64(t)
    }
    return
}

func main() {
    user, err := Decode(strings.NewReader(jsonString))
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%#v\n", user)
}
```

另外一个示例
```go
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

type Place struct {
    City    string `json:"city"`
    Country string `json:"country"`
}

func decode(jsonStr []byte) (persons []Person, places []Place) {
    var data map[string][]map[string]interface{}

    err := json.Unmarshal(jsonStr, &data)
    if err != nil {
        fmt.Println(err)
        return
    }

    for i := range data["things"] {
        item := data["things"][i]
        if item["name"] != nil {
            persons = addPerson(persons, item)
        } else {
            places = addPlace(places, item)
        }
    }

    return
}

func addPerson(persons []Person, item map[string]interface{}) []Person {
    name := item["name"].(string)
    age := item["age"].(float64)
    person := Person{name, int(age)}
    persons = append(persons, person)

    return persons
}

func addPlace(places []Place, item map[string]interface{}) []Place {
    city := item["city"].(string)
    country := item["country"].(string)
    place := Place{City: city, Country: country}
    places = append(places, place)

    return places
}

var jsonString string = `{
    "things": [
        {
            "name": "Alice",
            "age": 37
        },
        {
            "city": "Ipoh",
            "country": "Malaysia"
        },
        {
            "name": "Bob",
            "age": 36
        },
        {
            "city": "Northampton",
            "country": "England"
        }
    ]
}`

func main() {
    personA, placeA := decode([]byte(jsonString))

    fmt.Printf("%+v\n", personA)
    fmt.Printf("%+v\n", placeA)

    // [{Name:Alice Age:37} {Name:Bob Age:36}]
    // [{City:Ipoh Country:Malaysia} {City:Northampton Country:England}]
}
```

使用`json.RawMessage`来解析
```go
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

type Place struct {
    City    string `json:"city"`
    Country string `json:"country"`
}

func addPerson(item json.RawMessage, persons []Person) ([]Person) {
    person := Person{}
    if err := json.Unmarshal(item, &person); err != nil {
        fmt.Println(err)
    } else {
        if person != *new(Person) {
            persons = append(persons, person)
        }
    }

    return persons
}

func addPlace(item json.RawMessage, places []Place) ([]Place) {
    place := Place{}
    if err := json.Unmarshal(item, &place); err != nil {
        fmt.Println(err)
    } else {
        if place != *new(Place) {
            places = append(places, place)
        }
    }

    return places
}

func decode(jsonStr []byte) (persons []Person, places []Place) {
    var data map[string][]json.RawMessage

    err := json.Unmarshal(jsonStr, &data)
    if err != nil {
        fmt.Println(err)
        return
    }

    for _, item := range data["things"] {
        persons = addPerson(item, persons)
        places = addPlace(item, places)
    }

    return
}

var jsonString string = `{
    "things": [
        {
            "name": "Alice",
            "age": 37
        },
        {
            "city": "Ipoh",
            "country": "Malaysia"
        },
        {
            "name": "Bob",
            "age": 36
        },
        {
            "city": "Northampton",
            "country": "England"
        }
    ]
}`

func main() {
    personA, placeA := decode([]byte(jsonString))

    fmt.Printf("%+v\n", personA)
    fmt.Printf("%+v\n", placeA)

    // [{Name:Alice Age:37} {Name:Bob Age:36}]
    // [{City:Ipoh Country:Malaysia} {City:Northampton Country:England}]
}
```

补充示例
```go
var f interface{}
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
json.Unmarshal(b, &f)

for k, v := range f.(map[string]interface{}) {
  switch vv := v.(type) {
  case string:
    fmt.Println(k, "is string", vv)
  case int:
    fmt.Println(k, "is int ", vv)
  case float64:
    fmt.Println(k, "is float64 ", vv)
  case []interface{}:
    fmt.Println(k, "is array:")
    for i, j := range vv {
      fmt.Println(i, j)
    }
  }
}
```
