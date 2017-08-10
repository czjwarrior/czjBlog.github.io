---
layout: post
title: "Swift学习-函数"
date: 2016-08-11
tag: iOS
---

```
/* swift定义函数格式：
 语义：将前面计算结果返回给-> 返回值

 func 函数名称(参数列表) -> 返回值
 {
    执行代码
 }
 */

//没有返回值没有参数
// 如果函数没有返回值，就写Void
// 如果函数没有返回值，就写Void,还可以简写
// 1、()代替Void
// 2、可以省略 -> ()  ->Void

func say() -> Void {
    print("hello")
}

say()


func say2() -> () {
    print("hello")
}

say2()

func say3() {
    print("hello")
}

say3()

// 有返回值没有参数

func getNumber() -> Int {
    return 100;
}

print(getNumber())

// 有参数没有返回值
// swift2.0中，会自动将形参列表中的第二个参数开始的参数名称作为标签，以便于阅读

func sum(a: Int, b: Int) {
    print(a + b)
}

sum(8, b: 9)

// 添加标签，添加外部参数
// x/y称之为外部参数，a/b称之为内部参数

func sum2(x a: Int, y b: Int) {
    print(a + b)
}

sum2(x: 10, y: 20)


// 有参数有返回值

func sum4(a: Int, b: Int) -> Int {
    return a + b;
}

sum4(1, b: 3)
```



