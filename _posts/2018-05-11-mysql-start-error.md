---
title: Mac Mysql 启动的问题
tags: [Mysql]
date: 2018-05-11
---
 今天装了一个Navicat来准备使用数据库, 然后发现忘记了数据库的密码...也不知道初始密码是啥...略尴尬, 因为我现在手里的电脑是公司的, 也不知道上一个人设置的密码是啥, 所以决定重置密码来解决一下这个问题, 然后由重置密码又引出一堆问题...
![](/img/mysqlstartpanda.jpg)
<!-- more -->
首先是这个问题
```bash
mac The server quit without updating PID file (/usr/local/var/mysql/xxx.local)
```
大概就是`/usr/local/var/mysql/xxx.local`这个目录下没有对应用户的pid file, 首先想到的是权限问题, 这里还要吐槽一下新版的mac权限设置, 某些文件夹下是不能使用`chmod 777`的, 因为系统不允许, 提示`Operation not permitted`当然这也不是新出的功能, 很早的时候就有了, 只是我很少去东系统文件夹下面的东西。那么怎么解决这个问题呢? 重启mac, 按住`command + R`等电脑读条结束后会进入到recover模式, 然后选择工具->命令行, 执行`csrutil disable`然后重启。

然后我们去设置mysql的目录权限`chmod -R 777 /usr/local/var/mysql`, 这样第一个问题解决。

第一个问题解决之后又爆出了第二个问题, 在终端输入mysql出现了下面的错误
```bash
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock (2)
```
首先还是想到权限问题, `/tmp`这个文件夹的权限很奇怪, 即使你用了`sudo chmod -R 777 /tmp`系统还是提示权限不够, 当然这个权限问题还不是在这里发现的, 我以前也确实碰到过这样的问题, 为了保证以后的步骤不在遇到权限问题我决定用root账户去操作。

首先进入root`sudo -su`, 然后运行`mysql.server start`, 数据库启动成功, 早知道早用root了 = = 。接着我又切回到我自己的账户, 再次运行, 果然没出现上面的错误, 但是出现了这个:
```bash
pcf:~ ppd$ mysql.server start
Starting MySQL
 SUCCESS! 
pcf:~ ppd$ mysql
ERROR 1045 (28000): Access denied for user 'ppd'@'localhost' (using password: NO)
pcf:~ ppd$ sudo mysql
Password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)'
```
我用了sudo也不行, 看来原来是有密码的, 于是通过`./mysqld_safe --skip-grant-tables &`来禁用密码验证功能
```bash
pcf:~ ppd$ sudo mysql.server stop
Shutting down MySQL
.. SUCCESS! 
pcf:~ ppd$ mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
[1] 8802
pcf:~ ppd$ 180511 09:19:08 mysqld_safe Logging to '/usr/local/var/mysql/pcf.local.err'.
180511 09:19:08 mysqld_safe Starting mysqld daemon with databases from /usr/local/var/mysql
```
首先停止mysql服务, 然后禁用mysql的验证, 这个时候他会执行到最后那一条不动, 然后我们__再开启一个终端__。

通过`mysql -u root mysql`来进入mysql命令行模式:
```bash
Last login: Thu May 10 17:14:54 on ttys003
pcf:~ ppd$ mysql -u root mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.10 Homebrew

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
通过`SET PASSWORD FOR 'root'@'localhost' = PASSWORD('新的密码');`来设置新的密码, 有的时候由于环境的问题可能不允许, 执行`flush privileges`就可以啦。不过这是我本地的数据库还好, 现网环境慎用, 需要重启数据库, 并且安全性也比较难以保证。
```bash
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('新的密码');
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('新的密码');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
接下来就可以正常使用myql了
```bash
pcf:~ ppd$ sudo mysql.server start
Password:
Starting MySQL
 SUCCESS! 
pcf:~ ppd$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.10 Homebrew

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```





