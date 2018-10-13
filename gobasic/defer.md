### 分析defer和panic执行顺序
```go
func main() {
    deferCall()
}

func deferCall() {
    defer func() { fmt.Println("打印前") }()
    defer func() { fmt.Println("打印中") }()
    defer func() { fmt.Println("打印后") }()

    panic("触发异常")
}

```
1. `defer`本身遵循先进后出的顺序, 代码抛出的`panic`如果在所有的defer中都不使用`recover`恢复, 则直接退出程序, 但任然打印数据
2. 如果手动使用`os.Exit()`退出, 则`defer`不执行

### 分析defer和return执行顺序
```go
func main() {
    fmt.Println(double1(5))
    fmt.Println(double1(6))
    fmt.Println()
    fmt.Println(double2(5))
    fmt.Println(double2(6))
}

// 匿名返回
// 加倍参数，若结果超过 10 则还原
func double1(v1 int) int {
    var v2 int
    defer func() {
        if v2 > 10 {
            v2 = v1 // v2 不会被修改
        }
    }()

    v2 = v1 * 2
    return v2
}

// 有名返回
func double2(v1 int)(v2 int) {
    // v2 与函数一起被声明，在 defer 中能被修改
    defer func() {
        if v2 > 10 {
          v2 = v1 // v2 被修改
        }
    }()

    v2 = v1 * 2
    return
}
```

注意 return var 会分为三步执行：

return 语句为 var 赋值

匿名返回值函数：先声明，再赋值
有名返回值函数：直接赋值
检查是否存在 defer 语句：逆序执行多条 defer，有名返回函数可能会再次修改 var

真正返回 var 到调用处

10
12

10
6

### 分析defer参数的计算时机
```go
func main() {
    a := 1
    b := 2
    defer add("A", a, add("B", a, b))
    a = 0
    defer add("C", a, add("D", a, b))
    b = 1
}


func add(desc string, a, b int) int {
    sum := a + b
    fmt.Println(desc, a, b, sum)
    return sum
}
```

defer 语句会计算好 func 的参数，再放入执行栈中。
B 1 2 3
D 0 2 2
C 0 2 2
A 1 3 4

### 考察Go的组合
```go
type People struct{}

func (p *People) ShowA() {
    fmt.Println("people showA")
    p.ShowB()
}
func (p *People) ShowB() {
    fmt.Println("people showB")
}

// Teacher 通过嵌入 People 来获取了 ShowA() 和 showB()
type Teacher struct {
    People
}

// Teacher 实现并覆盖了 showB()
func (t *Teacher) ShowB() {
    fmt.Println("teacher showB")
}

func main() {
    t := Teacher{}
    t.ShowB()
    // 调用未覆盖的 showA()，因为它的 receiver 依旧是 People，相当于 People 调用
    t.ShowA()
}
```

```go

package main
import (
	"fmt"
)
type student struct {
	Name string
	Age  int
}
func pase_student() map[string]*student {
	m := make(map[string]*student)
	stus := []student{
		{Name: "zhou", Age: 24},
		{Name: "li", Age: 23},
		{Name: "wang", Age: 22},
	}
	for _, stu := range stus {
		m[stu.Name] = &stu
	}
	return m
}
func main() {
	students := pase_student()
	for k, v := range students {
		fmt.Printf("key=%s,value=%v \n", k, v)
	}
}
```

 因为for遍历时，变量stu指针不变，每次遍历仅进行struct值拷贝，故m[stu.Name]=&stu实际上一致指向同一个指针，最终该指针的值为遍历的最后一个struct的值拷贝。形同如下代码：
```go
var stu student
for _, stu = range stus {
	m[stu.Name] = &stu
}
```
修正方案，取数组中原始值的指针：
```go
for i, _ := range stus {
	stu:=stus[i]
	m[stu.Name] = &stu
}
```

```go
package main
import (
	"fmt"
)
type People interface {
	Speak(string) string
}
type Stduent struct{}
func (stu *Stduent) Speak(think string) (talk string) {
	if think == "bitch" {
		talk = "You are a good boy"
	} else {
		talk = "hi"
	}
	return
}
func main() {
	var peo People = Stduent{}
	think := "bitch"
	fmt.Println(peo.Speak(think))
}
```

编译失败，值类型 Student{} 未实现接口People的方法，不能定义为 People类型。

定义为指针 `var peo People = &Stduent{}`
方法定义在值类型上,指针类型本身是包含值类型的方法。
`func (stu Stduent) Speak(think string) (talk string) { //... }`
