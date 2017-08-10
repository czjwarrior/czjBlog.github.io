---
layout: post
title: "iOS判断运营商类型"
date: 2016-12-06
tag: iOS
---

一、获取运营类型

1、需要导入两个头文件


```
#import <CoreTelephony/CTCarrier.h>
#import <CoreTelephony/CTTelephonyNetworkInfo.h>
```

2、判断类型


```
// 获取运营商类型
+ (SSOperatorsType)getOperatorsType{
    CTTelephonyNetworkInfo *telephonyInfo = [[CTTelephonyNetworkInfo alloc] init];
    CTCarrier *carrier = [telephonyInfo subscriberCellularProvider];

    NSString *currentCountryCode = [carrier mobileCountryCode];
    NSString *mobileNetWorkCode = [carrier mobileNetworkCode];

    if (![currentCountryCode isEqualToString:@"460"]) {
        return SSOperatorsTypeOther;
    }

    // 参考 https://en.wikipedia.org/wiki/Mobile_country_code

    if ([mobileNetWorkCode isEqualToString:@"00"] ||
        [mobileNetWorkCode isEqualToString:@"02"] ||
        [mobileNetWorkCode isEqualToString:@"07"]) {

        // 中国移动
        return SSOperatorsTypeChinaMobile;
    }

    if ([mobileNetWorkCode isEqualToString:@"01"] ||
        [mobileNetWorkCode isEqualToString:@"06"] ||
        [mobileNetWorkCode isEqualToString:@"09"]) {

        // 中国联通
        return SSOperatorsTypeChinaUnicom;
    }

    if ([mobileNetWorkCode isEqualToString:@"03"] ||
        [mobileNetWorkCode isEqualToString:@"05"] ||
        [mobileNetWorkCode isEqualToString:@"11"]) {

        // 中国电信
        return SSOperatorsTypeTelecom;
    }

    if ([mobileNetWorkCode isEqualToString:@"20"]) {

        // 中国铁通
        return SSOperatorsTypeChinaTietong;
    }

    return SSOperatorsTypeOther;
}
```






