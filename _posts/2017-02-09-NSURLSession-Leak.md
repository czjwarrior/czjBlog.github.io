---
layout: post
title: "NSURLSession内存泄漏"
date: 2017-02-09
tag: iOS
---

检查代码是否有leak的时候，发现NSURLSession存在leak，最后发现必须session请求完成后，立即释放，代码如下：

```
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error{
    [session finishTasksAndInvalidate];
}
```




