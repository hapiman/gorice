- [go-humanize](https://github.com/dustin/go-humanize)
主要包含功能： 千分位转化 时间人性化 存储空间易识别

- [godotenv](https://github.com/joho/godotenv)
读取本地的`.env`配置文件

- [go-simplejson](https://godoc.org/github.com/bitly/go-simplejson)
json处理

- [go-spew](https://github.com/davecgh/go-spew)
打印`struct`和`slice`数据结构

- [govalidator](https://github.com/asaskevich/govalidator)
数据验证
- [mux](https://github.com/gorilla/mux)常用于路由配置
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
