关于`scan`中的`filter`使用,

```sh
# 行键以什么开始 ROWPREFIXFILTER
scan 'test', {ROWPREFIXFILTER => 'row2'}
# 查询的列簇:列名
COLUMNS => ['cf2:c2']

# 查询条数
LIMIT => 10

# cell值为
scan 'test1', FILTER=>"ValueFilter(=,'binary:sku188')"
```

创建表
create 'test1', 'lf', 'sf'
lf: column family of LONG values (binary value)
-- sf: column family of STRING values

导入数据
put 'test1', 'user1|ts1', 'sf:c1', 'sku1'
put 'test1', 'user1|ts2', 'sf:c1', 'sku188'
put 'test1', 'user1|ts3', 'sf:s1', 'sku123'

put 'test1', 'user2|ts4', 'sf:c1', 'sku2'
put 'test1', 'user2|ts5', 'sf:c2', 'sku288'
put 'test1', 'user2|ts6', 'sf:s1', 'sku222'
一个用户（userX），在什么时间（tsX），作为rowkey

对什么产品（value：skuXXX），做了什么操作作为列名，比如，c1: click from homepage; c2: click from ad; s1: search from homepage; b1: buy

查询案例

谁的值=sku188

scan 'test1', FILTER=>"ValueFilter(=,'binary:sku188')"

ROW                          COLUMN+CELL
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188

谁的值包含88

scan 'test1', FILTER=>"ValueFilter(=,'substring:88')"

ROW                          COLUMN+CELL
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188
 user2|ts5                   column=sf:c2, timestamp=1409122355030, value=sku288


通过广告点击进来的(column为c2)值包含88的用户

scan 'test1', FILTER=>"ColumnPrefixFilter('c2') AND ValueFilter(=,'substring:88')"

ROW                          COLUMN+CELL
 user2|ts5                   column=sf:c2, timestamp=1409122355030, value=sku288
通过搜索进来的(column为s)值包含123或者222的用户

scan 'test1', FILTER=>"ColumnPrefixFilter('s') AND ( ValueFilter(=,'substring:123') OR ValueFilter(=,'substring:222') )"

ROW                          COLUMN+CELL
 user1|ts3                   column=sf:s1, timestamp=1409122354954, value=sku123
 user2|ts6                   column=sf:s1, timestamp=1409122355970, value=sku222

rowkey为user1开头的

scan 'test1', FILTER => "PrefixFilter ('user1')"

ROW                          COLUMN+CELL
 user1|ts1                   column=sf:c1, timestamp=1409122354868, value=sku1
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188
 user1|ts3                   column=sf:s1, timestamp=1409122354954, value=sku123

FirstKeyOnlyFilter: 一个rowkey可以有多个version,同一个rowkey的同一个column也会有多个的值, 只拿出key中的第一个column的第一个version
KeyOnlyFilter: 只要key,不要value
`scan 'test1', FILTER=>"FirstKeyOnlyFilter() AND ValueFilter(=,'binary:sku188') AND KeyOnlyFilter()"`

ROW                          COLUMN+CELL
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=

从user1|ts2开始,找到所有的rowkey以user1开头的

scan 'test1', {STARTROW=>'user1|ts2', FILTER => "PrefixFilter ('user1')"}

ROW                          COLUMN+CELL
 user1|ts2                   column=sf:c1, timestamp=1409122354918, value=sku188
 user1|ts3                   column=sf:s1, timestamp=1409122354954, value=sku123



在hbase shell中使用scan命令时，可以使用filter来过滤记录。
这儿说明使用SingleColumnValueFilter来进行过滤的情况：

1）使用正则表达式：
scan ‘tweet0’, {FILTER=>”SingleColumnValueFilter(‘info’,’pubtime’,=,’regexstring:2014-11-08.*’)”}
匹配pubtime的以2014-11-08打头的值的记录。

