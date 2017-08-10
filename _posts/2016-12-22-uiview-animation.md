---
layout: post
title: "iOS动画-定时对UIView进行翻转和抖动"
date: 2016-12-22
tag: iOS
---

（翻转）方式一：

```
[NSTimer scheduledTimerWithTimeInterval:3.f repeats:YES block:^(NSTimer * _Nonnull timer) {
            CABasicAnimation* rotationAnimation = [CABasicAnimation animation];;
            rotationAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.y"];
            rotationAnimation.toValue = [NSNumber numberWithFloat: M_PI * 2.0 ];
            rotationAnimation.duration = 1;
            // 切换界面保证动画不停止
            rotationAnimation.removedOnCompletion = NO;
            rotationAnimation.repeatCount = 1;
            [self.bindCardImageView.layer addAnimation:rotationAnimation forKey:@"rotationAnimation"];
        }];
```

（翻转）方式二（这种方式较好一些）

```
CABasicAnimation *waitAnimation = [CABasicAnimation animation];
        waitAnimation.toValue = [NSNumber numberWithFloat:1.0];
        waitAnimation.duration = 3.f;
        waitAnimation.beginTime = 3.f;

        CABasicAnimation* rotationAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.y"];
        rotationAnimation.toValue = [NSNumber numberWithFloat: M_PI * 2.0 ];
        rotationAnimation.duration = 1.f;

        CAAnimationGroup *group = [CAAnimationGroup animation];
        group.duration = 4.f;
        group.repeatCount = CGFLOAT_MAX;
        group.removedOnCompletion = NO;

        [group setAnimations:@[waitAnimation, rotationAnimation]];

        [self.bindCardImageView.layer addAnimation:group forKey:@"bindCardImageViewAnimation"];
```

抖动：

```
CABasicAnimation* shake = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    //设置抖动幅度
    shake.fromValue = [NSNumber numberWithFloat:-0.2];
    shake.toValue = [NSNumber numberWithFloat:+0.2];
    shake.duration = 0.1;
    shake.autoreverses = YES; //是否重复
    shake.repeatCount = 3;

    [itemView.iconImageView.layer addAnimation:shake forKey:@"imageView"];
```



