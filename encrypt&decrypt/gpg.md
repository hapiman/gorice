## 使用GPG实现非对称加密
主要源自IB开户中使用的公钥和私钥而整理

### 起源
主要是`PGP`收费，社区就开发了`GPG`

### GPG环境安装

### 生成密钥过程

- 生成过程

```
gpg --gen-key
gpg (GnuPG) 2.0.14; Copyright (C) 2009 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥种类：
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (仅用于签名)
   (4) RSA (仅用于签名)
您的选择？ 1
RSA 密钥长度应在 1024 位与 4096 位之间。
您想要用多大的密钥尺寸？(2048)4096
您所要求的密钥尺寸是 4096 位
请设定这把密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0) 0
密钥永远不会过期
以上正确吗？(y/n)y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

真实姓名：Bank One
电子邮件地址：jian.peng@xx.com
注释：pengjian's test
您选定了这个用户标识：
    “Bank One (pengjian's test) <jian.peng@xx.com>”

更改姓名(N)、注释(C)、电子邮件地址(E)或确定(O)/退出(Q)？q
gpg: 密钥生成已取消。
[supdev@11-00]$ gpg --gen-key
gpg (GnuPG) 2.0.14; Copyright (C) 2009 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥种类：
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (仅用于签名)
   (4) RSA (仅用于签名)
您的选择？ 1
RSA 密钥长度应在 1024 位与 4096 位之间。
您想要用多大的密钥尺寸？(2048)
gpg: signal Interrupt caught ... exiting

[supdev@11-00]$
[supdev@11-00]$
[supdev@11-00]$
[supdev@11-00]$ clease
-bash: clease: command not found
[supdev@11-00]$ clear
[supdev@11-00]$ gpg --gen-key
gpg (GnuPG) 2.0.14; Copyright (C) 2009 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥种类：
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (仅用于签名)
   (4) RSA (仅用于签名)
您的选择？ 1
RSA 密钥长度应在 1024 位与 4096 位之间。
您想要用多大的密钥尺寸？(2048)4096
您所要求的密钥尺寸是 4096 位
请设定这把密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0) 0
密钥永远不会过期
以上正确吗？(y/n)y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

真实姓名：pengj
电子邮件地址：jian.peng@xx.com
注释：pengj's gpg test
您选定了这个用户标识：
    “pengj (pengj's gpg test) <jian.peng@xx.com>”

更改姓名(N)、注释(C)、电子邮件地址(E)或确定(O)/退出(Q)？N
真实姓名：pengjian
您选定了这个用户标识：
    “pengjian (pengj's gpg test) <jian.peng@xx.com>”

更改姓名(N)、注释(C)、电子邮件地址(E)或确定(O)/退出(Q)？C
注释：pengjian
您选定了这个用户标识：
    “pengjian (pengjian) <jian.peng@xx.com>”

更改姓名(N)、注释(C)、电子邮件地址(E)或确定(O)/退出(Q)？C
注释：pengjian's gpg test
您选定了这个用户标识：
    “pengjian (pengjian's gpg test) <jian.peng@xx.com>”

更改姓名(N)、注释(C)、电子邮件地址(E)或确定(O)/退出(Q)？O
您需要一个密码来保护您的私钥。

can't connect to `/home/supdev/.gnupg/S.gpg-agent': 没有那个文件或目录
我们需要生成大量的随机字节。这个时候您可以多做些琐事(像是敲打键盘、移动
鼠标、读写硬盘之类的)，这会让随机数字发生器有更好的机会获得足够的熵数。
我们需要生成大量的随机字节。这个时候您可以多做些琐事(像是敲打键盘、移动
鼠标、读写硬盘之类的)，这会让随机数字发生器有更好的机会获得足够的熵数。
gpg: 密钥 ECC36BE6 被标记为绝对信任
公钥和私钥已经生成并经签名。

gpg: 正在检查信任度数据库
gpg: 需要 3 份勉强信任和 1 份完全信任，PGP 信任模型
gpg: 深度：0 有效性：  2 已签名：  0 信任度：0-，0q，0n，0m，0f，2u
pub   4096R/ECC36BE6 2018-06-10
密钥指纹 = 712B CBB9 52BB 4B69 FB0B  155A 155A 53F4 ECC3 6BE6
uid                  pengjian (pengjian's gpg test) <jian.peng@xx.com>
sub   4096R/7CB581D2 2018-06-10
```

- 关于密钥说明
```sh
# 公钥文件名
/home/supdev/.gnupg/pubring.gpg
-------------------------------
# 显示公钥特征（4096位，Hash字符串和生成时间）
pub   4096R/1F9DA2AD 2018-05-17
# 显示"用户ID"
uid                  Bank One (DAM Key) <bankone@bankonemail.com>
# 显示私钥特征
sub   4096R/A140BD32 2018-05-17
```

- 导出密钥
```sh
# Exporting the public key
gpg --armor --export jian.peng@xx.com(电子邮件名称) > pengjian.asc
# 或
gpg --armor --output pengjian.asc --export [用户ID]
# 导出私钥
gpg --armor --output private-key.txt --export-secret-keys

# Convert public key from ASCII to BINARY format
gpg --dearmor < pengjian.asc > pengjian.bin
# Encode BINARY version of public key in base64
cat pengjian.bin | base64 | tr -d '\n' > pengjian.bin.encoded
```

- 导入密钥
```sh
# 如果密钥使用了base64加密，需要解密
cat IBPublicKey.encoded | base64 -d > IBPublicKey.asc
# 如果密钥中具有 Comment: Use "gpg --dearmor " for unpacking 说明，则需要执行该操作
gpg --import -vv (gpg --dearmor执行之后的文件)
# 使用公钥签名
gpg –sign-key ibpublickey@bankonemail.com
```

- 删除密钥
`gpg --delete-key [用户ID]`

### 加密
```sh
# recipient参数指定接收者的公钥，output参数指定加密后的文件名，encrypt参数指定源文件。
# 运行下面的命令后，demo.en.txt就是已加密的文件，可以把它发给对方。
gpg --recipient [用户ID] --output demo.en.txt --encrypt demo.txt

# 签名 + 加密
# --local-user发信者的私钥，--recipient接收者的公钥
gpg --local-user [发信者ID] --recipient [接收者ID] --armor --sign --encrypt test.txt

```

### 解密
```sh
cat BankOneData.file.gpg.encoded | base64 -d > BankOneData.file.gpg
gpg --decrypt -o BankOneData.file(解密之后生成的文件) BankOneData.file.gpg(需要解密的文件)
# 或者指定使用的公钥，使用recipient
gpg --recipient 74A64469 --output test_en.txt --encrypt test.txt
```

### 包函数
nodejs中可以使用[openpgp](https://www.npmjs.com/package/openpgp)

### 相关问题

1.在生成密钥的过程中，如果进度停止在下面所示的文本
```
我们需要生成大量的随机字节。这个时候您可以多做些琐事(像是敲打键盘、移动
鼠标、读写硬盘之类的)，这会让随机数字发生器有更好的机会获得足够的熵数。
我们需要生成大量的随机字节。这个时候您可以多做些琐事(像是敲打键盘、移动
鼠标、读写硬盘之类的)，这会让随机数字发生器有更好的机会获得足够的熵数。

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```
直接使用工具`rng-tools`解决
```sh
yum install rng-tools
sudo rngd -r /dev/urandom
```
