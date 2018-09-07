`curl ip.gs` 查询当前电脑的公网IP

`ifconfig | inet adr` 查看当前机器的IP

`find fabric -name "*.go" -not -path "fabric/vendor/*" | xargs cat | wc -l` 查看关键字的个数

shell命令中的`>`和`<`使用 ？

`grep`, `sed`结合`sed`统计查阅日志的行数

` dmidecode -t memory | grep Size: | grep -v "No Module Installed"`查看内存中总量


u 代表用户.
g 代表用户组.
o 代表其他.
a 代表所有.

这意味着chmod u+x somefile 只授予这个文件的所属者执行的权限
而 chmod +x somefile 和 chmod a+x somefile 是一样的
Just doing +x will apply it to all flags: [u]ser, [g]roup, [o]thers.




x        删除当前光标下的字符
dw       删除光标之后的单词剩余部分。
d$       删除光标之后的该行剩余部分。
dd       删除当前行。

c        功能和d相同，区别在于完成删除操作后进入INSERT MODE
cc       也是删除当前行，然后进入INSERT MODE

vim中
`0`表示跳转到行首
`$`表示跳转到行尾
