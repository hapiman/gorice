
### reflect基本使用

- 参考文档
> 1. https://www.cnblogs.com/golove/p/5909541.html
> 2. 手把手学习`reflect`, https://zhuanlan.zhihu.com/p/44350097


`reflect`用于在运行时判断数据的类型和值, 关注变量的实际类型和实际值, 主要是基于`reflect.TypeOf()`和`reflect.ValueOf()`使用

使用`Kind()`返回分类的值类型：
基础类型 bool、string，
数字类型，
聚合类型array、struct，
引用类型 chan、ptr、func、slice、map，
接口类型 interface，
无任何值的 Invalid 类型：

使用 `reflect.TypeOf(v)` 可在运行时动态的获取接口变量的类型：
使用 `reflect.ValueOf(v)` 获取到变量的值，它是只读的。若想修改 v 的值，需使用 `reflect.ValueOf(&v)`

```go
fmt.Printf("%v\n", reflect.TypeOf(s).Kind())		// slice
fmt.Printf("%v\n", reflect.TypeOf(me.GetIntro).Kind())	// func
```
使用 `Interface()` 能将变量的值以 `interface{}` 类型返回
可使用 `Elem()` 来获取它们指向或存储的元素值:

```go
func main() {
	str := "old"
	strVal := reflect.ValueOf(str)		// 只读，不可修改
	fmt.Println(strVal.Interface())		// "old"
	// strVal.Elem().SetString("new")	// panic: reflect: call of reflect.Value.Elem on string Value

	strPtrVal := reflect.ValueOf(&str)	// 取址，可修改
	// strPtrVal.Elem().SetInt(1)		// 不能设置不同类型的值 // panic: reflect: call of reflect.Value.SetInt on string Value
	strPtrVal.Elem().SetString("new")	// Set 指定类型
	fmt.Println(str) 				// "new"
	strPtrVal.Elem().Set(reflect.ValueOf("newNew"))	// Set Value 类型
	fmt.Println(str) 				// "newNew"
}
```

**获取、修改未知的 struct 类型字段的值、调用方法**
如果变量是 struct，可使用 NumField() 返回字段数量，再遍历获取、修改字段的值：
```go
type User struct {
	FirstName string `tag_name:"front"`
	LastName  string `tag_name:"back"`
	Age       int    `tag_name:"young"`
}

func main() {
	u := User{"wu", "Yin", 20}
	represent(u)

	uType := reflect.TypeOf(u)
  newU := reflect.New(uType) // 创建已知类型的变量
  fmt.Printf("init => %+v", newU.Elem()) // 打印初始值
	newU.Elem().Field(0).SetString("Frank")
	newU.Elem().Field(1).SetString("Underwood")
	newU.Elem().Field(2).SetInt(50)

	newUser := newU.Elem().Interface().(User)	// newUser 是 User 类型，断言不会 panic
	fmt.Printf("%+v", newUser)
}

func represent(i interface{}) {
	t := reflect.TypeOf(i)
	v := reflect.ValueOf(i)


  // 使用 NumField() 来遍历探测结构体的字段值
  // 分别取类型Type和值Value
	for i := 0; i < t.NumField(); i++ {
		fieldVal := v.Field(i)		// 注意调用者是 reflect.Value
		fieldType := t.Field(i)		// 注意调用者是 reflect.Type
		fieldTag := fieldType.Tag

		fmt.Printf("Field Name: %s\t Field Value: %v \tTag Value: %s\t\n",
			fieldType.Name,
			fieldVal,
			fieldTag.Get("tag_name"))
	}

	// 使用 NumMethod() 来遍历探测结构体的方法
	for i := 0; i < t.NumMethod(); i++ {
		m := t.Method(i)
		fmt.Printf("%s :%v\n", m.Name, m.Type)
	}

	// 通过方法名来调用
	// 如果方法不存在，则 panic: reflect: call of reflect.Value.Call on zero Value
	m := v.MethodByName("Intro")

	// args := make([]reflect.Valuye, 0)	// 方法无参数时

	// 有参数时，参数类型是 reflect.Value
	args := []reflect.Value{reflect.ValueOf("Beijing"), reflect.ValueOf("Xian")}
	m.Call(args)
}

func (u User) Intro(workLoc string, studyLoc string) {
	fmt.Printf("My name is %s%s, age %d, working in %s and study in %s\n",
		u.FirstName, u.LastName, u.Age, workLoc, studyLoc)
}
```
输出
```
Field Name: FirstName	 Field Value: wu 	Tag Value: front
Field Name: LastName	 Field Value: Yin 	Tag Value: back
Field Name: Age	 	 Field Value: 20 	Tag Value: young
Intro :func(main.User, string, string)
My name is wuYin, age 20, working in Beijing and study in Xian
{FirstName:Frank LastName:Underwood Age:50}
```


`reflect.TypeOf(x)`获取类型
`v := reflect.ValueOf(x)`获取值
`v.Type()`获取直接数据类型
`v.Kind()`获取基础数据类型
`v.Interface()`获取值
`v.Elem()`来获取它们指向或存储的元素值:

``` go
type myInt int
var i myInt
t := reflect.TypeOf(i) // myInt
k := t.Kind() // int
```


```go
var x float64 = 3.4
fmt.Println("type: ", reflect.TypeOf(x))
v := reflect.ValueOf(x)
fmt.Println("value:", v)
fmt.Println("type:", v.Type())
fmt.Println("kind:", v.Kind())
fmt.Println("value:", v.Float())
fmt.Println("interface =>", v.Interface())
fmt.Printf("value is %5.2e\n", v.Interface())
y := v.Interface().(float64) // 使用断言判断类型
fmt.Println(y)
// 使用 `reflect.ValueOf(v)` 获取到变量的值，它是只读的。若想修改 v 的值，需使用 `reflect.ValueOf(&v)`
fmt.Println("settability of v:", v.CanSet())
v = reflect.ValueOf(&x)
v = v.Elem()
fmt.Println("settability of v:", v.CanSet())
v.SetFloat(3.1415)
fmt.Println(v)
fmt.Println(reflect.ValueOf(v))
fmt.Println(v.Interface())

t := T{123, "skkkk"}
s := reflect.ValueOf(&t).Elem()
// 等价于 typeOfT := reflect.TypeOf(t)
typeOfT := s.Type()
fmt.Println("typeOfT ", typeOfT)
for i := 0; i < s.NumField(); i++ {
  f := s.Field(i)
  fmt.Println("f ", f)
  fmt.Printf("%d: %s %s = %v\n", i,
    typeOfT.Field(i).Name, f.Type(), f.Interface())
}
s.Field(0).SetInt(77)
s.Field(1).SetString("xxx")
fmt.Println("t is now", t)
```
