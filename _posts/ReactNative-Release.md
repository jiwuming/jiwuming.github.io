---
title: ReactNative 集成到 iOS 工程
tags: [ReactNative]
date: 2017-05-20
---

不知道用不用这个开发呢...<!--more-->

1.首先新建一个 iOS 工程, 我们就叫他 NewRNWithOC 吧.
2.在工程的根目录创建一个文件, 命名为 package.json, 里面的代码大概是这样的:
```js
{
  "name": "NewRNWithOC", // 工程名
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "15.4.1",
    "react-native": "0.42" // 使用的 ReactNative 版本
  }
}
```
现在的目录:
![](/img/IMG-RN-RELEASE/packagedir.png)
3.然后在根目录下执行 `npm install`
4.在根目录下面创建一个 `index.ios.js` 可以复制一份 hello world 的代码进去
![](/img/IMG-RN-RELEASE/indexdir.png)
5.写一个 Podfile 导入需要的 RN 组件
```js
platform :ios, '8.0'
# target的名字一般与你的项目名字相同
target 'NewRNWithOC' do

  # 'node_modules'目录一般位于根目录中
  # 但是如果你的结构不同，那你就要根据实际路径修改下面的`:path`
  pod 'React', :path => './node_modules/react-native', :subspecs => [
    'Core',
    'RCTText',
    'RCTImage',
    'RCTNetwork',
    'RCTAnimation',
    'RCTWebSocket', # 这个模块是用于调试功能的
    # 在这里继续添加你所需要的模块
  ]
  # 如果你的RN版本 >= 0.42.0，请加入下面这行
  pod "Yoga", :path => "./node_modules/react-native/ReactCommon/yoga"

end

```
6.安装组件之后, npm start, 实现原生跳转 RN :
```mm
#import "ViewController.h"
#import <React/RCTRootView.h>

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    self.view.backgroundColor = [UIColor whiteColor];
    UIButton *button = [[UIButton alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
    button.backgroundColor = [UIColor redColor];
    [self.view addSubview:button];
    
    [button addTarget:self action:@selector(buttonAction) forControlEvents:UIControlEventTouchUpInside];
}

- (void)buttonAction {
    NSURL *jsCodeLocation = [NSURL
                                 URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios"];
    
    RCTRootView *rootView =
    [[RCTRootView alloc] initWithBundleURL : jsCodeLocation
                         moduleName        : @"RNHome"
                         initialProperties : nil
                          launchOptions    : nil];
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view = rootView;
    [self.navigationController pushViewController:vc animated:YES];
}
```
7.RN 跳转 原生页面:
```mm
新建一个原生页面 继承于 NSObject 我就叫他 RNModule 了

.h
#import <Foundation/Foundation.h>
#import <React/RCTBridgeModule.h>

@interface RNModule : NSObject<RCTBridgeModule>

@end

.m

#import "RNModule.h"
#import "AppDelegate.h"

@implementation RNModule

RCT_EXPORT_MODULE(); // 导出模块

RCT_EXPORT_METHOD(popViewController){ // 实现 RN 页面上 的 popViewController 方法 由原生来执行 达到返回原生页面的目的
    
    dispatch_async(dispatch_get_main_queue(), ^{ // 必须在主线程中执行
        
        AppDelegate *app = (AppDelegate *)[[UIApplication sharedApplication] delegate];
        
        [app.nav popViewControllerAnimated:YES];
    });
}

@end

```
RN 部分
```js
var module = NativeModules.RNModule;

export default class Main extends Base {

    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {};
        this.pushActon = this.pushActon.bind(this)
    }

    componentDidMount() {
        this.setState({
            navTitle: '这是RN页面'
        })
    }

    pushActon() {
        this.props.navigator.push({
            component: SecondPage
        })
    }

    navBackButtonAction() {
        module.popViewController(); // 交给原生页面去实现 这里是无参数的方法 可根据需求自己加参数上去
    }

    createUI() {
        return(
            <TouchableOpacity style={styles.mainStyle} onPress={this.pushActon}>
                <Text style={{textAlign: 'center'}}>Hello ReactNative</Text>
            </TouchableOpacity>
        )
    }
}
```
8.打包
执行命令:
```js
react-native bundle --entry-file index.ios.js --platform ios --dev false --bundle-output ./index.ios.jsbundle --assets-dest ./
```
执行完上面的命令之后, 会在根目录里面得到一个 assets 的文件夹和 index.ios.jsbundle
用导入蓝色文件夹这种方式导入到工程里面
![](/img/IMG-RN-RELEASE/importRnmodule.png)
![](/img/IMG-RN-RELEASE/moduledir.png)
然后再把js的入口改成这样:
```mm
NSURL *jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"index.ios" withExtension:@"jsbundle"];
```
再把工程改成 Release 模式就可以打包发布了
