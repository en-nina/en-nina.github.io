---
title: macSQLCipher
date: 2020-09-04 18:56:27
tags:SQLCipher
---



**SQlite数据库加密**

**本人mac操作如下**

一： 安卓SQLClipher

~~~
brew install sqlcipher
~~~



二：下载sqlcipher

https://github.com/sqlcipher/sqlcipher

下面地址可以下载对应版本

下载release版本源码 [sqlcipher](https://github.com/sqlcipher/sqlcipher/releases/tag/v3.3.1)



三：编译sqlcipher

~~~
$ ./configure --enable-tempstore=yes CFLAGS="-DSQLITE_HAS_CODEC" LDFLAGS="-lcrypto -L/usr/local/opt/openssl/lib" CPPFLAGS="-I/usr/local/opt/openssl/include"
$ make
~~~

*以上要确保编译正确，编译完成后会看到目录多了一个sqlcipher 的可执行文件，其实他是个快捷方式。离开这个目录他啥也不是*



四：使用SQLCipher

*创建并加密sqlite*

~~~
##加密
./sqlcipher test.db  #创建一个新的db文件
sqlite> PRAGMA key = 'test';  #设置密码
sqlite> .read test.sql  #导入数据 我的数据sql是用工具导出的
sqlite> .e  #退出
~~~



**参考地址**

[SQLCipher 命令行使用 后台加密](https://blog.csdn.net/sskicgah/article/details/37502433)

[使用SQLClipher](https://luowei.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/sqlite-encrypt-with-sqlcipher.html)

[iOS sqlite加密](https://www.dazhuanlan.com/2019/10/28/5db613916c202/)

