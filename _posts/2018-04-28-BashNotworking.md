---
title: Mac 终端修改了.bash_profile之后终端命令无法使用的问题 
tags: [Terminal]
date: 2018-04-28
---
可能是由于我修改了.bash_profile文件的问题，我的终端什么命令都无法执行了。。。类似这样
```bash
Last login: Sat Apr 28 17:16:11 on ttys000
-bash: which: command not found
MacBook-Pro:~ ppd$ ls
-bash: ls: command not found
```
后来在网上找到了一句能临时使用终端的代码：
```bash
export PATH="/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/usr/X11/bin"
```
在终端里面输入这个直接回车，命令行就暂时可用了。然后我们用`open .bash_profile`打开这个文件，在把上面的一句添加到`.bash_profile`里面保存，然后`source .bash_profile`就可以了

