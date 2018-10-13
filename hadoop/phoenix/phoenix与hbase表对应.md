### `hbase`的行键就是`phoenix`的主键

在hbase shell 中建表：
```sh
create 'SCORE','A', 'B'

put 'SCORE','Tom','A:M','5'
put 'SCORE','Lilei','B:M','5'

scan 'SCORE'

# ROW              COLUMN+CELL
# Lilei            column=B:M, timestamp=1427679080104, value=5
# Tom              column=A:M, timestamp=1427679078854, value=5
```
phoenix client端建表

```sh
create table "SCORE" (pk VARCHAR primary key, "A"."M" VARCHAR , "B"."M" VARCHAR );
select * from score;
# +------------+------------+------------+
# |     PK     |     M      |     M      |
# +------------+------------+------------+
# | Lilei      | null       | 5          |
# | Tom        | 5          | null       |
# +------------+------------+------------+

# 删除数据表
drop table score;
```


首先创建一张HBase表，再创建的Phoenix表，表名必须和HBase表名一致即可。
create  'stu' ,'cf1','cf2'
put 'stu', 'key1','cf1:name','luozhao'
put 'stu', 'key1','cf1:sex','man'
put 'stu', 'key1','cf2:age','24'
put 'stu', 'key1','cf2:adress','cqupt'


create table "stu" (
id VARCHAR NOT NULL  PRIMARY KEY ,
"cf1"."name" VARCHAR ,
"cf1"."sex" VARCHAR ,
"cf2"."age" VARCHAR ,
"cf2"."adress" VARCHAR );
upsert into "stu"(id,"cf1"."name","cf1"."sex","cf2"."age","cf2"."adress") values('key6','zkk','man','111','Beijing');


select * from "stu";会发现两张表是数据同步的。
这里查询表名需要用双引号括起来，强制不转换为大写。
这里一定要注意的是表名和列族以及列名需要用**双引号**括起来，因为HBase是区分大小写的，
如果不用双引号括起来的话Phoenix在创建表的时候会自动将小写转换为大写字母
