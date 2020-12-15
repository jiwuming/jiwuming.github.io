# 1.weak 和 assign 不同
> 1. `weak`必须用于`oc`对象`assign`可以用于非`oc`对象
> 2. `weak`可以用来解决循环引用问题`weak`修饰的属性表示一种非持有关系 给该对象设置新值时 设置方法既不保留新值也不释放旧值 在属性遭到摧毁时会自动制空
> 3. `weak`可以用来修饰已经被强引用过得对象 比如使用`xib`进行布局时设置的属性 

# 2.weak 的实现原理
> 1. `runtime`维护了一个全局的`weak`引用表`weak_table_t`(封装版本为`SideTable`) 其底层是一个`hash`表(一种`key value`存储的数据结构 在`cpp`中体现为`struct`) 用于存储`weak`引用对象 `key`是所指对象地址 `value`是`weak`指针地址
> 2. `weak`对象释放自动置空实现: 在对象释放的最后一步`objc_clear_deallocating`根据对象地址获取所有`weak`指针的数组 遍历这个数组把其中的额数据置为空
> `oc`对象释放的顺序: 
> 1. `objc_release`, 
> 2. `dealloc`大部分对象是在这一环节被销毁的 其中主要做的事情为 object_dispose() associate 绑定的对象也是在这里销毁 所以不管是MRC还是ARC都不需要在`dealloc`中再次释放 associate 绑定对象,
> 3. `_objc_rootDealloc`,
> 4. `objc_destructInstance`,
> 5. `objc_clear_deallocating`

# 3.copy修饰的属性
> 1. `copy`即对象的浅拷贝 `copy`修饰的属性的值会有被`保护`的作用 一般`copy`会用来修饰不可变对象 比如 `NSString, NSArray`等 这样的对象在被可变对象接收以后 其值仍然不可变 可以利用此封装性质封装一些数据
> 2. 自己的属性想实现`copy`需要实现`NSCopying`协议 并实现 `- (id)copyWithZone:(NSZone *)zone`方法

# 4.@property的本质
> 1. 属性的本质是 `getter` + `setter` + `ivar`
> 2. 属性自动合成(autosynthesis)的规则: 自动合成会生成下面五个东西
```cpp
// 该属性的偏移量 这个偏移量是硬编码 标示该变量距离存放对象的内存地址有多远
OBJC_IVAR_$类名$属性名 
getter与setter方法对应的实现函数
// 成员列表
ivar_list 
// 方法列表
method_list
// 属性列表
prop_list
```
> 每增加一个属性 系统都会在 `ivar_list` 中添加一个成员变量的描述 在 `method_list` 中增加 `getter` 与 `setter` 方法的描述 在`prop_list`中给出属性描述 然后计算属性在对象中的偏移量然后根据偏移量的位置 实现具体的`setter`方法与`getter`方法

# 5. @synthesize 和 @dynamic
> 1. `@sythesize` 如果没有手动实现`getter`和`setter`由编译器自动创建 默认为 `@syntheszie var = _var;`
> 2. `@dynamic` `getter`和`setter`都需要手动创建

# 6.ARC下 不指定属性任何的关键字 默认的修饰符有哪些
> 1. `atomic` `readwrite` `string` oc 对象
> 2. `atomic` `readwrite` `assign` 基本类型

# 7.objc_msgSend()
> `Objective-C`是动态语言 每个方法会在运行时被动态转为消息发送 即`objc_msgSend(receiver, selector)`
> `oc`类的基本结构:

