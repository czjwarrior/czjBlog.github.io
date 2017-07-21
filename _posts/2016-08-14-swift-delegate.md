---
bg: "tools.jpg"
layout: post
title:  "Swift开发-代理"
crawlertitle: "Swift开发-代理"
summary: "Swift"
date:   2016-08-14 16:45:47 +0700
categories: blog
tags: ['2016-08']
author: 岑志军
---
在iOS开发中经常会用到代理,Swift开发中的代理这样写:

1、首先定义一个协议

```
// swift中如何定义协议: 必须遵守NSObjectProtocol
protocol VisitorViewDelegate : NSObjectProtocol{
    // 登录回调
    func loginBtnWillClick()
    // 注册回调
    func regiserBtnWillClick()

}
```

2、方法实现


```
func loginBtnClick(){
        delegate?.loginBtnWillClick()
    }

    func regiserBtnClick(){
        delegate?.regiserBtnWillClick()
    }
```

3、方法调用


```
private func setupVisitorView(){
        // 1、初始化未登录界面
        let customView = VisitorView()
        customView.delegate = self
        view = customView
        visitorView = customView

        // 2、设置导航条未登录按钮
//        navigationController?.navigationBar.tintColor = UIColor.orangeColor()

        navigationItem.leftBarButtonItem = UIBarButtonItem(title: "注册", style: UIBarButtonItemStyle.Plain, target: self, action: #selector(regiserBtnWillClick))
        navigationItem.rightBarButtonItem = UIBarButtonItem(title: "登录", style: UIBarButtonItemStyle.Plain, target: self, action: #selector(loginBtnWillClick))

    }

    func loginBtnWillClick() {
        print(#function)
    }

    func regiserBtnWillClick() {
        print(#function)
    }
```



