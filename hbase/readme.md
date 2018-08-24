海量数据存储 100亿行 准实时查询100ms

[HBase的rowkey设计](https://www.cnblogs.com/cxzdy/p/5118456.html)


[](https://juejin.im/entry/57981aef79bc44006654bb76)
建表：create ‘test’, ‘cf1’, ‘cf2’，即[create ‘表名’, ‘列族名’,..]，列族名可以有多个，list用于查看有哪些表

hbase(main):008:0> create 'test','cf1','cf2'
0 row(s) in 1.2280 seconds
=> Hbase::Table - test
hbase(main):009:0>
写数据：put ‘test’, ‘row1’, ‘cf1:c1’, ‘value1’，即[put ‘表名’,’行键’,’列族名:列名’,’数据’]

hbase(main):001:0> put 'test','row1','cf1:c1','value1'
0 row(s) in 0.3160 seconds
hbase(main):002:0> put 'test','row1','cf1:c1','value2'
0 row(s) in 0.3020 seconds
查看数据：
全表数据：scan ‘test’，即[scan ‘表名’]
hbase(main):001:0> scan 'test'
ROW                                            COLUMN+CELL
row1                                          column=cf1:c1, timestamp=1469277197280, value=value2
1 row(s) in 0.2710 seconds
hbase(main):002:0>
可以看到在put时指定的属性之外，有一个timestamp属性来作为版本标识，我们查看全表数据时，row1的cf1:c1列中展示的值是我们后一次写入的value2，sacn和get在不指定版本时，得到的是最近版本的数据
指定行的数据：get ‘test’, ‘row1’，即[get ‘表名’,’行键’]
指定版本的数据：
hbase(main):005:0> get 'test','row1',{COLUMN=>'cf1:c1',TIMESTAMP=>1469277197280}
COLUMN                                         CELL
cf1:c1                                        timestamp=1469277197280, value=value1
1 row(s) in 0.0270 seconds
hbase(main):006:0>
版本数量：每个列族有一个单独的VERSIONS属性，默认为1，可以在建表时指定：create 'test1',{NAME=>'cf1',VERSIONS=>3}，代表该列族的每个列最多保存最近3个版本的数据，也可以通过alter来更新：alter 'test1',NAME=>'cf1',VERSIONS=>3。查询数据时，可以通过设置VERSIONS来指定显示最近几个版本的数据(最大范围不超过该列族的VERSIONS属性值)：get 'test','row1',{COLUMN=>'cf1:c1',VERSIONS=>2}
删除数据：
删除指定单元格：delete ‘test’,’row1’,’cf1:c1’,1469277197280，将删除指定版本以及比其更早的版本
删除指定行的指定列：delete ‘test’,’row1’,’cf1:c1’
删除整行： deleteall ‘test’,’row1’
禁用表：disable ‘test’，即[disable ‘表名’]，在要删除表或者变更配置时，要先禁用该表。相应的，要重新启用该表，使用[enable ‘表名’]
删除表：drop ‘test’，即[drop ‘表名’]
退出HBase shell:exit或者quit
完整的命令列表，参考hbase-shell-commands

获取某一行数据
`get 'gbl-web', 'rowkey1', 'cd'`
或者, 指定版本
`get 'gbl-web', 'rowkey1', {COLUMN=>'cd', VERSIONS=>2}`

查看一段时间范围内数据
`get 'user','rk0001',{COLUMN=>'info',VERSIONS=>3,TIMERANGE=>[1477654000000],[1477654111111]}`

查询时`{}`中的字段, 值都是使用`=>`表示
`COLUMN(列簇)`, `START_ROW`, `END_ROW`, `FILTER`, `VERSIONS`, `TIMERANGE`, `LIMIT`和传统一致, 会把该行的所有数据显示, `ROWPREFIXFILTER(一般使用PrefixFilter)`,
`CACHE`, `RAW`

`TIMERANGE`示例`TIMERANGE=>[1477654000000,1477654111111]`

delete 删除指定对象的值（可以为表，行，列，对应的值，另外也可以指定时间戳的值）
`delete 'table1', 'row1', 'info.name'`

通过时间戳删除指定的版本
`delete 'table1', 'row1', 'info.name' 1477654000000`

`truncate`清空数据

`alter 'table1' intrest` 添加列簇
`alter 'table1' 'delete' => 'data'`删除`data`列簇
`alter 'table1',{NAME=>'data',METHOD=>'delete'}`

只读某些列数据
`scan ‘qy’,{COLUMNS=>[‘name’,’foo’], LIMIT=>10}, TIMERANGE=>[11111, 22222]}`
`scan ‘qy’,{COLUMNS=>'info:name', LIMIT=>10}, TIMERANGE=>[11111, 22222]}`
`scan ‘qy’,{COLUMNS=>['info:name', 'info:age'], LIMIT=>10}, TIMERANGE=>[11111, 22222]}`

scan 'gbl-web', FILTER=>"ValueFilter(=, 'binary:D37265A6-CE87-4162-846D-9704BD180AEC')"
多条件查询
`scan 'test1', FILTER=>"ColumnPrefixFilter('s') AND ( ValueFilter(=,'substring:123') OR ValueFilter(=,'substring:222') )"`
`scan 't1', {ROWPREFIXFILTER => 'row2', FILTER => "(QualifierFilter (>=, 'binary:xyz')) AND (TimestampsFilter ( 123, 456))"}`

`count '表名',INTERVAL => 100,CACHE => 5000000` 计算条数


scan 'gbl-web',FILTER=>" (ValueFilter(=,'binary:1533610218818')) AND (ValueFilter(=,'binary:3cfac9a7-465f-4970-997d-e95cf686ebd2'))"

scan 'gbl-web',FILTER=>" (ValueFilter(=,'binary:3cfac9a7-465f-4970-997d-e95cf686ebd2')) OR (ValueFilter(=,'binary:1533610218818'))"

scan 'gbl-web',FILTER=>"ValueFilter(=,'binary:1533610218818')"
scan 'gbl-web',FILTER=>"ValueFilter(=,'binary:3cfac9a7-465f-4970-997d-e95cf686ebd2')"

get 'gbl-web','3cfac9a7-465f-4970-997d-e95cf686ebd2_1533610218850'

- `FilterList`两个属性

`MUST_PASS_ONE` 类似于`where a like ? or b=?`
`MUST_PASS_ALL` 类似于`where a like ? and b=?`

通过上面的方式可以组装`where （A  like ？ and B=？）or（where A  like ？ Or B=?)`格式

- 过滤器说明

参考[https://blog.csdn.net/power0405hf/article/details/49824579](https://blog.csdn.net/power0405hf/article/details/49824579)

||||
|:--|:--|:--|
|PrefixFilter|rowKey前缀过滤|`scan ‘qy’,{FILTER=>”PrefixFilter(‘001’)”}`|
|QualifierFilter|列过滤器,对列的名称|`scan ‘qy’,{FILTER=>”PrefixFilter(‘t’) AND QualifierFilter(>=,’binary:b’)”}`|
|TimestampsFilter|时间戳过滤器|`scan ‘qy’,{FILTER=>”TimestampsFilter(1448069941270,1548069941230)” }`|




- 比较运算符`OP`

|||
|:-----|:------
|LESS|匹配小于设定值的值
|LESS_OR_EQUAL|匹配小于等于设定值的值
|EQUAL|匹配与设定值不相等的值
|NOT_EQUAL|匹配小于等于设定值的值
|GREATER_OR_EQUAL|匹配大于或等于设定值的值
|GREATER|匹配大于设定值的值
|NO_OP|排除一切值

- 比较器介绍

|关键字|解释|
|:--------|:---------|
|BinaryComparator|使用Bytes.compareTo()比较当前值与阀值
|NullComparator|不做匹配，只判断当前值是不是`null`|
|RegexStringComparator|根据一个正则表达式，在实例化这个比较器的时候去匹配表中的数据|
|SubstringComparator|把阀值和表中数据当String实例，同时通过contains()操作匹配字符串|

