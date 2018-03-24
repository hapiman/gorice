### 中间件

基本格式
``` go
func middleWare(next http.Handler) http.Handler {
  return http.HandleFunc(func(w http.ResponseWriter, r *http.Request) {
    // 该中间件逻辑，实际上调用的时next(w,r)方法
    next.ServeHttp()
  })
}
```

* 如何构造一个`Handler`

1.使用`http.HandlerFunc`函数，实现中间件接收`handler`返回`handler`.

``` go
  http.HandlerFunc(func final(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing finalHandler")
    w.Write([]byte("OK"))
  })
```
2.实现`ServeHTTP`的`struct`或者`interface`, 因为`http.Handler`是一个只有`ServeHTTP`方法的接口；
然后将该结构或者对象传入`http.ListenAndServe`第二个参数中。

``` go
  type SingleHost struct {
    handler     http.Handler
    allowedHost string
  }

  func NewSingleHost(handler http.Handler, allowedHost string) *SingleHost {
      return &SingleHost{handler: handler, allowedHost: allowedHost}
  }

  func (s *SingleHost) ServeHTTP(w http.ResponseWriter, r *http.Request) {
      host := r.Host
      if host == s.allowedHost {
          s.handler.ServeHTTP(w, r)
      } else {
          w.WriteHeader(403)
      }
  }

  func main () {

    singleHosted := NewSingleHost(myHandler, "example.com")
    // 第一个参数端口，第二个参数是handler
    http.ListenAndServe(":8080", singleHosted)
  }
```
3.直接使用`mux := http.NewServeMux()`，使用`mux`取代默认的`defaultServeMux`处理
``` go
type textHandler struct {
    responseText string
}

func (th *textHandler) ServeHTTP(w http.ResponseWriter, r *http.Request){
    fmt.Fprintf(w, th.responseText)
}

type indexHandler struct {}

func (ih *indexHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")

    html := `<doctype html>
        <html>
        <head>
          <title>Hello World</title>
        </head>
        <body>
        <p>
          <a href="/welcome">Welcome</a> |  <a href="/message">Message</a>
        </p>
        </body>
</html>`
    fmt.Fprintln(w, html)
}

func test1(w http.ResponseWriter, r *http.Request){
  fmt.Fprintln(w, "hello world")
}

func test2(w http.ResponseWriter, r *http.Request){
  fmt.Fprintln(w, "hello world")
}

func main() {
    mux := http.NewServeMux()

    mux.Handle("/", &indexHandler{})

    // 处理http请求
    mux.Handle("/test1", http.HandleFunc(test1))
    mux.HandleFunc("/test2", http.Handle(test2))

    thWelcome := &textHandler{"TextHandler !"}
    mux.Handle("/text",thWelcome)

    http.ListenAndServe(":8000", mux)
}
```



``` go
package main

import (
  "log"
  "net/http"
)

func middlewareOne(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing middlewareOne")
    next.ServeHTTP(w, r)
    log.Println("Executing middlewareOne again")
  })
}

func middlewareTwo(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    log.Println("Executing middlewareTwo")
    if r.URL.Path != "/" {
      return
    }
    next.ServeHTTP(w, r)
    log.Println("Executing middlewareTwo again")
  })
}

func final(w http.ResponseWriter, r *http.Request) {
  log.Println("Executing finalHandler")
  w.Write([]byte("OK"))
}

func main() {
  finalHandler := http.HandlerFunc(final)

  http.Handle("/", middlewareOne(middlewareTwo(finalHandler)))
  http.ListenAndServe(":3000", nil)
}
```

* 自定义Server
``` go

// Handle使用默认方式
func main(){
    http.HandleFunc("/", index)

    server := &http.Server{
        Addr: ":8000",
        ReadTimeout: 60 * time.Second,
        WriteTimeout: 60 * time.Second,
    }
    server.ListenAndServe()
}

  // 使用mux拥有的handler
  mux := http.NewServeMux()
  mux.HandleFunc("/", index)

  server := &http.Server{
    Addr: ":8000",
    ReadTimeout: 60 * time.Second,
    WriteTimeout: 60 * time.Second,
    Handler: mux,
  }
  server.ListenAndServe()
```
