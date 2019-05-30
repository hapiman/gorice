### 去掉前端代码
如果只是作`API`接口使用，使用`--no-frontend`去掉前端相关代码

### 通过.sailsrc来控制加载的模块
```js
// false 表示不加载
{
  “hooks”: {
    “sockets”: false,
    “pubsub”: false,
    “grunt”: false,
    “i18n”: false,
    “csrf”: false,
    “views”: false
  }
}
```
### 查看日志,设置是否显示sql日志
```
LOG_QUERIES=true
```
