> https://www.jianshu.com/p/9a666b98d7a2
> https://www.jianshu.com/p/ae9b5628d18b
> https://blog.csdn.net/lengyuezuixue/article/details/79094323

```go
// 方式一: 直接使用http.HandleFunc 和 http.ListenAddServe(":port", nil)方式
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("httpserver v1"))
	})
	http.HandleFunc("/bye", sayBye)
	log.Println("Starting v1 server ...")
	log.Fatal(http.ListenAndServe(":1210", nil))
}

func sayBye(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("bye bye ,this is v1 httpServer"))
}
// http.ListenAndServe(":1210", nil) =>使用传入参数构造server对象, 然后执行server.ListenAndServe()
```

```go
// 方式二, 关联使用ServeMux对象, 该对象中实现了ServeHttp方法
func main() {
   mux := http.NewServeMux()
	 mux.Handle("/", &myHandler{})
	 // sayBye函数被HandleFunc(sayBye)之后,转化成为了handler
   mux.HandleFunc("/bye", sayBye)

   log.Println("Starting v2 httpserver")
   log.Fatal(http.ListenAndServe(":1210", mux))
}
type myHandler struct{}

func (*myHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
      w.Write([]byte("this is version 2"))
}
func sayBye(w http.ResponseWriter, r *http.Request) {
     w.Write([]byte("bye bye ,this is v2 httpServer"))
}
```

```go
// 方式三
func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &myHandler{})
	mux.HandleFunc("/bye", sayBye)

	// 不再使用http直接调用ListenAndServer
	// 直接使用http.Server定制端口, 超时及路由控制等其他配置
	server := &http.Server{
		Addr:         ":1210",
		WriteTimeout: time.Second * 3, //设置3秒的写超时
		Handler:      mux,
	}
	log.Println("Starting v3 httpserver")
	log.Fatal(server.ListenAndServe())
}

type myHandler struct{}

func (*myHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("this is version 3"))
}

func sayBye(w http.ResponseWriter, r *http.Request) {
	// 睡眠4秒  上面配置了3秒写超时，所以访问 “/bye“路由会出现没有响应的现象
	time.Sleep(4 * time.Second)
	w.Write([]byte("bye bye ,this is v3 httpServer"))
}
```

### 分析
无论使用哪种方式, 最终都会调用`func (srv *Server) ListenAndServe() error {}`实现服务监听, 逻辑封装使用了`Server`结构体, 最重要的就是使用其中的`Addr`和`Handler`参数, 完成数据的替换, 内部调用了`func (srv *Server) Serve(l net.Listener) error {}`方法, 具体到方法`func (c *conn) serve(ctx context.Context) {}`

``` go
type Server struct {
	Addr    string  // TCP address to listen on, ":http" if empty
	Handler Handler // handler to invoke, http.DefaultServeMux if nil

	// TLSConfig optionally provides a TLS configuration for use
	// by ServeTLS and ListenAndServeTLS. Note that this value is
	// cloned by ServeTLS and ListenAndServeTLS, so it's not
	// possible to modify the configuration with methods like
	// tls.Config.SetSessionTicketKeys. To use
	// SetSessionTicketKeys, use Server.Serve with a TLS Listener
	// instead.
	TLSConfig *tls.Config

	// ReadTimeout is the maximum duration for reading the entire
	// request, including the body.
	//
	// Because ReadTimeout does not let Handlers make per-request
	// decisions on each request body's acceptable deadline or
	// upload rate, most users will prefer to use
	// ReadHeaderTimeout. It is valid to use them both.
	ReadTimeout time.Duration
	// ...
	// ...
	// ...
}

```
