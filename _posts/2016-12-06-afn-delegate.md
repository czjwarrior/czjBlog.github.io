---
layout: post
title: "AFN挂代理访问"
date: 2016-12-06
tag: iOS
---


```
AFHTTPSessionManager *manager = [[AFHTTPSessionManager alloc] initWithBaseURL:@"" sessionConfiguration:[self setProxyWithConfig]];
    [manager.requestSerializer setValue:@"MQQBrowser" forHTTPHeaderField:@"User-Agent"];
    [manager.responseSerializer setAcceptableContentTypes:[NSSet setWithObjects:@"text/html", nil]];
    manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    [manager.requestSerializer setValue:cookieStr forHTTPHeaderField:@"Cookie"];

    [manager GET:@"" parameters:nil success:^(NSURLSessionDataTask * _Nonnull task, NSData * responseObject) {

        NSData *data = [[NSData alloc] initWithData:responseObject];

        NSString *dataStr = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];



    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {

    }];
```


```
// 设置代理
- (NSURLSessionConfiguration *)setProxyWithConfig
{
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
    config.requestCachePolicy = NSURLRequestReloadIgnoringLocalCacheData;
    config.connectionProxyDictionary = @
    {
        @"HTTPEnable":@YES,
        (id)kCFStreamPropertyHTTPProxyHost:@"192.168.1.1",
        (id)kCFStreamPropertyHTTPProxyPort:@80,
        @"HTTPSEnable":@YES,
        (id)kCFStreamPropertyHTTPSProxyHost:@"192.168.1.1",
        (id)kCFStreamPropertyHTTPSProxyPort:@80
    };

    return config;
}
```





