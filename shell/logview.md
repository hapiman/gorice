### 查看日志的实例
``` sh
# 1.查询两个时间段的日志, 首先要保证确实存在这两个标示
sed -n '/2018-03-31 12:31:04/,/2018-03-31 12:31:11/p'  xx

# 2.分页显示
cat -n xx |grep "debug" |more

# 3.将日志文件存储起来
cat -n test.log |grep "debug"  >debug.txt

# 4.连接文件，形成新的文件
cat file1 file2 > file3
```
