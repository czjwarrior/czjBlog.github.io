---
layout: post
title: "iOS开发 UIlabel 文字两边对齐"
date: 2016-09-28 
tag: iOS
---

1、给UIlabel添加一个分类即可，代码如下：

- 必须导入这个头文件：CoreText/CoreText.h


```
- (void)changeAlignmentRightandLeft{

    CGRect textSize = [self.text boundingRectWithSize:CGSizeMake(self.frame.size.width, MAXFLOAT) options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingTruncatesLastVisibleLine |NSStringDrawingUsesFontLeading attributes:@{NSFontAttributeName : self.font} context:nil];

    CGFloat margin = (self.frame.size.width - textSize.size.width) / (self.text.length - 1);

    NSNumber *number = [NSNumber numberWithFloat:margin];
    NSMutableAttributedString *attributeString = [[NSMutableAttributedString alloc]initWithString:self.text];
    [attributeString addAttribute:(id)kCTKernAttributeName value:number range:NSMakeRange(0, self.text.length - 1)];
    self.attributedText = attributeString;

}
```

效果如下：

![](/images/posts/label-align.jpg)





