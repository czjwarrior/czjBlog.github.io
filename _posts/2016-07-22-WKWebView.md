---
bg: "tools.jpg"
layout: post
title:  "WKWebView常见功能及如何返回上级界面"
crawlertitle: "WKWebView常见功能及如何返回上级界面"
summary: ""
date:   2016-07-22 14:12:47 +0700
categories: blog
tags: ['2016-07']
author: 岑志军
---

1、WKWebView的简单初始化

{% highlight js %}
- (WKWebView *)webView{
    if (_webView == nil) {
        _webView = [[WKWebView alloc] initWithFrame:self.sContentView.bounds];
        [_webView setAutoresizingMask:UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight];
        [_webView setNavigationDelegate:self];
        [_webView setUIDelegate:self];
        [_webView setMultipleTouchEnabled:YES];
        [_webView setAutoresizesSubviews:YES];
        [_webView.scrollView setAlwaysBounceVertical:YES];
        // 这行代码可以是侧滑返回webView的上一级，而不是根控制器（*只针对侧滑有效）
        [_webView setAllowsBackForwardNavigationGestures:true];
    }
    return _webView;
}
{% endhighlight %}
至于如何加载webView用法和UIWebViewle类似，自行百度，下面介绍r如何返回上一层，代码结合ReactiveCocoa,[ReactiveCocoa的简单使用](http://www.jianshu.com/p/87ef6720a096%20)

{% highlight js %}
@weakify(self)
    // 返回按钮
    [self.baseView.navView addSubview:self.backBtn];
    [[self.backBtn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {
        @strongify(self)
        if ([self.webView canGoBack]) {
            [self.webView goBack];
        } else {
            [self.viewModel.services popViewModelAnimated:YES];
        }
    }];
{% endhighlight %}
如果要做到类似于微信里面的返回上一级出现有好的提示，可以采用ReactiveCocoa非常牛逼的监听机制：

{% highlight js %}
// 绑定关闭按钮
    RAC(self.baseView.popBtn, hidden) = [RACObserve(self.webView, canGoBack) map:^id(NSNumber *canGoBackNum) {
        @strongify(self)
        if (canGoBackNum.boolValue) {
            [self.backBtn setTitle:@"返回" forState:UIControlStateNormal];
        } else {
            [self.backBtn setTitle:@"" forState:UIControlStateNormal];
        }
        [self.backBtn sizeToFit];

        return @(!canGoBackNum.boolValue);
    }];
{% endhighlight %}
现在好多APP都会在导航栏下方添加进度条，提醒用户webView的加载进度，这在WKWebView中实现起来也非常简单，只需自定义UIProgressView即可：

{% highlight js %}
// 监听进度
    [RACObserve(self.webView, estimatedProgress) subscribeNext:^(id x) {
        @strongify(self)

        [self.progressView setAlpha:1.0f];

        BOOL animated = self.webView.estimatedProgress > self.progressView.progress;
        [self.progressView setProgress:self.webView.estimatedProgress animated:animated];

        if(self.webView.estimatedProgress >= 1.0f) {
            [UIView animateWithDuration:0.3f delay:0.3f options:UIViewAnimationOptionCurveEaseOut animations:^{
                [self.progressView setAlpha:0.0f];

            } completion:^(BOOL finished) {
                [self.progressView setProgress:0.0f animated:NO];
            }];
        }
    }];
{% endhighlight %}




