### 基本操作

!table查看表信息
!describe tablename可以查看表字段信息
!history可以查看执行的历史SQL
!dbinfo
!index tb;查看tb的索引
help查看其他操作

0.创建表
```sql
create table tb (stat varchar not null primary key, city varchar, num varchar)
```

1.插入数据
```sql
upsert into tb values('ak','hhh',222)
upsert into tb(stat,city,num) values('ak','hhh',222)

upsert into tb1 (state,city,population) select state,city,population from tb2 where population < 40000;
upsert into tb1 select state,city,population from tb2 where population > 40000;
upsert into tb1 select * from tb2 where population > 40000;
```

2.删除数据
```sql
delete from tb; -- 清空表中所有记录，Phoenix中不能使用truncate table tb；
delete from tb where city = 'kenai';
drop table tb; -- 删除表
delete from system.catalog where table_name = 'int_s6a';
drop table if exists tb;
drop table my_schema.tb;
drop table my_schema.tb cascade; -- 用于删除表的同时删除基于该表的所有视图。
```

3.更新数据
```sql
-- 由于HBase的主键设计，相同rowkey的内容可以直接覆盖，这就变相的更新了数据。
-- 所以Phoenix的更新操作仍旧是upsert into 和 upsert select
-- 示例中ak为主键
upsert into us_population (state,city,population) values('ak','juneau',40711);
```

4.查询数据
```sql
-- union all， group by， order by， limit 都支持
select * from test limit 1000;
select * from test limit 1000 offset 100;
select full_name from sales_person where ranking >= 5.0 union all select reviewer_name from customer_review where score >= 8.0
```
5.创建表
**A.SALT_BUCKETS(加盐)**
加盐Salting能够通过预分区(pre-splitting)数据到多个region中来显著提升读写性能。
本质是在hbase中，rowkey的byte数组的第一个字节位置设定一个系统生成的byte值，
这个byte值是由主键生成rowkey的byte数组做一个哈希算法，计算得来的。
Salting之后可以把数据分布到不同的region上，这样有利于phoenix并发的读写操作。


SALT_BUCKETS的值范围在（1 ~ 256）：
```sql
create table test(host varchar not null primary key, description  varchar)salt_buckets=16;

upsert into test (host,description) values ('192.168.0.1','s1');
upsert into test (host,description) values ('192.168.0.2','s2');
upsert into test (host,description) values ('192.168.0.3','s3');
```

salted table可以自动在每一个rowkey前面加上一个字节，这样对于一段连续的rowkeys，它们在表中实际存储时，就被自动地分布到不同的region中去了。
当指定要读写该段区间内的数据时，也就避免了读写操作都集中在同一个region上。
简而言之，如果我们用Phoenix创建了一个saltedtable，那么向该表中写入数据时，
原始的rowkey的前面会被自动地加上一个byte（不同的rowkey会被分配不同的byte），使得连续的rowkeys也能被均匀地分布到多个regions。


**B.Pre-split（预分区）**
Salting能够自动的设置表预分区，但是你得去控制表是如何分区的，
所以在建phoenix表时，可以精确的指定要根据什么值来做预分区，比如：
```sql
create table test (host varchar not null primary key, description varchar) split on ('cs','eu','na');
```

**C.使用多列族**
列族包含相关的数据都在独立的文件中，在Phoenix设置多个列族可以提高查询性能。
创建两个列族：
```sql
create table test (
 mykey varchar not null primary key,
 a.col1 varchar,
 a.col2 varchar,
 b.col3 varchar
);
upsert into test values ('key1','a1','b1','c1');
upsert into test values ('key2','a2','b2','c2');
```

D.使用压缩
```sql
create table test (host varchar not null primary key, description varchar) compression='snappy';
```

6.对列操作
```
ALTER TABLE my_schema.my_table ADD d.dept_id char(10) VERSIONS=10
ALTER TABLE my_table ADD dept_name char(50)
ALTER TABLE my_table ADD parent_id char(15) null primary key
ALTER TABLE my_table DROP COLUMN d.dept_id
ALTER TABLE my_table DROP COLUMN dept_name
ALTER TABLE my_table DROP COLUMN parent_id
ALTER TABLE my_table SET IMMUTABLE_ROWS=true
```
