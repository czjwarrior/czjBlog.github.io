---
layout: post
title: "（1）Objective-C的本质"
date: 2018-03-13
tags: iOS底层
---

众说周知，我们平时编写的OC代码，底层都是C/C++实现的

![](http://otogtitz7.bkt.clouddn.com/2018-03-13-15209130736084.jpg)

我们可以通过一个终端指令，将我们的OCdiamante转换成C/C++代码

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc 文件名 -o 输出的CPP文件
```

例如：

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main.cpp
```

思考一：OC的对象、类都是基于C/C++什么数据结构实现的？？

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










