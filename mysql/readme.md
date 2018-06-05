`mysql`修改某一列数据
```sh
# CHANGE 修改某一列的所有信息，相当于删除某列然后新建一列
ALTER TABLE MyTable CHANGE COLUMN foo bar VARCHAR(32) NOT NULL;

# MODIFY 不用修改列名的其他修改，详单于Change删了一列，然后新建了列名相同，其他信息可能不同的一列。
ALTER TABLE MyTable MODIFY COLUMN foo VARCHAR(32) NOT NULL
```

对于原始的`sql`当在某种类型错误时如何处理？
```sql
-- 使用on duplicate key来处理
CREATE PROCEDURE `updateuserlogintime`(IN inmaxclientdataid int,OUT outmaxclientdataid int)
BEGIN
if inmaxclientdataid >0 then
insert into stat_app_devices(deviceid,platform,productkey,updatedAt,createdAt)
  select deviceid,platform, productKey,insertdate, date
  from `cobub`.`razor_clientdata` where id>=inmaxclientdataid
  on duplicate key update updatedAt = Values(updatedAt),productkey = Values(productKey);
  select max(id) into outmaxclientdataid from `cobub`.`razor_clientdata`;
else
  select max(id) into outmaxclientdataid from `cobub`.`razor_clientdata`;
end if;
END
```

```sql
begin
  -- 定义变量
  set @inpar = 0,@outpar = 0;
  -- 确保数据库存在
  create table if not exists cobstat.tmp_updatetime(maxid int not null);
  -- 取数据，并赋值
  select @inpar := maxid from cobstat.tmp_updatetime;
  -- 调用函数
  call cobstat.updateuserlogintime(@inpar,@outpar);
  -- 清空数据库
  truncate table cobstat.tmp_updatetime;
  -- 存数据
  insert into cobstat.tmp_updatetime values(@outpar);
end
```
