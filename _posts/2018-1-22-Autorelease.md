# Autoreleasepool

## autoreleasepool结构

autoreleasepool实际上是由一串AutoreleasePoolPage组成的链式结构。

page的结构是：

![image](http://upload-images.jianshu.io/upload_images/809311-fa45561b82906a98.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/541)

每个AutoreleasePoolPage的大小都是4096字节，除去7字节存储成员变量，其他部分都是用来存放Autoreleased对象的指针。在 AutoreleasePool Push的时候系统会初始化一个AutoreleasePoolPage，填充相应的字段。

```Objective-c
@autoreleasepool {
    //code
}
```
会被翻译成
```
void *context = objc_autoreleasePoolPush();
// code
objc_autoreleasePoolPop(context);
```
objc_autoreleasePoolPush()创建并返回AutoreleasePoolPage，objc_autoreleasePoolPush()释放page引用的所有autoreleased对象。

## autorelease

autorelease与autoreleasepool关系密切，被autorelease的对象会被加入autoreleasepool。

autorelease是在ARC和MRC中都存在的机制（ARC中不会显示调用autorelease，只是使用autoreleasePool）。

autorelease实际上就是延迟release。

## autorelease和autoreleasepool的用法

1. 工厂方法初始化时需要让alloc出来的对象有超过方法作用域的存活时间，使用autorelease延迟释放。
    
    例：
    ```
    + (*MyObject)objectWithNone {
        MyObject *object = [MyObject new];
        return [object autorelease];
    }
    ```
    
    注意：swift不建议使用工厂方法（苹果已经把cocoa所有工厂方法API改了），不过你依然可以在你的类中使用工厂方法，这时ARC也会自动为你加上autorelease。

        
2. 函数内部循环初始化对象，占用过多内存时。使用@autoreleasepool{}把相关代码括起来。==（new出来的应该是非autorelesed对象吧？@autorelease结束时为什么会释放非autoreleased对象？）==

## autoreleasepool创建与释放时机

如图：

![image](https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=3512129242,1639248542&fm=27&gp=0.jpg)

简单说就是在handle event前创建，handle event后释放autoreleasepool。（其实我们写的代码大都是在handle event内运行的，autoreleasepool的创建和释放逻辑在UIKit框架内）

### 测试一下

证明很简单，只要看对象能否在一个runloop循环内的两个函数作用域不被释放即可

网上很多人引用了[黑幕背后的Autorelease](https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjViIbb1pjYAhUBVbwKHYUnCZcQFggnMAA&url=http%3A%2F%2Fblog.sunnyxx.com%2F2014%2F10%2F15%2Fbehind-autorelease%2F&usg=AOvVaw0gUw0zsR44ajoSvJH71TCw)这篇博文的例子（viewDidLoad里创建的对象在viewWillAppear方法里依然没有被释放）做证明。然而我在模仿实验时发现结果不符，弱引用的对象在viewWillAppear时已经被释放。网上查了下似乎iphone5前的设备才会出现预期结果。我的xcode已经没有iphone5模拟器，所以我换了一个测试程序：

注：如果用NSString对象测试的话会发现对象一直没被释放，推测原因是字符串会被优化成宏或者静态c字符串，实际上已经不是对象了，自然也不存在引用计数了。

```Objective-c
@implementation ViewController {
    __weak NSArray *_weak_0;
    __weak NSArray *_weak_1;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    [self method];
    NSLog(@"weak_0: %@", _weak_0);
    NSLog(@"weak_1: %@", _weak_1);
}

- (void)method {
    NSArray *array_0 = [NSArray arrayWithObjects:@1, @3, nil];
    NSArray *array_1 = [[NSArray alloc] initWithObjects:@1, @3, nil];
    _weak_0 = array_0;
    _weak_1 = array_1;
}
```

## 子线程的autoreleasepool

子线程默认不开启runloop，也就没有autoreleasepool。这种情况下创建autoreleased对象时会调用autoreleaseNoPage方法。在这个方法中，会自动帮你创建一个hotpage，并调用page->add(obj)将对象添加到autoreleasePoolPage 的栈中。具体参考[does NSThread create autoreleasepool automaticly now?](https://link.jianshu.com/?t=http://stackoverflow.com/questions/24952549/does-nsthread-create-autoreleasepool-automaticly-now)