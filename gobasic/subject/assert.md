## 断言


关于common-ok的说明

1. 直接判断是否是该类型的变量：`value, ok = element.(T)`，这里value就是变量的值，ok是一个bool类型；
element是interface变量，T是断言的类型。

2. 在switch中直接使用`element.(type)`判断

```go
type Element interface{}
type List [] Element

type Person struct {
	name string
	age int
}

//定义了String方法，实现了fmt.Stringer
func (p Person) String() string {
	return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
}

func main() {
	list := make(List, 3)
	list[0] = 1 // an int
	list[1] = "Hello" // a string
	list[2] = Person{"Dennis", 70}

	for index, element := range list {

		// 每个以此判断
		if value, ok := element.(int); ok {
			fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
		} else if value, ok := element.(string); ok {
			fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
		} else if value, ok := element.(Person); ok {
			fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
		} else {
			fmt.Printf("list[%d] is of a different type\n", index)
		}

		// 直接使用switch判断
		switch value := element.(type) {
			case int:
				fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
			case string:
				fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
			case Person:
				fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
			default:
				fmt.Println("list[%d] is of a different type", index)
		}
	}
}
```
使用另外的示例
```go
var f interface{}
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
json.Unmarshal(b, &f)

// 因为json.Unmarshal解析出来的格式为interface{}, 不能够直接使用
// 当前位置使用map[string]interface{}转化
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
