---
title: Cordova iOS插件开发
tags: [cordova]
date: 2019-3-14
---
首先安装插件开发工具
```bash
npm install -g plugman
```
然后创建插件, 目录无所谓先放在桌面上就行(貌似参数都是必填的)
```bash
plugman create --name 插件名 --plugin_id 插件ID --plugin_version 插件版本号
```
创建完之后得到一个文件和两个文件夹
```bash
plugin.xml
src/
www/
```
cd到插件文件夹下, 添加 iOS 平台
```bash
plugman platform add --platform_name ios
```
删掉掉自带的文件, 在`src`的`ios`目录下编写iOS插件, 比如做一个alert
```mm
.h
#import <Cordova/CDV.h> // 注意如果使用继承 CDVPlugin 直接创建的可能不是这个 要改掉 否则会报错
NS_ASSUME_NONNULL_BEGIN

@interface TestAlertPlugin : CDVPlugin
- (void)alertWithjsParams:(CDVInvokedUrlCommand *)jsParams;
@end

NS_ASSUME_NONNULL_END
---------------------------------------------------
.m
- (void)alertWithjsParams:(CDVInvokedUrlCommand *)jsParams {
    // 使用js的第一个参数来进行提示
    NSString *str = jsParams.arguments[0];
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:str message:@"" preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *action = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        // 回调给js
        CDVPluginResult* pluginResult = [CDVPluginResult resultWithStatus:CDVCommandStatus_OK messageAsString:@"回调"];
        [self.commandDelegate sendPluginResult:pluginResult callbackId:jsParams.callbackId];
    }];
    [alert addAction:action];
    // 获取当前控制器弹出alert
    [[self getCurrentVC] presentViewController:alert animated:YES completion:nil];
}
```
然后在`www`目录下面编写供前端调用的`js`接口
```js
var exec = require('cordova/exec');

exports.testAlert = function(arg0, success, error) {
    // 参数分别为成功回调, 失败回调, OC类名, OC类中的方法, js传过来的参数(数组形式, 不过我一般用对象, 所以传一个就好了) 
    exec(success, error, "TestAlertPlugin", "alertWithjsParams", [arg0]);
};
```
之后编写plugin.xml配置
```html
<?xml version='1.0' encoding='utf-8'?>
<plugin id="插件id" version="1.0.0" xmlns="http://apache.org/cordova/ns/plugins/1.0" xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 插件名(重要) -->
    <name>testAlert</name>
    <!-- js模块(重要) -->
    <js-module name="testAlert" src="www/testAlert.js">
        <clobbers target="cordova.plugins.testAlert" />
    </js-module>
    <platform name="ios">
        <config-file parent="/*" target="config.xml">
            <!-- 原生插件名称(重要) -->
            <feature name="TestAlertPlugin">
                <!-- 原生插件类(重要) -->
                <param name="ios-package" value="TestAlertPlugin" />
            </feature>
        </config-file>
        <!-- 原生插件文件(重要) -->
        <source-file src="src/ios/TestAlertPlugin.m" />
        <header-file src="src/ios/TestAlertPlugin.h" />
    </platform>
</plugin>
```
最后一步, 创建package.json文件, 在插件根目录`npm init`, 其实写一个名字就好了, 其他的我都没填
```json
{
  "name": "test-alert",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
最后的目录结构:
```bash
package.json
plugin.xml
src/ios/原生插件
www/前端接口文件
```
cd 到 cordova 目录下安装插件 `cordova plugin add 本地插件目录` 
引入`cordova.js`
```html
<script type="text/javascript" src="cordova.js"></script>
```
在调用就可以了:
```js
 <script>
    function buttonClick() {
        // 这里写的比较粗糙,第一个 testAlert 是js模块名(见插件目录下的plugin.xml 的 js-module标签) 第二个testAlert 是js模块方法(见前端接口文件www目录下的 export 的方法名)
        cordova.plugins.testAlert.testAlert('js传参', success, fail)

        // 如果你不确定你写的功能是否好用可以用cordova直接调用
        // 参数: 成功回调 失败回调 OC类名 OC方法 传参
        // cordova.exec(success,fail,"TestAlertPlugin","alertWithjsParams",["我是JS传的参数！"]);

    }
    function success(params) {
        console.log('调用成功')
        console.log(params)
        console.log(cordova.plugins)
    }
    function fail() {
        console.log('调用失败')
    }
</script>
<button onclick="buttonClick()">点我测试功能</button>
```
![](/img/cordovanativealert.jpeg)
![](/img/cordoaconsole.jpeg)

