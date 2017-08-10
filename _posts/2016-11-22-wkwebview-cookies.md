---
layout: post
title: "清除WKWebView cookies"
date: 2016-11-22
tag: iOS
---

在UIWebView下，可以使用


```
[[NSURLCache sharedURLCache] removeAllCachedResponses];//清除缓存 
```
WKWebView清除cookies的方法（iOS9以上）

```
WKWebsiteDataStore *dateStore = [WKWebsiteDataStore defaultDataStore];  
    [dateStore fetchDataRecordsOfTypes:[WKWebsiteDataStore allWebsiteDataTypes]  
                     completionHandler:^(NSArray<WKWebsiteDataRecord *> * __nonnull records) {  
                         for (WKWebsiteDataRecord *record  in records)  
                         {  
//                             if ( [record.displayName containsString:@"baidu"]) //取消备注，可以针对某域名清除，否则是全清  
//                             {  
                                 [[WKWebsiteDataStore defaultDataStore] removeDataOfTypes:record.dataTypes  
                                                                           forDataRecords:@[record]  
                                                                        completionHandler:^{  
                                                                            NSLog(@"Cookies for %@ deleted successfully",record.displayName);  
                                                                        }];  
//                             }  
                         }  
                     }];  
```

iOS9一下用这种方法：

```
NSString *libraryPath = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES) objectAtIndex:0];  
NSString *cookiesFolderPath = [libraryPath stringByAppendingString:@"/Cookies"];  
NSError *errors;  
[[NSFileManager defaultManager] removeItemAtPath:cookiesFolderPath error:&errors]; 
```

查看cookie


```
NSHTTPCookie *cookie;
NSHTTPCookieStorage *cookieJar = [NSHTTPCookieStorage sharedHTTPCookieStorage];
for (cookie in [cookieJar cookies]) {
   NSLog(@"%@", cookie);
}
```

参考链接：http://stackoverflow.com/questions/31289838/how-to-delete-wkwebview-cookies




