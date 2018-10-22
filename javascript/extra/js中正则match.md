### 截取字符串
主要是使用`match`来处理
```js
var str = "aaabbbcccdddeeefff";
var strResult = str.match(/aaa(\S*)fff/);
console.log(strResult);
/*
// strResult是一个类数组,
["aaabbbcccdddeeefff", "bbbcccdddeee", index: 0, input: "aaabbbcccdddeeefff", groups: undefined]
["原字符串",            "截取出来的字符串",  "位置编码",  "输入",                   "组"]
*/
```
