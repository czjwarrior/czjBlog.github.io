---
layout: post
title: "iOS获取当前网络环境"
date: 2016-12-06
tag: iOS
---


```
// 获取网络环境的方法
+ (NSString *)networktype{
    NSArray *subviews = [[[[UIApplication sharedApplication] valueForKey:@"statusBar"] valueForKey:@"foregroundView"]subviews];
    NSNumber *dataNetworkItemView = nil;

    for (id subview in subviews) {
        if([subview isKindOfClass:[NSClassFromString(@"UIStatusBarDataNetworkItemView") class]]) {
            dataNetworkItemView = subview;
            break;
        }
    }

    switch ([[dataNetworkItemView valueForKey:@"dataNetworkType"]integerValue]) {
        case 0:
            return  @"无服务";

        case 1:
            return @"2G";

        case 2:
            return @"3G";

        case 3:
            return @"4G";

        case 4:
            return @"LTE";

        case 5:
            return @"Wifi";


        default:
            break;
    }
    return @"";
}
```

