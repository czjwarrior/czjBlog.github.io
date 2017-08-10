---
layout: post
title: "TTTAttributedLabel高亮显示手机号码、网址"
date: 2017-05-12
tag: iOS
---

1、初始化label

```
- (TTTAttributedLabel *)traceLabel{
    if (_traceLabel == nil) {
        _traceLabel = [TTTAttributedLabel new];
        [_traceLabel setTextAlignment:NSTextAlignmentLeft];

        // NSTextCheckingTypeLink
        // 设置识别类型
        _traceLabel.enabledTextCheckingTypes = NSTextCheckingTypePhoneNumber;

        [_traceLabel setLinkAttributes:@{NSForegroundColorAttributeName:SS_CUSTOM_DARK_BLUE_COLOR, NSUnderlineStyleAttributeName:@(0)}];

        //链接高亮状态文本属性
        [_traceLabel setActiveLinkAttributes:@{NSForegroundColorAttributeName:[SS_CUSTOM_DARK_BLUE_COLOR colorWithAlphaComponent:.6f],NSUnderlineStyleAttributeName:@(0)}];
        [_traceLabel setUserInteractionEnabled:YES];
        [_traceLabel setDelegate:self];
        [_traceLabel setNumberOfLines:0];
    }
    return _traceLabel;
}
```

2、设置文字

```
NSMutableAttributedString *attStr = [NSMutableAttributedString attributedStringWithFont:SS_NORMAL_FONT_WITH_6P(13, 16)
                                                                                  textColor:traceColor
                                                                                  lineSpace:SS_ADAPT_FLOAT_WITH_6P(8, 9)
                                                                              lineBreakMode:NSLineBreakByWordWrapping
                                                                              textAlignment:NSTextAlignmentLeft
                                                                                       text:traceModel.status];
// 这里必须用setText:方法，如果用setAttributedText:高亮无效                                                                                
[self.traceLabel setText:attStr];
```




