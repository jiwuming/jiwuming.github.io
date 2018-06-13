---
title: 使用Charles抓HTTPS的包
tags: [iOS, Swift]
date: 2018-04-28
---
又开始写新的项目了，这次的项目是用修改本地hosts去实现开发环境，测试环境和生产环境的切换的，然后使用到了Charles来抓取获取HTTPS的数据，Charles获取HTTP数据很简单，但是没经过配置获取HTTPS的数据是有乱码或者看不到的，记录一下解决HTTPS抓包的过程。
![](/img/charlesLunching.png)
<!--more-->

首先是在电脑端安装Charles的证书：

![](/img/charles-installcomputer certificate.png)

刚安装完的证书是不受信任的，我们需要把它改成信任的证书他才能正常工作：

![](/img/charles-certificate.png)

接下来我们设置手机代理，首先把Charles的端口配置好，默认8888我没有更改：

![](/img/charles-port.png)

然后用手机连上工作的网络，确保手机和电脑是在同一个区域网内，并配置手机的HTTP代理： 

![](/img/charles-phonedelegate.PNG)

这个地址就是电脑的ip地址可以通过`ifconfig en0`或者 系统偏好设置->网络 来进行查看。

这个时候手机应该就能正常抓包了，不过有的时候可能会出现一种情况：配置正确但是Charles没有弹出允许抓包的弹框，而且手机不能上网。这个我在网上查了一下有的人说是路由器的问题，我自己刚开始电脑和手机都用同一个WiFi的时候出现了这种情况，不过电脑换成有线的就好用了，如果有这种情况注意有线和无线都试一下就好，也有可能是端口被占用，切换端口试试。

接下来用手机去下载Charles的证书，在手机的Safari中打开这个地址：

![](/img/charles-phonecertificate-address.png)

这个时候手机就会有提示安装描述文件，点击确定，然后安装：

![](/img/charles-certificate-alert.PNG)

![](/img/charles-install-phonecertificate.JPG)

接下来还有一个步骤要设置，否则还是不能进行HTTPS的抓包，在手机上 通用->关于本机->证书信任设置 将Charles证书设置为信任：

![](/img/charles-certificate-correct.PNG)

接下来就可以愉快的使用HTTPS抓包和环境切换了。

![](/img/charles-result.png)

环境切换直接修改hosts中的文件就可以，把不需要的地址注释掉即可，如果没有配置即为生产环境。
`sudo vi /etc/hosts`

![](/img/charles-changehosts.png)
