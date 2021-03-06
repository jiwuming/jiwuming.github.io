---
title: Mac抹掉系统并重装遇到的一些问题
tags: [Mac]
date: 2020-05-10
---
去年本来以为电脑要变成板砖了。。。基本都在使用公司的电脑。今天在家没事搞搞自己的电脑，没想到突然卡死，然后被我强制关机，再开机就无法打开了。情况是进度条是满的，但无法进入桌面，开机按`Command + R`也是一样，都是进度条满了无法进入。

根据网上提供的重置[NVRAM](https://support.apple.com/zh-cn/HT204063)，开机`Command + S`之类的都没有什么效果，都是满进度条无法进入系统，不过我电脑上没有什么重要的东西，就想到了如果能把电脑格式化掉重装系统也可以，在网上查了半天终于有了一点结果：

开机按住`Command + option + R`会有一个小地球出来，然后等下会提示连接无线网络，输入可用的无线网络，继续等待。

这其中时间可能会很长。

最后会进入到恢复模式，就是开机长按`Command + S`的那个页面。

接下来就是重装系统了，先抹掉磁盘的数据，如果电脑上的系统是`Catalina`的话，可能会看到两个宗卷（之前的系统就不清楚了，我使用的就是`Catalina`）一个是`Macintosh HD`另一个是`Macintosh HD-数据`。

我先格式化的`Macintosh HD`，这其中会自动删除掉`Macintosh HD-数据`这个宗卷，并且删除到最后会报错，有一个进程`kextcache`正在占用硬盘，既无法卸载硬盘，也无法运行急救之类的操作。

这不就真成板砖了？硬盘数据删掉了还没删干净，系统又不能重新启动。但如果之前仔细观察一下这两个宗卷会发现：如果一个宗卷里面的数据显示是亮着的（蓝色），那么这个宗卷中的数据在另一个宗卷中显示的就是暗着的（灰色），这种感觉就像`Windows`硬盘分区但又没有分盘符一样。

如果在这种状态下尝试重新安装`Catalina`是不行的，因为到了选择硬盘的那一步会提示：无法安装到目标宗卷，因为该宗卷是不完整的系统宗卷。

反正已经成砖了，我抱着试一试的心态在`Macintosh HD`这个宗卷上右键点击新建宗卷，创建了一个`Macintosh HD-Data`，然后再次抹掉`Macintosh HD`，居然成功了！

![](/img/mac-rebuild-system.jpeg)

接下来重装`Catalina`就ok了！

顺便再提一句，重装后的系统，配置环境变量的时候也出了一些问题。如果发现在`.bash_profile`中配置的环境变量，在退出命令之后就无效了的话，把环境变量配置在`.zshrc`中，如果没有就创建。