2）使用上下边界值：
scan ‘tweet0’, {FILTER=>”SingleColumnValueFilter(‘info’,’pubtime’,>=,’binary:2014-11-08 19:26:27’) AND SingleColumnValueFilter(‘info’,’pubtime’,<=,’binary:2014-11-10 20:20:00’)”}

3）比较整数值：
scan ‘tweet0’, {FILTER=>”SingleColumnValueFilter(‘emotion’,’PB’,=,’binary:\x00\x00\x00\x05’)”, COLUMNS=>[‘emotion:PB’]}
这是比较整数值5，因为在查询时，这个5显示的也是\x00\x00\x00\x05。

4）参数说明：
SingleColumnValueFilter 这个过滤器有6个参数：列族、列名、比较运算符、比较器和两个可选参数：filterIfColumnMissing和setLatestVersionOnly。
filterIfColumnMissing：如果设置为true，则会把没有过滤器所指定列的行都过滤掉。默认值是false，所以看上去，当没有过滤器所指定的列时，过滤器不起作用。
setLatestVersionOnly：如果设置为false，则除了检查最新版本，还会检查以前的版本。默认值是true，只检查最新版本的值。
这两个参数要么一起使用，要么都不使用。

5）比较器：
前面例子中的regexstring:2014-11-08.*、binary:\x00\x00\x00\x05，这都是比较器。HBase的filter有四种比较器：
（1）二进制比较器：如’binary:abc’，按字典排序跟’abc’进行比较
（2）二进制前缀比较器：如’binaryprefix:abc’，按字典顺序只跟’abc’比较前3个字符
（3）正则表达式比较器：如’regexstring:ab*yz’，按正则表达式匹配以ab开头，以yz结尾的值。这个比较器只能使用=、!=两个比较运算符。
（4）子串比较器：如’substring:abc123’，匹配以abc123开头的值。这个比较顺也只能使用=、!=两个比较运算符。

6）比较运算符：
HBase的filter中有7个比较运算符：

1. LESS (<)
2. LESS_OR_EQUAL (<=)
3. EQUAL (=)
4. NOT_EQUAL (!=)
5. GREATER_OR_EQUAL (>=)
6. GREATER (>)
7. NO_OP (no operation)（不知道这个怎么用）
7）还可以如下写法：
import org.apache.hadoop.hbase.filter.SingleColumnValueFilter
import org.apache.hadoop.hbase.filter.CompareFilter
import org.apache.hadoop.hbase.filter.BinaryComparator
scan ‘tweet0’, { FILTER => SingleColumnValueFilter.new(Bytes.toBytes(‘info’), Bytes.toBytes(‘id’), CompareFilter::CompareOp.valueOf(‘EQUAL’), BinaryComparator.new(Bytes.toBytes(‘1001158684’))), COLUMNS=>[‘info:id’]}

但是，当在比较器中写一个整数时，总是匹配不成功，不知道错在哪儿。如：
scan ‘tweet0’, { FILTER => SingleColumnValueFilter.new(Bytes.toBytes(‘emotion’), Bytes.toBytes(‘PB’), CompareFilter::CompareOp.valueOf(‘EQUAL’), BinaryComparator.new(Bytes.toBytes(5))), COLUMNS=>[‘emotion:PB’]}

注意：
1）如果scan中指定了COLUMNS，则FILTER中所使用的列需要包含在所指定的COLUMNS中，否则，filter不起作用。
2）HBase中主要的操作对象是一个个的cell，每个cell都可以有多个版本。如果使用过滤器ValueFilter，就会只有那些符合条件的cell被查出来。跟关系数据库的查询不同，关系数据库查出来的结果中各行都有相同的列。而HBase，查出来的结果中，不同的行会有不同的列。
3）filter不会降低服务方的IO，它会把符合条件的子集传给客户端。即，它是在对查出的结果进行过滤，而不是象原来sql中的where子句。所以，如果要查出的结果中不包含filter需要的列，则filter就不能发挥作用。

