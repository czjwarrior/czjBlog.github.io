---
layout: post
title: "（1）Objective-C的本质"
date: 2018-03-13
tags: iOS底层
---

众说周知，我们平时编写的OC代码，底层都是C/C++实现的

![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209130736084.jpg)

我们可以通过一个终端指令，将我们的OC代码转换成C/C++代码

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc 文件名 -o 输出的CPP文件
```

例如：

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main.cpp
```

## 思考一：OC的对象、类都是基于C/C++什么数据结构实现的？？

首先看下面代码：

```
#import <Foundation/Foundation.h>
#import <objc/runtime.h>

@interface Student : NSObject
{
    @public
    int _age;
    int _no;
}
@end

@implementation Student
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Student *stu = [[Student alloc] init];
        
        stu->_age = 4;
        stu->_no = 5;
        
        NSLog(@"%@", stu);
        
        NSLog(@"%zd", class_getInstanceSize([NSObject class]));
        NSLog(@"%zd", class_getInstanceSize([Student class]));
    }
    return 0;
}
```

通过终端命令生成.cpp文件

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main.cpp
```

我们发现Student和NSObject的底层实现代码如下：

```
struct Student_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	int _age;
	int _no;
};

struct NSObject_IMPL {
	Class isa;
};
```

所以，OC的对象、类都是基于C/C++当中结构体实现的。

那么，一个NSObject对象占用多少内存呢？

通过以上代码，我们发现一个NSObject对象占用的内存大小是一个指针变量所占用的大小（64bit，8个字节。32bit，4个字节）

同样可以通过代码检验

方法一：通过runtime方法检验

```
NSLog(@"%zd", class_getInstanceSize([NSObject class]));
```
终端打印结果：

```
2018-03-13 12:02:26.584356+0800 TestDemo[33726:1665977] 8
```

方法二：实时查看内存数据

```
Debug -> Debug Workfllow -> View Memory （Shift + Command + M）
```
![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209139194113.jpg)

![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209142507883.jpg)

输入内存地址：
![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209142877899.jpg)

同样发现一个Student占16个字节，其中指针占了8个字节

方法三：可以通过lldb命令查看

常用lldb命令
![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209143991235.jpg)

查看结果如下：

```
(lldb) x/4xw 0x102c0a590
0x102c0a590: 0x000011c9 0x001d8001 0x00000004 0x00000005
```

还可以通过lldb命令修改对象的值：
![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209148405061.jpg)

## 类、实例对象、元类（class、instance、meta-class）

### 类(class)
**类**：类是现实世界或思维世界中的实体在计算机中的反映，它将数据以及这些数据上的操作封装在一起(百科上的回答)。简单的说就是数据及行为的封装

通过查阅 Apple 官方开源的 [objc 源码](https://opensource.apple.com/tarballs/objc4/)，可以看到类的数据结构如下：

```
struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

    class_rw_t *data() { 
        return bits.data();
    }
    void setData(class_rw_t *newData) {
        bits.setData(newData);
    }

    ...
    ...
    ...（省略）
}
```


```
struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    method_array_t methods; // 方法信息
    property_array_t properties;    // 属相信息
    protocol_array_t protocols;     // 协议信息

    Class firstSubclass;
    Class nextSiblingClass;
    
    ...
    ...
    ...（省略）
```


```
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;  // 对象占用的内存大小
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;
    
    const char * name;  // 类名
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;  // 成员变量列表

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};
```

通过以上代码我们发现Class对象在内存中存储的信息主要包括：
* isa指针
* superclass指针
* 属性信息
* 协议信息

同时我们发现objc_class 继承自 objc_object，哈哈，其实类也是一个对象。。。


```
NSObject *obj1 = [[NSObject alloc] init];
        NSObject *obj2 = [[NSObject alloc] init];
        
        Class objClass1 = [obj1 class];
        Class objClass2 = [obj2 class];
        Class objClass3 = [NSObject class];
        Class objClass4 = object_getClass(obj1);
        Class objClass5 = object_getClass(obj2);
        
        NSLog(@"\n%p\n%p\n%p\n%p\n%p\n%p\n%p\n", obj1, obj2, objClass1, objClass2, objClass3, objClass4, objClass5);
