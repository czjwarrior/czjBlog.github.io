---
layout: post
title: "UITextField添加点击高亮状态"
date: 2016-11-16 
tag: iOS
---

一、继承自UITextfield自定义一个SSTouchTextField 

代码如下：

```
#import "SSTouchTextField.h"

@implementation SSTouchTextField

#pragma mark - Private

- (void)setBackgroundHighlighted:(BOOL)highlighted{
    [UIView animateWithDuration:0.3f delay:0.f options:UIViewAnimationOptionCurveEaseInOut animations:^{
        if (highlighted) {
            [self setBackgroundColor:SS_LINE_COLOR];

        } else {
            [self setBackgroundColor:[UIColor clearColor]];
        }

    } completion:^(BOOL finished) {

    }];
}

#pragma mark - Overwrite

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event{
    [super touchesBegan:touches withEvent:event];
    [self setBackgroundHighlighted:YES];
}

- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event{
    [super touchesMoved:touches withEvent:event];
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event{
    [super touchesEnded:touches withEvent:event];
    [self setBackgroundHighlighted:NO];
}

- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event{
    [super touchesCancelled:touches withEvent:event];
    [self setBackgroundHighlighted:NO];
}

// 增大点击区域
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{

    CGPoint convertPoint = [self convertPoint:point toView:self.superview];

    UIEdgeInsets edgeInset = UIEdgeInsetsMake(0, 100, 0, 100);
    CGRect edgeFrame = CGRectMake(self.view_minX - edgeInset.left,
                                  self.view_minY - edgeInset.top,
                                  self.view_width + edgeInset.left + edgeInset.right,
                                  self.view_height + edgeInset.top + edgeInset.bottom);

    CGRect convertFrame = [self.superview convertRect:edgeFrame toView:self.superview];
    if (CGRectContainsPoint(convertFrame, convertPoint)) {

        return self;
    }

    return [super hitTest:point withEvent:event];
}

@end
```


