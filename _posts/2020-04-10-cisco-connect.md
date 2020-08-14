---
title: 思科AnyConnect本地网络不被信任的问题
tags: [Cisco]
date: 2020-04-10
---
如果使用的`Mac`系统版本是`Catalina`，请使用4.8以上的`AnyConnect`，否则无法运行。然后在连接`VPN`的时候可能会弹出这样的提示：
```bash
AnyConnect cannot confirm it is connected to your secure gateway. The local network may not be trustworthy. Please try another network
```
这个其实和本地网络环境没有关系的，我尝试换了几个网络也不行，是`AnyConnect`的配置问题。
移动到目录`/opt/cisco/anyconnect/`找到`AnyConnectLocalPolicy.xml`，将`ExcludeMacNativeCertStore`设置成`true`，重新连接即可。