```

![](http://otogtitz7.bkt.clouddn.com/2018-03-19-15214463000418.jpg)

通过以上试验，我们发现，NSObject生成两个实例对象obj1、obj2，这两个实例对象分布在不同的内存地址，但是他们的Class指针是一样的，所以我们得出以下结论：
* objClass1 ~ objClass5都是NSObject的class对象（类对象）
* 它们是同一个对象。每个类在内存中有且只有一个class对象

### 实例对象(instance)
**对象**：对象是具有类类型的变量（百科）。其实对象就是一个类的具体实例。在 Objective-C 中，含有一个 isa 指针并且可以正确指向某个类的数据结构，都可以视作为一个对象，其中 isa 指针指向当前对象所属的类，通过苹果开源的官方文档，同样可以发现它的数据结构，如下代码：

```
struct objc_object {
private:
    isa_t isa;

public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();

    // initIsa() should be used to init the isa of new objects only.
    // If this object already has an isa, use changeIsa() for correctness.
    // initInstanceIsa(): objects with no custom RR/AWZ
    // initClassIsa(): class objects
    // initProtocolIsa(): protocol objects
    // initIsa(): other objects
    void initIsa(Class cls /*nonpointer=false*/);
    void initClassIsa(Class cls /*nonpointer=maybe*/);
    void initProtocolIsa(Class cls /*nonpointer=maybe*/);
    void initInstanceIsa(Class cls, bool hasCxxDtor);
    
    ...
    ...
    ...（省略）
}
```

其实上面代码中obj1、obj2就是NSObject生成的两个实例对象，不同的对象分别占据两块不同的内存。

通过查看对象的底层代码，同样可以发现，对象在内存中的存储信息包含：
* isa指针
* 其它成员变量的值等

就比如多个对象存在相同的属性，但是属性的值却存在不同的对象当中。

### 元类（meta-class）
**元类**：元类其实就是描述类对象的类。简单的说就是类描述的是对象，而元类描述的是类。所以元类也定义了类的行为（类方法），其实元类的数据结构和类基本相同，只不过元类定义类的行为是类方法（+），而对象是对象方法（-）。原理都是遍历方法列表或者缓存列表

```
Class objMetaClass1 = objc_getMetaClass("NSObject");
Class objMetaClass2 = object_getClass([NSObject class]);
```
![](http://otogtitz7.bkt.clouddn.com/2018-03-19-15214483111601.jpg)

objMetaClass1、objMetaClass2就是NSObject的meta-class对象（元类对象）

每个类在内存中有且只有一个meta-class对象

meta-class对象和class对象的内存结构是一样的，但是用途不一样，在内存中存储的信息主要包括：
* isa指针
* superclass指针
* 类的类方法信息（class method）等

* 查看class是否为meta-class

```
BOOL result = class_isMetaClass([NSObject class]);
```

## 方法的调用流程

通过以上信息我们就了解到了类、对象、元类之间的关系，那么类方法和对象方法的调用过程是怎样的呢？？如下图所示：
![](http://otogtitz7.bkt.clouddn.com/2018-03-19-15214487235189.jpg)

instance的isa指向class：
当调用对象方法时，通过instance的isa找到class，最后找到对象方法的实现进行调用

class的isa指向meta-class：
当调用类方法是，通过class的isa找到meta-class,最后找到类方法的实现进行调用

## superclass的作用

当一个对象调用父类方法时，其实就是通过isa找到class，然后通过superclass找到父类的class，最后找到对象方法的实现进行调用（类方法调用也是这个原理，通过isa找到meta-class,然后通过superclass找到父类的meta-class，最后找到类对象的实现进行调用）

## isa和superclass的调用流程

[Greg Parker的一份精彩图谱](http://www.sealiesoftware.com/blog/class%20diagram.pdf)

![](http://otogtitz7.bkt.clouddn.com/2018-03-19-15214498988799.jpg)

通过上图可以总结如下：

* instance的isa指向class
* class的isa指向meta-class
* meta-class的isa指向基类的meta-class

* class的superclass指向父类的class
* 如果没有父类，superclass指针为nil
* meta-class的superclass指向父类的meta-class
* 基类的meta-class的superclass指向基类的class

* instance调用对象方法的轨迹
    * isa找到class，方法不存在，就通过superclass找父类
* class调用类方法的轨迹
    * isa找meta-class，方法不存在，就通过superclass找父类









