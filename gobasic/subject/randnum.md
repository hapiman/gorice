## 生成随机数的两种方式

### 使用 math/rand
Go的math/rand包提供了生成随机数的API，重要的API如下：
```go
// 该函数设置随机种子
// 若不调用此函数设置随机种子，则默认的种子值为1，由于随机算法是固定的，
// 如果每次都以1作为随机种子开始产生随机数，则结果都是一样的，因此一般
// 都需要调用此函数来设置随机种子，通常的做法是以当前时间作为随机种子
// 以保证每次随机种子都不同，从而产生的随机数也不通
// 该函数协程安全
func Seed(seed int64)

// 以下函数用来生成相应数据类型的随机数，带n的版本则生成[0,n)的随机数。
// 注意生成的随机数都是非负数
func Float32() float32
func Float64() float64
func Int() int
func Int31() int32  // 注意该函数只返回int32表示范围内的非负数，位数为31，因此该函数叫做Int31
func Int31n(n int32) int32
func Int63() int64
func Int63n(n int64) int64
func Intn(n int) int
func Uint32() uint32
func Uint64() uint64

// 另外，rand包还提供了一个类，接口和上面的大致相同：
type Rand struct {
    // ...
}

// 创建一个以seed为种子的源，注意该源不是协程安全的
func NewSource(seed int64) Source
// 以src为源创建随机对象
func New(src Source) *Rand
// 设置或重置种子，注意该函数不是协程安全的
func (r *Rand) Seed(seed int64)
// 下面的函数和全局版本的函数功能一样
func (r *Rand) Float32() float32
func (r *Rand) Float64() float64
func (r *Rand) Int() int
func (r *Rand) Int31() int32
func (r *Rand) Int31n(n int32) int32
func (r *Rand) Int63() int64
func (r *Rand) Int63n(n int64) int64
func (r *Rand) Intn(n int) int
func (r *Rand) Uint32() uint32
func (r *Rand) Uint64() uint64
```

以当前的时间作为随机种子
```go
// 返回当前时间
func Now() Time

// 为了将Time类型转换为int64类型以作为随机种子
// 可以使用如下两个函数：

// 返回从1970年1月1日到t的秒数
func (t Time) Unix() int64
// 返回从1970年1月1日到t的纳秒数
func (t Time) UnixNano() int64
```
相关例子
```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {

    //
    // 使用全局函数 ==========================================
    //

    rand.Seed(time.Now().Unix())

    fmt.Println(rand.Int())       // int随机值，返回值为int
    fmt.Println(rand.Intn(100))   // [0,100)的随机值，返回值为int
    fmt.Println(rand.Int31())     // 31位int随机值，返回值为int32
    fmt.Println(rand.Int31n(100)) // [0,100)的随机值，返回值为int32
    fmt.Println(rand.Float32())   // 32位float随机值，返回值为float32
    fmt.Println(rand.Float64())   // 64位float随机值，返回值为float64

    // 如果要产生负数到正数的随机值，只需要将生成的随机数减去相应数值即可
    fmt.Println(rand.Intn(100) - 50) // [-50, 50)的随机值

    //
    // 使用Rand对象 =======================================
    //
    r := rand.New(rand.NewSource(time.Now().Unix()))

    fmt.Println(r.Int())       // int随机值，返回值为int
    fmt.Println(r.Intn(100))   // [0,100)的随机值，返回值为int
    fmt.Println(r.Int31())     // 31位int随机值，返回值为int32
    fmt.Println(r.Int31n(100)) // [0,100)的随机值，返回值为int32
    fmt.Println(r.Float32())   // 32位float随机值，返回值为float32
    fmt.Println(r.Float64())   // 64位float随机值，返回值为float64

    // 如果要产生负数到正数的随机值，只需要将生成的随机数减去相应数值即可
    fmt.Println(r.Intn(100) - 50) // [-50, 50)的随机值
}
```

### 使用 crypto/rand
这个包使用了系统的部分功能，相比`math/rand`更加的安全，但是稍微慢点

方法1 `func Read(b []byte) (n int, err error)`
```go
import (
    "crypto/rand"
    "fmt"
)

func main() {
    b := make([]byte, 20)
    fmt.Println(b)       //

    _, err := rand.Read(b)
    if err != nil {
        fmt.Println(err.Error())
    }

    fmt.Println(b)
}

// [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
// [68 221 145 73 115 224 13 110 218 130 19 139 38 170 145 58 251 188 126 197]
```

方法2 `func Int(rand io.Reader, max *big.Int) (n *big.Int, err error)`

```go
s  := "i am a good boy"
n, err := rand.Int(rand.Reader, big.NewInt(int64(len(s))))
```
