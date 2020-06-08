---
title: 从SDWebImage看多线程带来的一些问题
tags: [iOS]
date: 2017-05-16
---
最近整理了一波多线程的相关知识, 多线程开发除了能够充分利用系统资源, 提升工作效率和响应速度之外, 同时也带来了一些缺点, 比如线程同步问题, 共享资源调度问题等。针对这些问题记录一些解决方案。

# 多线程的好处与坏处
优点:
* 充分利用CPU资源, 提升工作效率
* 不会阻塞页面操作, 可以进行后台任务
* 可以指定任务的优先级, 开始暂停和取消

缺点:
* 大量的线程开销会导致内存飚升
* 需要线程同步机制
* 调试增加了一定的难度

对于串行的队列, 几乎不会产生太大的问题, 最多就是我们可能需要进行一下线程同步, 可以使用`@synchronized`, `dispatch_semaphore`, `dispatch_barrier_sync`之类的方法同步一下就好, 问题往往出现在那些异步的并发线程上, 可以举一些例子来看一下。

# 高并发带来的问题
其实作为前端来说可能实际的业务中可能不会写很多高并发请求的代码, 但我们可以模拟一下这种情况, 比如, 一个页面中同时进行三四十个请求会发生什么?
```cpp
NSArray *urlArray = @[
        @"https://www.baidu.com/",
        @"https://www.sina.com.cn/",
        @"https://www.qq.com/",
        @"https://www.sohu.com/",
        @"https://www.taobao.com/",
        @"https://www.alibaba.com/",
        @"https://www.aliyun.com/",
        @"https://blog.csdn.net/",
        @"https://github.com/",
        @"https://www.bilibili.com/",
        @"https://www.zhihu.com/",
        @"https://www.apple.com/",
        @"https://www.stackoverflow.com/",
        @"https://cn.bing.com/",
        @"https://www.microsoft.com/",
        @"https://www.ctrip.com/",
        @"https://www.ly.com/",
        @"https://www.csair.com/cn/",
        @"https://www.163.com/",
        @"https://www.2345.com/",
        @"https://www.toutiao.com/",
        @"https://www.hao123.com/",
        @"https://www.douyu.com/",
        @"https://www.huya.com/",
        @"https://www.twitch.com/",
        @"https://cloud.tencent.com/",
        @"https://www.oschina.net/",
        @"https://www.qidian.com/",
        @"https://www.17173.com/",
        @"https://www.7k7k.com/",
        @"https://www.5173.com/",
        @"https://www.iconfont.cn/"
    ];
    for (NSString *i in urlArray) {
        [self request:i];
    }
}

- (void)request:(NSString *)url {
    NSURLSession *session = [NSURLSession sharedSession];
    NSURLSessionDataTask *task6 = [session dataTaskWithURL:[NSURL URLWithString:url] completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"开始响应 = %ld, url = %@", [(NSHTTPURLResponse *)response statusCode], [(NSHTTPURLResponse *)response URL]);
    }];
    [task6 resume];
}
```
严格来说这并不是并发, 但即使是这样的请求也能使内存"跳跃"式的增长(大概10多兆左右), 这便是高并发给内存带来的压力。

# 合理的执行任务
话说回来, 其实高并发的场景在绝大部分应用里都会有, 比如我有一个列表, 列表中有图片和文字的排版, 那么这个图片的加载实际上就很接近高并发请求, 尤其是当我们不断滑动列表时。当然这些问题我们一般会选择图片加载框架去做, 比如`SDWebImage`这种, 那么它们又是怎么处理这些任务的呢?

