正则表达式构造函数：`new RegExp("pattern"[,"flags"]);`
正则表达式替换变量函数：`stringObj.replace(RegExp, replace Text);`
```js
//下面的例子用来获取url的两个参数，并返回urlRewrite之前的真实Url
var reg=new RegExp("(http://www.qidian.com/BookReader/)(\\d+),(\\d+).aspx","gmi");
var url="http://www.qidian.com/BookReader/1017141,20361055.aspx";

//方式一,最简单常用的方式
var rep=url.replace(reg,"$1ShowBook.aspx?bookId=$2&chapterId=$3");
console.log(rep); // http://www.qidian.com/BookReader/ShowBook.aspx?bookId=1017141&chapterId=20361055

//方式二 ,采用固定参数的回调函数
var rep2=url.replace(reg,function(m,p1,p2,p3){
  console.log('mmmm => ', m)
  return p1+"ShowBook.aspx?bookId="+p3+"&chapterId="+p3
});
alert(rep2);

//方式三，采用非固定参数的回调函数
var rep3=url.replace(reg,function(){var args=arguments; return args[1]+"ShowBook.aspx?bookId="+args[2]+"&chapterId="+args[3];});
alert(rep3);

//方法四
//方式四和方法三很类似, 除了返回替换后的字符串外，还可以单独获取参数
var bookId;
var chapterId;
function capText()
{
    var args=arguments;
    bookId=args[2];
    chapterId=args[3];
    return args[1]+"ShowBook.aspx?bookId="+args[2]+"&chapterId="+args[3];
}

var rep4=url.replace(reg,capText);
alert(rep4);
alert(bookId);
alert(chapterId);

//使用test方法获取分组
var reg3=new RegExp("(http://www.qidian.com/BookReader/)(\\d+),(\\d+).aspx","gmi");
reg3.test("http://www.qidian.com/BookReader/1017141,20361055.aspx");
//获取三个分组
console.log(RegExp.$0)
console.log(RegExp.$1); // http://www.qidian.com/BookReader/
console.log(RegExp.$2); // 1017141
console.log(RegExp.$3); // 20361055
```
