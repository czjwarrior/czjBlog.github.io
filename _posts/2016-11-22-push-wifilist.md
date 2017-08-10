---
layout: post
title: " 解决iOS10不能跳转系统WiFi列表的问题"
date: 2016-11-22 
tag: iOS
---

第一种方式：

在iOS10更新后，系统设置跳转被禁用，只能跳转App设置，但是最近发现苹果又更新了URLscheme，亲测可用，建议iOS10已下，还用原来的scheme


```
#define iOS10 ([[UIDevice currentDevice].systemVersion doubleValue] >= 10.0)
NSString * urlString = @"App-Prefs:root=WIFI";
if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:urlString]]) {
    if (iOS10) {
        [[UIApplication sharedApplication] openURL:[NSURL URLWithString:urlString] options:@{} completionHandler:nil];
    } else {
        [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=WIFI"]];
    }
}
```

第二种方式： 
用到了私有API，慎用，若想使用并通过审核，可以对私有方法名等加密


```
NSURL*url=[NSURL URLWithString:@"Prefs:root=WIFI"];
    Class LSApplicationWorkspace = NSClassFromString(@"LSApplicationWorkspace");
    [[LSApplicationWorkspace performSelector:@selector(defaultWorkspace)] performSelector:@selector(openSensitiveURL:withOptions:) withObject:url withObject:nil];
```
附录：iOS10之后，其它界面的跳转


```
当前iOS10支持的所有跳转,亲测可用（测试系统：10.2.1）

跳转  写法
无线局域网   App-Prefs:root=WIFI
蓝牙  App-Prefs:root=Bluetooth
蜂窝移动网络  App-Prefs:root=MOBILE_DATA_SETTINGS_ID
个人热点    App-Prefs:root=INTERNET_TETHERING
运营商 App-Prefs:root=Carrier
通知  App-Prefs:root=NOTIFICATIONS_ID
通用  App-Prefs:root=General
通用-关于本机 App-Prefs:root=General&path=About
通用-键盘   App-Prefs:root=General&path=Keyboard
通用-辅助功能 App-Prefs:root=General&path=ACCESSIBILITY
通用-语言与地区    App-Prefs:root=General&path=INTERNATIONAL
通用-还原   App-Prefs:root=Reset
墙纸  App-Prefs:root=Wallpaper
Siri    App-Prefs:root=SIRI
隐私  App-Prefs:root=Privacy
Safari  App-Prefs:root=SAFARI
音乐  App-Prefs:root=MUSIC
音乐-均衡器  App-Prefs:root=MUSIC&path=com.apple.Music:EQ
照片与相机   App-Prefs:root=Photos
FaceTime    App-Prefs:root=FACETIME
```






