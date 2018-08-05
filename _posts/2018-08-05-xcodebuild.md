---
title: 使用 xcodebuild 命令打包与 fir-cli 自动发布项目
tags: [Terminal]
date: 2018-08-05
---
由于最近需要开发的工程有多个环境, 又是企业式发布 app 到 fir 上, 每次打一个包实在是太麻烦了, 而且需要不停的更换证书和描述文件, 所以决定把打包操作用 shell 执行。记录一下操作过程。

我目前的 app 是有三个环境的, 一个是开发环境, 另一个是 UAT 环境, 剩下的一个就是生产环境了。由于开发环境和生产环境我们可以用 Debug 和 Release 来区分, 所以我们只要多配置一个 UAT 环境的就可以了。

首先先建立一个 configurations 我这里叫 WM_UAT
![](/img/wm-proj-confg.png)

然后把证书配好
![](/img/wm-proj-sign.png)

接下来去配置不同的 bundle id, bundle version 和 bundle display name 这个可以在 build setting 里面加入配置 然后在工程的 plist 文件中去取, 首先在 build setting 中增加配置
![](/img/wm-proj-addsetting1.png)

然后加入 bundle version 和 bundle display name
![](/img/wm-proj-addsetting2.png)

对于 bundle id 的配置在这里
![](/img/wm-proj-addsetting3.png)

最后在 plist 中读取相应的值, 这样就做到了不同的环境对应不同的配置
![](/img/wm-proj-plist.png)

对于程序中请求 url 可以通过 bundle id 来区分, 或者用 host 区分的也可以, 这个看项目。

工程配置完毕了开始写 shell 我这里是只打了 UAT 和 生产的包, 根据包的不同要有不同的导出配置文件
![](/img/wm-xcodebuild-config.png)

这个配置文件的内容可以先用 Xcode 的 archive 打一个包, 然后 export 然后把导出文件夹中的 ExportOptions.plist 复制过来, 改一改 bundle id 和 描述文件名称就可以了, 这两项位于 ExportOptions.plist 的 provisioningProfiles 字段中

然后安装 fir-cli 这个安装命令倒是很简单, 只要执行`gem install fir-cli`就可以了。不过我安装这个的时候遇到了点问题, 他管我要了两个目录的权限, 其中一个是`/usr/bin`, 我手贱的设置了更改了`/usr/bin`的权限之后发现命令行用不了了!!提示是这样的
```bash
login: Could not determine audit condition
[进程已完成]
```
网上有说开启root账户的然后执行以下命令的
```bash
#第一个解决办法
chown -R root:wheel /usr/bin
chmod 4755 /usr/bin/sudo
reboot //重启电脑

#第二个解决办法(和第一个差不多)
sudo -s    //登录你的ROOT用户密码
chown -R root:wheel /usr/bin
chmod 4755 /usr/bin/sudo
chmod 4755 /usr/bin/login
reboot
```
反正我试过了, 都不行。
后来我试了[这个方法](http://www.powenko.com/wordpress/mac-terminal-error-could-not-determine-audit-condition/)终于把命令行登进去了

>Mac Terminal Error : Could not determine audit condition
When I launched my terminal today, was welcomed with this error :-
login: PAM Error (line 396): System error
login: Could not determine audit condition
[Process completed]
It is most probably because I was playing with my /usr/bin permissions the other day.
The fix is easy. Just delete the "/usr/bin/login" dir.
But how do I delete it, if I can't access the "Terminal" altogether ?
Come on - You can access any folder using the "Finder".
>1. Open "Finder"
>2. Open "Go To Folder"
>3. Type "/usr/bin/login"
>4. Delete it.

开搞之前记得先备份...

最后, archive.sh的代码
```bash
#创建打包ipa的目录
if [ ! -d ./ipaPackage ];
then
mkdir -p ipaPackage;
fi
project_path=工程的绝对路径
project_name=ipa的名字targets叫啥名这里就叫啥
scheme_name=scheme的名称
development_mode=打包模式 Debug Release other... 这里不写也行因为后面会改掉
#打包工程要先编译, 这里配置的是编译路径
build_path=${project_path}/build
#导出.ipa文件所在路径
exportIpaPath=${project_path}/ipaPackage/${development_mode}

echo "要打生产的包还是UAT的包? [1:生产 2:UAT] 输入数字之后按回车"

read number
while([[ $number != 1 ]] && [[ $number != 2 ]])
do
echo "请输入数字1或2"
echo "要打生产的包还是UAT的包? [1:生产 2:UAT] 输入数字之后按回车"
read number
done

if [ $number == 1 ];then
#更换打包模式
development_mode=Release
#打包配置文件路径
exportOptionsPlistPath=${project_path}/ExportOptions.plist
echo '准备打包生产...'
else
#更换打包模式
development_mode=WM_UAT
#打包配置文件路径
exportOptionsPlistPath=${project_path}/ExportOptionsUAT.plist
echo '准备打包UAT...'
fi

xcodebuild \
clean -configuration ${development_mode} -quiet  || exit

echo '正在编译工程:'${development_mode}

xcodebuild \
archive -workspace ${project_path}/${project_name}.xcworkspace \
-scheme ${scheme_name} \
-configuration ${development_mode} \
-archivePath ${build_path}/${project_name}.xcarchive  -quiet  || exit

echo '编译完成'
echo '开始ipa打包...'

xcodebuild -exportArchive -archivePath ${build_path}/${project_name}.xcarchive \
-configuration ${development_mode} \
-exportPath ${exportIpaPath} \
-exportOptionsPlist ${exportOptionsPlistPath} \
-allowProvisioningUpdates \
-quiet || exit

if [ -e $exportIpaPath/$scheme_name.ipa ]; then
echo 'ipa导出完成'
open $exportIpaPath
else
echo 'ipa导出失败!'
fi

# fir上传 这里自动上传我只配置了UAT环境的
if [ $number == 2 ];then
fir login -T 这里填写fir的api_token
fir publish $exportIpaPath/$scheme_name.ipa
fi

```







