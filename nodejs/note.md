字符串和`base64`转化

```js
const sStr = 'This is test'
const sStrBase64 = Buffer.from(sStr).toString('base64')
const sStr2 = Buffer.from(sStrBase64, 'base64')
```
