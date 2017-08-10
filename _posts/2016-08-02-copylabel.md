---
layout: post
title: "对UILabel添加UIMenuController扩展"
date: 2016-08-02 
tag: iOS
---
## 一、UIMenuController认识 
1、默认情况下，UITextView / UITextFiled / UIWebView 都有苹果自带的有UIMenuController功能
## 二、对UILabel添加UIMenuController扩展
2、新建一个SSCopyLabel，继承UIlabel，.m文件如下：


```
#import "SSCopyLabel.h"

@implementation SSCopyLabel

- (instancetype)initWithFrame:(CGRect)frame{
    self = [super initWithFrame:frame];
    if (self) {
        self.userInteractionEnabled = YES;
        UILongPressGestureRecognizer *touch = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(handleTap:)];
        [self addGestureRecognizer:touch];
    }
    return self;
}

-(void)handleTap:(UIGestureRecognizer*) recognizer {

    [self becomeFirstResponder];
    // 1.获得菜单 menu
    UIMenuController *menu = [UIMenuController sharedMenuController];
    // 2.设置菜单最终显示的位置
    [menu setTargetRect:self.frame inView:self.superview];
    UIMenuItem *menuItem = [[UIMenuItem alloc] initWithTitle:@"粘贴" action:@selector(pasteAction)];

    menu.menuItems = [NSArray arrayWithObjects:menuItem, nil];
    // 当label有内容的时候，再添加一个UIMenuItem
    if (self.text.length > 0) {
        UIMenuItem *menuItem1 = [[UIMenuItem alloc] initWithTitle:@"拷贝" action:@selector(copyAction)];
        menu.menuItems = [NSArray arrayWithObjects:menuItem, menuItem1, nil];
    }
    // 让UIMenuController显示出来，第二个参数不能直接写YES，否则会导致UIMenuController不断地闪烁
    [menu setMenuVisible:YES animated:!menu.isMenuVisible];
}

- (void)pasteAction{
    UIPasteboard *pBoard = [UIPasteboard generalPasteboard];

    if (pBoard.string != nil) {
        self.text = pBoard.string;
    }
}

- (void)copyAction{
    UIPasteboard *pBoard = [UIPasteboard generalPasteboard];
    pBoard.string = self.text;
}

- (BOOL)canBecomeFirstResponder{
    return YES;
}
```
demo示例： 
没内容的时候：
![](/images/posts/2016-08-02-demo1.jpg)

有内容的时候： 
![](/images/posts/2016-08-02-demo2.jpg)