我们可以写一个`tableView`其`Cell`中只有一个`imageView`, 这样写一个图片列表来观察`SDWebImage`的加载过程。
```cpp
// 这里只讨论加载过程, 先不考虑缓存, 每次加载的时候先把缓存清掉
[[SDImageCache sharedImageCache] clearMemory];
[[SDImageCache sharedImageCache] clearDiskOnCompletion:^{
    [self createTabelView];
}];
```
使用`SDWebImage`加载图片最简单的api应该就是这个了`[self.imgView sd_setImageWithURL:[NSURL URLWithString:imageUrl]];`我们可以一步一步点进去可以到这样的一个方法:
```cpp
- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                      operationKey:(nullable NSString *)operationKey
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                         completed:(nullable SDExternalCompletionBlock)completedBlock;
```
其中的参数无非是对站位图的, 加载策略, 回调等设置, 我们暂且不考虑站位图, 不考虑加载的url为空这种特殊情况, 直接往下看, 这个方法再往下看可以看到这样的一个方法:
```cpp
id <SDWebImageOperation> operation = [SDWebImageManager.sharedManager loadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
    // ...
}
```
可以看到进入了`SDWebImageManager`这个类中, 我们可以点开这个方法, 它的上面有一系列的加载失败的url处理, 这个我们也先不管, 再往下是缓存查找, 在缓存查找的block中我们可以看到相关的下载方法:
```cpp
operation.cacheOperation = [self.imageCache queryCacheOperationForKey:key done:^(UIImage *cachedImage, NSData *cachedData, SDImageCacheType cacheType) {
    // ...
    SDWebImageDownloadToken *subOperationToken = [self.imageDownloader downloadImageWithURL:url options:downloaderOptions progress:progressBlock completed:^(UIImage *downloadedImage, NSData *downloadedData, NSError *error, BOOL finished) {
         __weak SDWebImageDownloader *wself = self;
        return [self addProgressCallback:progressBlock completedBlock:completedBlock forURL:url createCallback:^SDWebImageDownloaderOperation *{
            // ...
        }
    }
    // ...
}
```
其中 最内部的方法主要是添加下载任务和生成下载`token`

```cpp
- (nullable SDWebImageDownloadToken *)addProgressCallback:(SDWebImageDownloaderProgressBlock)progressBlock
                                           completedBlock:(SDWebImageDownloaderCompletedBlock)completedBlock
                                                   forURL:(nullable NSURL *)url
                                           createCallback:(SDWebImageDownloaderOperation *(^)())createCallback {
    // The URL will be used as the key to the callbacks dictionary so it cannot be nil. If it is nil immediately call the completed block with no image or data.
    if (url == nil) {
        if (completedBlock != nil) {
            completedBlock(nil, nil, nil, NO);
        }
        return nil;
    }

    __block SDWebImageDownloadToken *token = nil;

    dispatch_barrier_sync(self.barrierQueue, ^{
        SDWebImageDownloaderOperation *operation = self.URLOperations[url];
        if (!operation) {
            operation = createCallback();
            // URLOperations 字典类型 以 url: SDWebImageDownloaderOperation 的形式添加
            self.URLOperations[url] = operation;
            __weak SDWebImageDownloaderOperation *woperation = operation;
            operation.completionBlock = ^{
              SDWebImageDownloaderOperation *soperation = woperation;
              if (!soperation) return;
              if (self.URLOperations[url] == soperation) {
                  [self.URLOperations removeObjectForKey:url];
              };
            };
        }
        // 是一个block回调
        id downloadOperationCancelToken = [operation addHandlersForProgress:progressBlock completed:completedBlock];

        token = [SDWebImageDownloadToken new];
        token.url = url;
        token.downloadOperationCancelToken = downloadOperationCancelToken;
    });
    
    return token;
}
```
接下来就是主要的下载任务:
```cpp
// 处理超时
__strong __typeof (wself) sself = wself;
NSTimeInterval timeoutInterval = sself.downloadTimeout;
if (timeoutInterval == 0.0) {
    timeoutInterval = 15.0;
}

// In order to prevent from potential duplicate caching (NSURLCache + SDImageCache) we disable the cache for image requests if told otherwise
// 请求与缓存设置
NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url cachePolicy:(options & SDWebImageDownloaderUseNSURLCache ? NSURLRequestUseProtocolCachePolicy : NSURLRequestReloadIgnoringLocalCacheData) timeoutInterval:timeoutInterval];
request.HTTPShouldHandleCookies = (options & SDWebImageDownloaderHandleCookies);
request.HTTPShouldUsePipelining = YES;
if (sself.headersFilter) {
    request.allHTTPHeaderFields = sself.headersFilter(url, [sself.HTTPHeaders copy]);
}
else {
    request.allHTTPHeaderFields = sself.HTTPHeaders;
}
SDWebImageDownloaderOperation *operation = [[sself.operationClass alloc] initWithRequest:request inSession:sself.session options:options];
operation.shouldDecompressImages = sself.shouldDecompressImages;

// 认证设置
if (sself.urlCredential) {
    operation.credential = sself.urlCredential;
} else if (sself.username && sself.password) {
    operation.credential = [NSURLCredential credentialWithUser:sself.username password:sself.password persistence:NSURLCredentialPersistenceForSession];
}

// 任务优先级设置
if (options & SDWebImageDownloaderHighPriority) {
    operation.queuePriority = NSOperationQueuePriorityHigh;
} else if (options & SDWebImageDownloaderLowPriority) {
    operation.queuePriority = NSOperationQueuePriorityLow;
}

// 开始执行任务
[sself.downloadQueue addOperation:operation];
if (sself.executionOrder == SDWebImageDownloaderLIFOExecutionOrder) {
    // Emulate LIFO execution order by systematically adding new operations as last operation's dependency
    [sself.lastAddedOperation addDependency:operation];
    sself.lastAddedOperation = operation;
}

return operation;
```
可以看到, `SDWebImage`下载的核心队列名字叫`downloadQueue`, 我们这时再去初始化方法里面看一下就知道他的下载方法了:
```cpp
_downloadQueue = [NSOperationQueue new];
_downloadQueue.maxConcurrentOperationCount = 6;
_downloadQueue.name = @"com.hackemist.SDWebImageDownloader";
```
可以看到, `SDWebImage`使用`NSOperationQueue`进行下载, 默认最大并发数为6, 其余的任务都必须要等待之前的任务完成之后再去进行, 这样就不会开辟太多的系统资源, 如果只是下载流程不放置图片的话, 内存几乎没有涨幅。

