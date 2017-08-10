---
layout: post
title: " AFN拦截重定向设置httpBody"
date: 2016-12-06
tag: iOS
---

1、拦截重定向获取里面的cookie

```
AFHTTPRequestOperation *requestOperation=[[AFHTTPRequestOperation alloc] initWithRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@""]]];
    [requestOperation setRedirectResponseBlock:^NSURLRequest *(NSURLConnection *connection, NSURLRequest *request, NSURLResponse *redirectResponse) {
        if (redirectResponse) {

            NSHTTPURLResponse *response = (NSHTTPURLResponse *)redirectResponse;

            NSString *cookieString = [[response allHeaderFields] valueForKey:@"Set-Cookie"];

            [self getMobileHtmlByCookie:cookieString];

        }
        return request;
    }];
    [requestOperation start];
```

2、设置body


```
AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    NSString *requestUrlStr = @"";
    NSMutableURLRequest *req = [[AFHTTPRequestSerializer serializer] requestWithMethod:@"POST" URLString:finalRequestStr parameters:nil error:nil];
    [req setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];

    // 设置postBody
    [req setHTTPBody:postBody];

    [[manager dataTaskWithRequest:req completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {

        if (!error) {
             // 返回数据成功

        } else {
            // 解析失败
        }
    }] resume];
```