```cpp
struct objc_class {
  Class isa OBJC_ISA_AVAILABILITY; //isa指针指向Meta Class，因为Objc的类的本身也是一个Object，为了处理这个关系，runtime就创造了Meta Class，当给类发送[NSObject alloc]这样消息时，实际上是把这个消息发给了Class Object
  #if !__OBJC2__
  Class super_class OBJC2_UNAVAILABLE; // 父类
  const char *name OBJC2_UNAVAILABLE; // 类名
  long version OBJC2_UNAVAILABLE; // 类的版本信息，默认为0
  long info OBJC2_UNAVAILABLE; // 类信息，供运行期使用的一些位标识
  long instance_size OBJC2_UNAVAILABLE; // 该类的实例变量大小
  struct objc_ivar_list *ivars OBJC2_UNAVAILABLE; // 该类的成员变量链表
  struct objc_method_list **methodLists OBJC2_UNAVAILABLE; // 方法定义的链表
  struct objc_cache *cache OBJC2_UNAVAILABLE; // 方法缓存，对象接到一个消息会根据isa指针查找消息对象，这时会在method Lists中遍历，如果cache了，常用的方法调用时就能够提高调用的效率。
  struct objc_protocol_list *protocols OBJC2_UNAVAILABLE; // 协议链表
  #endif
  } OBJC2_UNAVAILABLE;
```

> `unrecognized selector` 就是由于消息转发的时候没有在方法列表中找到产生的

> 消息转发过程 `objc_msgForward`: 
> 1. objc在向一个对象发送消息时`runtime`会根据对象的`isa`指针找到该对象所属的类 然后在该类的方法列表中查找目标方法 如果该类中没有这个方法 则去父类中查找 一直查找到父类的最上层
> 2. 如果父类的最上层也没有找到相对应的方法 则会报`unrecognized selector`但在这之前 仍有三次补救的机会
> 3. 第一次补救机会`Method resolution`: `runtime`会调用本类中`+resolveInstanceMethod:`或`+resolveClassMethod:`让你提供一个函数实现 如果你添加了目标函数 那`runtime`会重新启动一次消息发送的过程 否则则运行到下一步 消息转发
> 4. 第二次补救机会`Fast forwarding`: 如果目标对象实现了`-forwardingTargetForSelector:` `runtime`就会调用这个方法 根据这个方法返回的对象(不为nil或self)重启消息发送, 当然新的消息放松过程也会转移到返回的对象上面 否则则进行最后一次消息转发
> 5. 第三次补救机会`Normal forwarding`: 首先`runtime`会发送`-methodSignatureForSelector:`获得函数的参数及返回类型 如果 `-methodSignatureForSelector:` 返回了`nil` 这时程序就会崩溃 如果返回了一个函数签名 则`runtime`会生成一个`NSInvocation`对象并发送`-forwardInvocation:`给目标对象 (有些框架 例如`JSPatch` 需要利用第三步返回的`NSInvocation`自定义实现 而直接调用了 `objc_msgForward` 这样原来的方法就不会被执行 而是走了别的实现)

# 8.下面的代码输出什么
```cpp
  @implementation Son : Father
   - (id)init
   {
       self = [super init];
       if (self) {
           NSLog(@"%@", NSStringFromClass([self class])); // Son
           NSLog(@"%@", NSStringFromClass([super class])); // Son
       }
       return self;
   }
   @end
```
> 都是`Son` `super`和`self`指向同一个消息接收者`super`会在父类中调用而已 但他仍然是本类对象

# 9.objc 中类方法和实例方法有什么本质区别和联系
> 类方法:
1. 类方法是属于类对象的
2. 类方法只能由类对象调用
3. 类方法中的self是类对象
4. 类方法可以调用其他类的方法
5. 类方法中不能访问成员变量
6. 类方法中不能直接调用对象方法

> 实例方法:
1. 实例方法是属于实例对象的
2. 实例方法由对象实例调用
3. 实例方法中的self是实例对象
4. 实例方法中可以访问成员变量
5. 实例方法中可以调用实例方法
6. 实例方法也可以调用类方法(通过类名)

# 10.能否想编译后的类中添加实例变量 运行时呢?
> 不能向编译后的类中添加实例变量 运行时可以
> 因为编译后的类已经在`runtime`中注册类结构体中的实例变量链表`objc_Ivar_list`和实例变量内存`instance_size`大小已确定

> 运行时可以添加但的在类被创建但还没有向`runtime`注册之前`objc_allocateClassPair` - `objc_registerClassPair`区间内

# 11.测试