# 使用线程安全的对象
上面描述的仅仅是`SDWebImage`的下载过程, 当然这个框架不局限于做下载过程, 还有缓存的过程, 其步骤分为内存缓存及硬盘缓存, 我们先来探究内存缓存的过程。

首先我们可以来看下`SDWebImageManager`在初始化的时候做了什么:
```cpp
- (nonnull instancetype)init {
    SDImageCache *cache = [SDImageCache sharedImageCache];
    SDWebImageDownloader *downloader = [SDWebImageDownloader sharedDownloader];
    return [self initWithCache:cache downloader:downloader];
}

- (nonnull instancetype)initWithCache:(nonnull SDImageCache *)cache downloader:(nonnull SDWebImageDownloader *)downloader {
    if ((self = [super init])) {
        _imageCache = cache;
        _imageDownloader = downloader;
        _failedURLs = [NSMutableSet new];
        _runningOperations = [NSMutableArray new];
    }
    return self;
}
```
我们可以看到他初始化了一个缓存对象`SDImageCache`, 我们可以沿着它缓存的方法一步一步的点进去, 最终到这样的一个方法:
```cpp
- (void)storeImage:(nullable UIImage *)image
         imageData:(nullable NSData *)imageData
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock {
    if (!image || !key) {
        if (completionBlock) {
            completionBlock();
        }
        return;
    }
    // if memory cache is enabled
    if (self.config.shouldCacheImagesInMemory) {
        NSUInteger cost = SDCacheCostForImage(image);
        [self.memCache setObject:image forKey:key cost:cost];
    }
    
    if (toDisk) {
        dispatch_async(self.ioQueue, ^{
            NSData *data = imageData;
            
            if (!data && image) {
                SDImageFormat imageFormatFromData = [NSData sd_imageFormatForImageData:data];
                data = [image sd_imageDataAsFormat:imageFormatFromData];
            }
            
            [self storeImageDataToDisk:data forKey:key];
            if (completionBlock) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    completionBlock();
                });
            }
        });
    } else {
        if (completionBlock) {
            completionBlock();
        }
    }
}
```
中间一部分我们可以看到他是一个`self.memCache`对象存储了图片对象, 而这个对象点上去的属性是`NSCache`:
```cpp
@property (strong, nonatomic, nonnull) NSCache *memCache;
```
我在之前也没有用过这个对象, 可以去[苹果官方文档](https://developer.apple.com/documentation/foundation/nscache?language=objc)去看一下:
> A mutable collection you use to temporarily store transient key-value pairs that are subject to eviction when resources are low.
> 
> Cache objects differ from other mutable collections in a few ways:
>
> The NSCache class incorporates various auto-eviction policies, which ensure that a cache doesn’t use too much of the system’s memory. If memory is needed by other applications, these policies remove some items from the cache, minimizing its memory footprint.
> 
> You can add, remove, and query items in the cache from different threads without having to lock the cache yourself.
> 
> Unlike an NSMutableDictionary object, a cache does not copy the key objects that are put into it.

从它的介绍中我们可以看到, 首先这是一个键值存储对象, 但它的对象会在系统资源不足时会被系统释放掉最小化占用内存, 这个对象的操作线程安全的, 可以从任意线程添加或删除, 与`NSMutableDictionary`不同的是, `NSCache`不会拷贝键值。

这样我们就明白了, 这个对象最大的优势就在于他的系统管理和线程安全层面上, 如果使用的是`NSMutableDictionary`管理缓存对象的话, 则不具备这些优势, 所有的添加和删除都要加线程锁, 降低了效率, 而且不能自动释放对象需要手动管理。

# 资源的释放问题
`SDWebImage`图片在缓存之后, 再次读取图片会先从缓存中查找, 如果内存中没有就去硬盘中查, 如果硬盘中没有就去看当前是否有下载的任务, 如果没有任务才进行下载。我们去看一下它查找缓存的过程:
```
// SDIMageCache.m
- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key done:(nullable SDCacheQueryCompletedBlock)doneBlock {
    if (!key) {
        if (doneBlock) {
            doneBlock(nil, nil, SDImageCacheTypeNone);
        }
        return nil;
    }

    // First check the in-memory cache...
    UIImage *image = [self imageFromMemoryCacheForKey:key];
    if (image) {
        NSData *diskData = nil;
        if ([image isGIF]) {
            diskData = [self diskImageDataBySearchingAllPathsForKey:key];
        }
        if (doneBlock) {
            doneBlock(image, diskData, SDImageCacheTypeMemory);
        }
        return nil;
    }

    NSOperation *operation = [NSOperation new];
    dispatch_async(self.ioQueue, ^{
        if (operation.isCancelled) {
            // do not call the completion if cancelled
            return;
        }

        @autoreleasepool {
            NSData *diskData = [self diskImageDataBySearchingAllPathsForKey:key];
            UIImage *diskImage = [self diskImageForKey:key];
            if (diskImage && self.config.shouldCacheImagesInMemory) {
                NSUInteger cost = SDCacheCostForImage(diskImage);
                [self.memCache setObject:diskImage forKey:key cost:cost];
            }

            if (doneBlock) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    doneBlock(diskImage, diskData, SDImageCacheTypeDisk);
                });
            }
        }
    });

    return operation;
}
```
注意观察, 去硬盘查找这一步使用了异步串行队列, 这是由于硬盘查找属于I/O操作, 时间上会慢一些所以使用了异步操作。而下面的查找方法使用了`@autoreleasepool`, 这个是因为`UIImage`与`NSData`的操作都是比较耗费内存的操作, 案例可以参考[这里](https://segmentfault.com/q/1010000006100424/a-1020000006100470), 所以采用`@autoreleasepool`来避免内存达到峰值的情况,
[有关@autoreleasepool的官方文档](https://developer.apple.com/documentation/foundation/nsautoreleasepool?language=occ)
文档中提到了:

> The Application Kit creates an autorelease pool on the main thread at the beginning of every cycle of the event loop, and drains it at the end, thereby releasing any autoreleased objects generated while processing an event. If you use the Application Kit, you therefore typically don’t have to create your own pools. If your application creates a lot of temporary autoreleased objects within the event loop, however, it may be beneficial to create “local” autorelease pools to help to minimize the peak memory footprint.

其实`SDWebImage`不止在上述代码中用到了`@autoreleasepool`在`SDWebImageDecoder`这个类中一样也用到了`@autoreleasepool`, 按照`UIImage *diskImage = [self diskImageForKey:key];`这句代码中的方法一步一步点进去就能找到:
```cpp
- (nullable UIImage *)diskImageForKey:(nullable NSString *)key {
    NSData *data = [self diskImageDataBySearchingAllPathsForKey:key];
    if (data) {
        UIImage *image = [UIImage sd_imageWithData:data];
        image = [self scaledImageForKey:key image:image];
        if (self.config.shouldDecompressImages) {
            // 这里的性能仍然有问题, 对于大的图片可以使用 [[SDWebImageDownloader sharedDownloader] setShouldDecompressImages:NO];禁用压缩
            image = [UIImage decodedImageWithImage:image];
        }
        return image;
    }
    else {
        return nil;
    }
}
```
# 线程安全问题
`SDWebImage`多数使用`@synchronized`来为线程加锁, 使用`dispatch_main_async_safe`的宏定义来进行线程同步,
```cpp
#ifndef dispatch_main_async_safe
#define dispatch_main_async_safe(block)\
    if (strcmp(dispatch_queue_get_label(DISPATCH_CURRENT_QUEUE_LABEL), dispatch_queue_get_label(dispatch_get_main_queue())) == 0) {\
        block();\
    } else {\
        dispatch_async(dispatch_get_main_queue(), block);\
    }
#endif
```
其意义在于保证数据的准确性及UI的主线程操作。我们可以举一个例子, 比如网购的秒杀活动, 要保证库存读取的准确性, 不能出现负数的库存, 我们可以模拟一下这个情况:
```cpp
_items = 200;
NSLog(@"主线程开始");
dispatch_queue_t queue1 = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue2 = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue3 = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue4 = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue5 = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
dispatch_queue_t queue6 = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);

dispatch_async(queue1, ^{
    [self buy];
});
dispatch_async(queue2, ^{
    [self buy];
});
dispatch_async(queue3, ^{
    [self buy];
});
dispatch_async(queue4, ^{
    [self buy];
});
dispatch_async(queue5, ^{
    [self buy];
});
dispatch_async(queue6, ^{
    [self buy];
});
NSLog(@"主线程结束");

- (void)buy {
    while (true) {
        if (_items > 0) {
            _items--;
            NSLog(@"%ld, ---- %@", _items, [NSThread currentThread]);
        } else {
            break;
        }
    }
    NSLog(@"结束了");
}
```
这里列举了了六个队列抢200件货物的情景, 这样写是因为如果队列比较少的话, 不太好分析线程情况, 有可能只能看到一条线程的执行情况, 需要多次打印才能分析出来, 而200件货物也足够观察打印情况了。我们的目标是:数据读取准确, 不能出现负数, 如果不做任何处理, 上面的打印数据有可能出现这种情况:
```bash
***** 时间14:03:47 SDImageLoadViewController.m 第91行 *****
GQLOG: 150, ---- <NSThread: 0x282179400>{number = 4, name = (null)}

***** 时间14:03:47 SDImageLoadViewController.m 第91行 *****
GQLOG: 149, ---- <NSThread: 0x282179400>{number = 4, name = (null)}

***** 时间14:03:47 SDImageLoadViewController.m 第91行 *****
GQLOG: 191, ---- <NSThread: 0x282149f80>{number = 6, name = (null)}

***** 时间14:03:47 SDImageLoadViewController.m 第91行 *****
GQLOG: 147, ---- <NSThread: 0x282149f80>{number = 6, name = (null)}
```
可以看到线程6在第一次读取的时候出现了问题, 本来只剩149件货物了, 但是他读取成了191, 最危险的是像下面这种:
```bash
***** 时间14:03:47 SDImageLoadViewController.m 第97行 *****
GQLOG: 结束了   <NSThread: 0x282178f80>{number = 5, name = (null)}

***** 时间14:03:47 SDImageLoadViewController.m 第91行 *****
GQLOG: 13, ---- <NSThread: 0x282179400>{number = 4, name = (null)}
```
货物已经售罄了, 但是还是读取到了13件, 这也就相当于出现了负数, 那么如何避免这种情况呢? 可以用上面提到的`SDWebImage`的方法:
```cpp
- (void)buy {
    while (true) {
        @synchronized (self) {
            if (_items > 0) {
                _items--;
                NSLog(@"%ld, ---- %@", _items, [NSThread currentThread]);
            } else {
                break;
            }
        }
    }
    NSLog(@"结束了   %@", [NSThread currentThread]);
}
```
`GCD`的信号量也可以起到这种作用
```cpp
_sem = dispatch_semaphore_create(1);

- (void)buy {
    while (true) {
        dispatch_semaphore_wait(_sem, DISPATCH_TIME_FOREVER);
        if (_items > 0) {
            _items--;
            NSLog(@"%ld, ---- %@", _items, [NSThread currentThread]);
            dispatch_semaphore_signal(_sem);
        } else {
            dispatch_semaphore_signal(_sem);
            break;
        }
    }
    NSLog(@"结束了   %@", [NSThread currentThread]);
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"主线程回调");
    });
}
```
# 总结
总结一下多线程带来的问题大概有内存问题, 线程安全问题, 线程同步问题, 以及可能会遇到的资源释放问题, 对应的解决办法即合理控制队列数量, 使用线程安全的对象, 必要时使用线程锁, 这目前还只是理论篇, 关键还是要放到应用中去使用。