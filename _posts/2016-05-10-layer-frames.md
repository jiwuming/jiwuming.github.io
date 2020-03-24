---
title: 解决tableViewCell中有圆角imageView卡顿的问题
tags: [Swift, iOS]
date: 2016-05-10
---
对项目中的列表页做了一些简单的优化, 其中涉及到界面流畅度的问题, 关于流畅到什么程度, 可以用KMCGeigerCounter来检测帧数达到多少, 在根据情况去进行优化. 圆角是导致滑动卡顿的问题之一, 记录一下解决过程.

测试环境: iPhone6s, iOS9.2
用到的工具: Alamofire, Kingfisher, KMCGeigerCounter
<!--more-->
首先去请求网络数据:
```javascript
// 请求地址
let urlString = "http://c.3g.163.com/recommend/getChanListNews?channel=T1457068979049&passport=zvZe9MQ7cxTSrcWyjXHlNmWlYJxG7i%2FExMsC%2FIX1t2E%3D&devId=tYiXx8W73%2BGGquuNxF8c8PkrobMEegFcSVAs152mOWH7Vqve0omfVq78wYMgkonU&size=40&version=9.0&spever=false&net=wifi&lat=&lon=&ts=1464162516&sign=rC4vOZ5ChMjNUe8YKmdGHE3VIm%2BZjtNzO49jFFbSUJx48ErR02zJ6%2FKXOnxX046I&encryption=1&canal=appstore"
```
```javascript
// 发起请求
func requestData(success: ([String]) -> Void) {
Alamofire.request(.GET, urlString, parameters: nil)
    .responseData { (resultData) -> Void in
        // 请求是否成功
        switch resultData.result {
            // 请求成功
            case .Success:
                var jsonDic: [String : AnyObject] = [:]
                var resultArr: [String] = []
                guard let data = resultData.data else {
                    return
                }
                do{
                    jsonDic = try NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions.MutableContainers) as! [String : AnyObject]
                }
                catch let error as NSError {
                    print("解析出现错误: \(error)")
                    return
                }
                let arr = jsonDic["视频"] as! [AnyObject]
                for str in arr {
                    let imgStr = str["cover"] as! String
                    resultArr.append(imgStr)
                }
                success(resultArr)
            // 请求失败
            case .Failure(let error):
                print(error)
        }
    }
}
```
接下来把他们显示在tableView上:
```javascript
imgView.kf_setImageWithURL(NSURL(string: imageName)!)
```
如图:![](/img/before.png)
顶部显示的帧数是60帧, 而且无论如何滑动也不会丢帧.

但是我们想要一个圆角的imageView, 如果只设置layer.cornerRadius, 加载了图片之后, 一样不是圆角的image. 但是如果使用了layer.masksToBounds 或者         imgView.clipsToBounds 这样就会导致tableView滑动的时候非常的卡

![](/img/ka.png)

第一列 被我设置成了圆角, 滑动了几下, 就剩41帧了, 这个时候手机已经感觉到明显的卡顿了
后来我采用了直接绘制图片的方法, 而不是直接去设置imageView
```mm
imgView.kf_setImageWithURL(NSURL(string: imageName)!, placeholderImage: nil, optionsInfo: .None) { (image, error, cacheType, imageURL) -> () in
    let img = image
    UIGraphicsBeginImageContextWithOptions(self.imgView.bounds.size, false, 2.0);
    UIBezierPath(roundedRect: self.imgView.bounds, cornerRadius: 10).addClip()
    img?.drawInRect(self.imgView.bounds)
    self.imgView.image = UIGraphicsGetImageFromCurrentImageContext()
    UIGraphicsEndImageContext()            
}
```
![](/img/done.png)

这样再滑动就不会卡了.