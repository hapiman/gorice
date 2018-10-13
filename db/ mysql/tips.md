## 关键点
- 如何`mysql`处理大批量数据的删除，使用`limit`控制删除的数量

`show processlist` 查看当前mysql正在执行的命令

`service mysqld restart`重启mysql服务

### 相关问题
```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket
```
方案1.
ps -A|grep mysql
显示类似：
1829 ?        00:00:00 mysqld_safe
1876 ?        00:00:31 mysqld
2.#kill -9 1829
3.#kill -9 1876
4.#/etc/init.d/mysql restart
5.#mysql -u root -p
