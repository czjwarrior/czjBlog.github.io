---
layout: post
title: "iOS模仿系统相机拍照你不曾注意过的细节"
date: 2018-07-01
tags: iOS
---

> 距离上次写博客竟然过了一个月了，一方面是最近项目比较忙，另一方面是实在是有点儿懈怠了，强烈谴责一下自己。其实我最近在看一些技术书籍，发现一些好的书真心对自己帮助很大，看书的过程，好多原来模糊的概念、问题，都能感觉恍然大悟。当提笔想总结成一篇文章的时候，发现网上早已经有大量的优秀文章出现，所以就不敢献丑了。今天写的一篇文章，是最近自己项目中用到的，不算什么难点，只是感觉有必要记录一下。


## 需求

由于我们APP集成了有道翻译的SDK，需要将拍出来的图片翻译成对应的语言，但是有道的SDK目前还做的不是很完善（比如：照片倾斜的时候，返回的角度不是很对，有道的技术说下个版本可能会更新）。于是产品要求拍照页面做成跟系统相机类似，当用户横屏拍摄的时候，需要客户端自己讲图片纠正回来，倒着拍的时候亦然。

自定义相机功能就不多说了，网上有大量的优秀文章，[这里随便从网上找了一个](https://www.jianshu.com/p/3949ac312be7)，需要的可以参考下

## 基础知识

首先我们需要知道每一个`UIImage`对象，都有一个`imageOrientation`属性，里面保存着方向信息：

```
typedef NS_ENUM(NSInteger, UIImageOrientation) {
    UIImageOrientationUp,            // default orientation
    UIImageOrientationDown,          // 180 deg rotation
    UIImageOrientationLeft,          // 90 deg CCW
    UIImageOrientationRight,         // 90 deg CW
    UIImageOrientationUpMirrored,    // as above but image mirrored along other axis. horizontal flip
    UIImageOrientationDownMirrored,  // horizontal flip
    UIImageOrientationLeftMirrored,  // vertical flip
    UIImageOrientationRightMirrored, // vertical flip
};
```

根据这个属性信息，我们便可以对图像进行相应的旋转，将图片转到正确的方向，如何旋转？？有两种解决方案：

#### 第一种：给UIImage添加Category


```
- (UIImage *)fixOrientation {

    // No-op if the orientation is already correct
    if (self.imageOrientation == UIImageOrientationUp) return self;

    // We need to calculate the proper transformation to make the image upright.
    // We do it in 2 steps: Rotate if Left/Right/Down, and then flip if Mirrored.
    CGAffineTransform transform = CGAffineTransformIdentity;

    switch (self.imageOrientation) {
        case UIImageOrientationDown:
        case UIImageOrientationDownMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.width, self.size.height);
            transform = CGAffineTransformRotate(transform, M_PI);
            break;

        case UIImageOrientationLeft:
        case UIImageOrientationLeftMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.width, 0);
            transform = CGAffineTransformRotate(transform, M_PI_2);
            break;

        case UIImageOrientationRight:
        case UIImageOrientationRightMirrored:
            transform = CGAffineTransformTranslate(transform, 0, self.size.height);
            transform = CGAffineTransformRotate(transform, -M_PI_2);
            break;
        case UIImageOrientationUp:
        case UIImageOrientationUpMirrored:
            break;
    }

    switch (self.imageOrientation) {
        case UIImageOrientationUpMirrored:
        case UIImageOrientationDownMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.width, 0);
            transform = CGAffineTransformScale(transform, -1, 1);
            break;

        case UIImageOrientationLeftMirrored:
        case UIImageOrientationRightMirrored:
            transform = CGAffineTransformTranslate(transform, self.size.height, 0);
            transform = CGAffineTransformScale(transform, -1, 1);
            break;
        case UIImageOrientationUp:
        case UIImageOrientationDown:
        case UIImageOrientationLeft:
        case UIImageOrientationRight:
            break;
    }

    // Now we draw the underlying CGImage into a new context, applying the transform
    // calculated above.
    CGContextRef ctx = CGBitmapContextCreate(NULL, self.size.width, self.size.height,
                                             CGImageGetBitsPerComponent(self.CGImage), 0,
                                             CGImageGetColorSpace(self.CGImage),
                                             CGImageGetBitmapInfo(self.CGImage));
    CGContextConcatCTM(ctx, transform);
    switch (self.imageOrientation) {
        case UIImageOrientationLeft:
        case UIImageOrientationLeftMirrored:
        case UIImageOrientationRight:
        case UIImageOrientationRightMirrored:
            // Grr...
            CGContextDrawImage(ctx, CGRectMake(0,0,self.size.height,self.size.width), self.CGImage);
            break;

        default:
            CGContextDrawImage(ctx, CGRectMake(0,0,self.size.width,self.size.height), self.CGImage);
            break;
    }

    // And now we just create a new UIImage from the drawing context
    CGImageRef cgimg = CGBitmapContextCreateImage(ctx);
    UIImage *img = [UIImage imageWithCGImage:cgimg];
    CGContextRelease(ctx);
    CGImageRelease(cgimg);
    return img;
}
```

#### 第二种：利用`drawInRect`方法将图像画到画布上

```
- (UIImage *)normalizedImage {
    if (self.imageOrientation == UIImageOrientationUp) return self; 

    UIGraphicsBeginImageContextWithOptions(self.size, NO, self.scale);
    [self drawInRect:(CGRect){0, 0, self.size}];
    UIImage *normalizedImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return normalizedImage;
}
```

通过上面两种方式转换之后的`UIImage`对象，其`imageOrientation`属性，都会被修改成`UIImageOrientationUp`，这样将图片传到后台，或者导出相册的时候，就不会出现照片旋转90度的问题。

但是有时候我们希望图片该旋转的时候，按照我们的意愿旋转（比如横评拍摄的时候），竖直拍摄的时候，图像正常显示，这时候我们就不能直接用上面的方法来判断了。仔细观察系统相机的拍摄，我发现除了竖直拍摄以外，别的情况下拍摄，图片都会自动旋转，这个时候就需要我们利用iPhone手机自带的硬件传感器对方向进行判断，以达到我们想要的结果，这里主要用到加速仪

## 加速仪(类型：CMAcceleration)

加速仪可以检测三维空间中的加速度 ，坐标对应如下：

![o_coremotion1](http://otogtitz7.bkt.clouddn.com/2018-07-01-o_coremotion1.png)
例如：当垂直手持手机且顶部向上，Y坐标上回收到 -1G的加速度，（0，-1，0），当手机头部朝下，得到的各个坐标为：（0，1，0）

#### 主要代码如下：

```
- (void)startDeviceMotion{
    if (![self.motionManager isDeviceMotionAvailable]) {return;}

    [self.motionManager setDeviceMotionUpdateInterval:1.f];
    [self.motionManager startDeviceMotionUpdatesToQueue:[NSOperationQueue mainQueue] withHandler:^(CMDeviceMotion * _Nullable motion, NSError * _Nullable error) {
        
        double gravityX = motion.gravity.x;
        double gravityY = motion.gravity.y;
        
        if (fabs(gravityY)>=fabs(gravityX)) {
            
            if (gravityY >= 0) {
                
                // UIDeviceOrientationPortraitUpsideDown
                [self setDeviceDirection:SSDeviceDirectionDown];
                NSLog(@"头向下");
                
            } else {
                
                // UIDeviceOrientationPortrait
                [self setDeviceDirection:SSDeviceDirectionUp];
                NSLog(@"竖屏");
            }
            
        } else {
            
            if (gravityX >= 0) {
                // UIDeviceOrientationLandscapeRight
                [self setDeviceDirection:SSDeviceDirectionRight];
                NSLog(@"头向右");
            } else {
                
                // UIDeviceOrientationLandscapeLef
                [self setDeviceDirection:SSDeviceDirectionLeft];
                NSLog(@"头向左");
            }
        }
    }];
}
```

获取到方向信息，下面就可以对图片进行对应的处理了，主要用到了下面的这个方法：

```
- (instancetype)initWithCIImage:(CIImage *)ciImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation NS_AVAILABLE_IOS(6_0);
```

该方法的作用是：

```
Creates and returns an image object with the specified scale and orientation factors.
```

```
创建并返回具有指定比例和方向特征的image对象。
```

最后对拍摄的图片进行处理：


```
UIImage *transImage = [rsltImage fixOrientation];
        
        switch (self.deviceDirection) {
            case SSDeviceDirectionUp:
                transImage = [rsltImage fixOrientation];
                break;
            case SSDeviceDirectionLeft:
                transImage = [rsltImage fixImageByOrientation:UIImageOrientationLeft];
                break;
            case SSDeviceDirectionRight:
                transImage = [rsltImage fixImageByOrientation:UIImageOrientationRight];
                break;
            case SSDeviceDirectionDown:
                transImage = [rsltImage fixImageByOrientation:UIImageOrientationDown];
                break;
            default:
                break;
        }
```

## 最终效果图
![take_pic_ocr](http://otogtitz7.bkt.clouddn.com/2018-07-01-take_pic_ocr.gif)

## 总结

功能实现起来其实并不难，当时和同事纠结的地方在于，到底是采用支持横竖屏还是采用加速度传感器上面，最后经过分析系统相机，我还是采用了利用传感器做判断，期间也是查阅了很多的技术文章，无意中发现了一篇真心值得仔细阅读的[关于图片解压缩的文章](http://blog.leichunfeng.com/blog/2017/02/20/talking-about-the-decompression-of-the-image-in-ios/)。最后，再次对最近的松懈进行反思，继续撸起袖子，加油干！！！

## Refrence

https://www.cnblogs.com/sunyanyan/p/5213854.html

http://feihu.me/blog/2015/how-to-handle-image-orientation-on-iOS/

  









