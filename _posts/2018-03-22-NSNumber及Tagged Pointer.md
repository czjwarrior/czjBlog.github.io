---
layout: post
title: "（2）NSNumber及Tagged Pointer"
date: 2018-03-22
tags: iOS底层
---

根据上一篇文章的总结，我们很容易发现

```
@interface Student : NSObject
{
    @public
    int _age;
    int _no;
}
```

一个Student对象在64位架构下占了16个字节，其中isa占8个字节，两个int变量分别占了4个字节，但是这种方式适合所有OC对象吗？？哈哈，并不是。。。

今天早上有朋友问NSNumber为啥占用8个字节（64bit），请看NSNumber头文件，发现如下代码：

```
@property (readonly) char charValue;
@property (readonly) unsigned char unsignedCharValue;
@property (readonly) short shortValue;
@property (readonly) unsigned short unsignedShortValue;
@property (readonly) int intValue;
@property (readonly) unsigned int unsignedIntValue;
@property (readonly) long longValue;
@property (readonly) unsigned long unsignedLongValue;
@property (readonly) long long longLongValue;
@property (readonly) unsigned long long unsignedLongLongValue;
@property (readonly) float floatValue;
@property (readonly) double doubleValue;
@property (readonly) BOOL boolValue;
@property (readonly) NSInteger integerValue API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
@property (readonly) NSUInteger unsignedIntegerValue API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
```

NSNumber对象里面有很多成员变量，为啥只占8个字节呢？？我们先做几个试验：


```
NSNumber *number1 = [NSNumber numberWithInt:9];
NSNumber *number2 = [NSNumber numberWithInt:9];
NSNumber *number3 = [NSNumber numberWithInt:5];
NSNumber *number4 = [NSNumber numberWithInt:6];
NSLog(@"\n%p\n%p\n%p\n%p", number1, number2,number3, number4);
```

下面是打印结果：

```
0xb000000000000093
0xb000000000000093
0xb000000000000052
0xb000000000000062
```

我们发现：

* number1和number2的对象地址竟然是一样的
* 这几个地址除了`0xb`和后面的`3`、`2`，其它的数刚好对应其NSNumber的值

所以苹果确实是将值直接存在了指针本身当中了

Google上发现一张NSNumber的内存图，很形象：

![NSNumbe](http://otogtitz7.bkt.clouddn.com/2018-03-22-NSNumber.png)


这就很有意思了，我尝试着打印下他们的ISA指针，发现报如下错误：

![](http://otogtitz7.bkt.clouddn.com/2018-03-22-15217046164134.jpg)


这是为什么呢，通过查找一些资料发现，[唐巧在很早前的一篇文章](https://blog.devtang.com/2014/03/21/weak_object_lifecycle_and_tagged_pointer/)中提到`Tagged Pointer`：一下是摘录：

> 在WWDC2013的《Session 404 Advanced in Objective-C》视频中，苹果介绍了Tagged Pointer。Tagged Pointer的存在主要是为了节省内存。我们知道，对象的指针大小一般是与机器字长有关，在32位系统中，一个指针的大小是32位（4字节），而在64位系统中，一个指针的大小将是64位（8字节）。

> 在64位系统中，如果我们真正使用一个指针来存储NSNumber实例，那么我们首先需要一个8字节的指针，另外需要一块内存存储NSNumber实例，这通常又是8字节。这样的内存开销是比较大的。苹果对于NSNumber和NSDate对象，改成了用Tagged Pointer来存储，简单来说，Tagged Pointer是一个假的指针，它的值不再是另一个地址，而就是对应变量的值。

> Tagged Pointer主要有以下3个特点：

> Tagged Pointer专门用来存储小的对象，例如NSNumber和NSDate
Tagged Pointer指针的值不再是地址了，而是真正的值。所以，实际上它不再是一个对象了，它只是一个披着对象皮的普通变量而已！所以，它的内存并不存储在堆中，也不需要malloc和free。
在内存读取上有着3倍的效率（以前是寻址->发消息->获取值，现在直接获取值），创建时比以前快106倍。

相关英文文档截图如下：

![](http://otogtitz7.bkt.clouddn.com/2018-03-22-15217049155699.jpg)

看了上面的文章，终于恍然大悟了，大神不愧是大神，这篇文章是14年初写的。。。

所以我们得出如下结论：
* `Tagged Pointer`并不是真正的对象,而是一个伪对象

因为`Tagged Pointer`不是一个真正的对象，所以当你访问它的ISA的时候自然就会报上面的错误了。


如果一个数超过了`Tagged Pointer`所能表示的范围，又会怎么处理呢？同样做个试验：

```
NSNumber *bigNumber = @(0xEFFFFFFFFFFFFFFF);
NSLog(@"%p", bigNumber);
```

打印结果：

```
0x6000002310c0
```

我们发现`bigNumber`更像一个普通的地址，跟他本身的值并没有什么关系，我们可以打印一下他的ISA，发现是可以打印的：

![](http://otogtitz7.bkt.clouddn.com/2018-03-22-15217063008204.jpg)

所以可以得出如下结论：

* 当8字节可以承载用于表示的数值时，系统就会以`Tagged Pointer`的方式生成指针，如果8字节承载不了时，则又用以前的方式来生成普通的指针。


References:

https://blog.devtang.com/2014/03/21/weak_object_lifecycle_and_tagged_pointer/

http://www.infoq.com/cn/articles/deep-understanding-of-tagged-pointer/




