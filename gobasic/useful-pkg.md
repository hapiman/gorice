* [go-spew](https://github.com/davecgh/go-spew)打印`struct`和`slice`数据结构
* [godotenv](https://github.com/joho/godotenv)将命令行中的命令提前配置在文件中
* [mux](https://github.com/gorilla/mux)常用于路由配置
```go
func makeMuxRouter() http.Handler {
	muxRouter := mux.NewRouter()
	muxRouter.HandleFunc("/", handleGetBlockchain).Methods("GET")
	muxRouter.HandleFunc("/", handleWriteBlock).Methods("POST")
	return muxRouter
}

mux := makeMuxRouter()

s := &http.Server{
  Addr:           ":8080" ,
  Handler:        mux,
  ReadTimeout:    10 * time.Second,
  WriteTimeout:   10 * time.Second,
  MaxHeaderBytes: 1 << 20,
}

if err := s.ListenAndServe(); err != nil {
  return err
}

```
