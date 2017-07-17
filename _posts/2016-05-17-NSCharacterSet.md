---
bg: "tools.jpg"
layout: post
title:  "NSCharacterSet 简单用法"
crawlertitle: "NSCharacterSet 简单用法"
summary: ""
date:   2016-05-14 20:31:47 +0700
categories: blog
tags: ['2016-05']
author: 岑志军
---
[转载来源](http://blog.sina.com.cn/s/blog_70854cfb01013ux4.html)
NSCharacterSet其实是许多字符或者数字或者符号的组合，在网络处理的时候会用到


```
NSMutableCharacterSet *base = [NSMutableCharacterSet lowercaseLetterCharacterSet]; //字母
  NSCharacterSet *decimalDigit = [NSCharacterSet decimalDigitCharacterSet];   //十进制数字
  [base formUnionWithCharacterSet:decimalDigit];    //字母加十进制
  NSString *string = @"ax@d5s#@sfn$5`SF$$%x^(#e{]e";
  //用上面的base隔开string然后组成一个数组，然后通过componentsJoinedByString，来连接成一个字符串
  NSLog(@"%@",[[string componentsSeparatedByCharactersInSet:base] componentsJoinedByString:@"-"]);
  [base invert];  //非 字母加十进制
  NSLog(@"%@",[[string componentsSeparatedByCharactersInSet:base] componentsJoinedByString:@"-"]);
```
答应结果：

`ax@d-s#@sfn−‘SF$%x^(#e{]e`

判断字符串是否是纯数字

```
- (BOOL)isPureNumandCharacters:(NSString *)string
{
    string = [string stringByTrimmingCharactersInSet:[NSCharacterSet decimalDigitCharacterSet]];
    if(string.length > 0)
    {
        return NO;
    }
    return YES;
}
```

