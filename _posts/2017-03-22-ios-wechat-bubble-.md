---
title: iOS 类似微信的图片气泡
tags: [iOS, Objective-C]
date: 2018-03-22
---

最近要准备一个即时通讯相关的项目, 其中主要的页面当然还是聊天页面了, 今天来主要写一下没有气泡边框的`imageView`. 大致的效果是这样

![](/img/weChatBubbleImage.jpeg)
<!-- more -->

实现思路大概是根据获取到图片的比例来拉伸气泡, 然后通过气泡的形状创建一个`imageView`的`imageView.layer.mask`是一个`CALayer`对象.

关于气泡拉伸的方法可以参考MJ的这篇博客[iOS图片拉伸技巧](https://blog.csdn.net/q199109106q/article/details/8615661)

首先是这样一个气泡
![](/img/messageBubble.png)

然后创建一个`UIImageView`拉伸气泡
```mm
    _imgView = [[UIImageView alloc] initWithFrame:CGRectMake(15, 15, 100, 200)];
    _imgView.backgroundColor = [UIColor redColor];
    [self.view addSubview:_imgView];

    UIImage *img = [[UIImage imageNamed:@"messageBubble"] resizableImageWithCapInsets:UIEdgeInsetsMake(30, 15, 15, 15) resizingMode:UIImageResizingModeStretch];
    _imgView.image = img;
```
这里为了方便查看效果我把`_imgView`的大小设置成了固定值

拉伸之后的效果是这样的
![](/img/messageBubbleStretch.png)
然后我们创建一个`CALayer`根据气泡绘制出来
```mm
CALayer *layer = [[CALayer alloc] init];
layer.contents = (__bridge id _Nullable)(img.CGImage);
layer.contentsCenter = [self contentsCenterWithImage:img];
layer.frame = CGRectMake(0, 0, 100, 200);
layer.contentsScale = [UIScreen mainScreen].scale;
layer.opacity = 1;
_imgView.layer.mask = layer;

- (CGRect)contentsCenterWithImage:(UIImage *)image {
    return CGRectMake(image.capInsets.left / image.size.width, image.capInsets.top / image.size.height, (image.size.width - image.capInsets.right - image.capInsets.left) / image.size.width, (image.size.height - image.capInsets.bottom - image.capInsets.top) / image.size.height);
}
```
我们现在可以看到我们的`imageView`已经和气泡是一样的形状了
![](/img/messageBubbleRed.png)
然后再把图片重新赋值就可以了_(:з」∠)_
```mm
_imgView.contentMode = UIViewContentModeScaleAspectFill;
_imgView.image = [UIImage imageNamed:@"timg.jpeg"];
```
![](/img/messageBubbleDog.png)