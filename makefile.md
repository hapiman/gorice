- 打印变量信息
```makefile
# 打印变量
$(warning $(VARIABLE_NAME))
# 显示文本
$(warning this is an $(VARIABLE_NAME))
# 直接报错,终止编译
$(error this is error message)
```

- 赋值

= 是最基本的赋值
:= 是覆盖之前的值
?= 是如果没有被赋值过就赋予等号后面的值
+= 是添加等号后面的值

`=`和`:=`的区别

make会将整个makefile展开后，再决定变量的值。也就是说，变量的值将会是整个makefile中最后被指定的值。看例子：
```makefile
x = foo
y = $(x) bar
x = xyz
```
在上例中，y的值将会是 xyz bar ，而不是 foo bar 。

“:=”表示变量的值决定于它在makefile中的位置，而不是整个makefile展开后的最终值。
```makefile
x := foo
y := $(x) bar
x := xyz
```
在上例中，y的值将会是 foo bar ，而不是 xyz bar 了。

```makefile
ifdef DEFINE_VRE
    VRE = “Hello World!”
else
endif

ifeq ($(OPT),define)
    VRE ?= “Hello World! First!”
endif

ifeq ($(OPT),add)
    VRE += “Kelly!”
endif

ifeq ($(OPT),recover)
    VRE := “Hello World! Again!”
endif

all:
    @echo $(VRE)
# make DEFINE_VRE=true OPT=define 输出：Hello World!
# make DEFINE_VRE=true OPT=add 输出：Hello World! Kelly!
# make DEFINE_VRE=true OPT=recover  输出：Hello World! Again!
# make DEFINE_VRE= OPT=define 输出：Hello World! First!
# make DEFINE_VRE= OPT=add 输出：Kelly!
# make DEFINE_VRE= OPT=recover 输出：Hello World! Again!
```

- patsubst使用

格式：$(patsubst <pattern>,<replacement>,<text> )
名称：模式字符串替换函数——patsubst。
功能：查找<text>中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式<pattern>，如果匹配的话，则以<replacement>替换。

　　　这里，<pattern>可以包括通配符“%”，表示任意长度的字串。如果<replacement>中也包含“%”，那么，<replacement>中的这个“%”将是<pattern>中的那个“%”所代表的字串。

　　　（可以用“\”来转义，以“\%”来表示真实含义的“%”字符）
返回：函数返回被替换过后的字符串。

示例：

$(patsubst %.c,%.o, a.c b.c)

把字串“a.c b.c”符合模式[%.c]的单词替换成[%.o]，返回结果是“a.o b.o”

make中有个变量替换引用



对于一个已经定义的变量，可以使用“替换引用”将其值中的后缀字符（串）使用指定的字符（字符串）替换。格式为“$(VAR:A=B)”（或者“${VAR:A=B}”），

意思是，替换变量“VAR”中所有“A”字符结尾的字为“B”结尾的字。“结尾”的含义是空格之前（变量值多个字之间使用空格分开）。而对于变量其它部分的“A”字符不进行替换。

例如：

foo := a.o b.o c.o

bar := $(foo:.o=.c)



在这个定义中，变量“bar”的值就为“a.c b.c c.c”。使用变量的替换引用将变量“foo”以空格分开的值中的所有的字的尾字符“o”替换为“c”，其他部分不变。

如果在变量“foo”中如果存在“o.o”时，那么变量“bar”的值为“a.c b.c c.c o.c”而不是“a.c b.c c.c c.c”。

使用以下3个内置函数
1、wildcard : 扩展通配符
2、notdir ： 去除路径
3、patsubst ：替换通配符

建立一个测试目录，在测试目录下建立一个名为sub的子目录
$ mkdir  mk
$ cd mk
$ mkdir sub

在mk下，建立a.c和b.c 2个文件，在sub目录下，建立aa.c和bb.c 2 个文件

建立一个简单的Makefile
src=$(wildcard *.c ./sub/*.c)
file=$(notdir $(src))
obj=$(patsubst %.c,%.o,$(src) )

all:
　　@echo $(src)　　
　　@echo $(file)
　　@echo $(obj)


执行结果分析：
第一行输出：
a.c b.c ./sub/aa.c ./sub/bb.c

wildcard把 指定目录 ./ 和 ./sub/ 下的所有后缀是c的文件全部展开。

第二行输出：
a.c b.c aa.c bb.c
notdir把展开的文件去除掉路径信息

第三行输出：
a.o b.o aa.o bb.o

在$(patsubst %.c,%.o,$(src) )中，patsubst把$(file)中的变量符合后缀是.c的全部替换成.o，
任何输出。
