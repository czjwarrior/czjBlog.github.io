---
layout: post
title: "iOS客户端图片处理组件技术方案"
date: 2021-09-21
tags: iOS
---

## 背景

原有的CDN功能支持用法比较复杂，尤其多个功能组合（例如：Resize+Corner）时，会造成代码逻辑比较复杂，各个业务无法复用，相同逻辑有大量重复代码。业务方在使用API是对传入的ptsize大小不知所措，不同机型对应的ptsize大小固定，不能很好地展示图片内容。

## 技术分析

在收集完业务方的各个痛点之后，我主要对以下几点进行优化：
1、图片处理统一放到组件内部，对服务端处理过的图片Url进行还原
2、组件内部对象初始化懒加载，保证不同Url只初始化一次
3、简化api接口，采用链式调用，不同action进行明显区分，保证新加的action易扩展
4、业务方只需要关心UI设计稿上的ptsize，框架内部按照设备scale进行统一处理
5、对于不同设备状态，尤其是低端机型采用升降级策略，图片展示优先

## 技术设计

图片处理组件只对外暴露这一个接口：

```
/**
 *  创建一个PFLImageMaker，内部采用懒加载
 *  @param block 添加想处理的action
 *  @return 返回处理好的图片Url
 */
- (NSString *)imageMaker:(void (^)(PFLImageMaker *maker))block;
```


得到服务到返回图片Url之后，需要对图片进行一系列处理，大概流程图如下：
![](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16310177136715.jpg)

为了方便业务接入，直接为NSString添加一个扩展类，通过接口可以直接调用，方法内部懒加载创建一个PFLImageMaker对象。目前图片处理组件提供`Resize`、`Corner`、`Crop`、`Circle`四种Action，基本可以满足伴鱼内部各业务线的相关需求，后期正在考虑加入`毛玻璃、格式转换`等功能，同时在各个Action内部，进行升降级管理，监控设备性能（电量、内存、网络情况），在设备性能不好的情况下，主动降低图片质量，保证用户体验。

## 技术实现

![](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16306525363315.jpg)


**NSString+ImageMaker：**主要用于提供接口给各个业务方，内部进行图片Url预处理，保证进入图片处理的图片Url是原图的Url。同时进行Url校验，对于不合法的Url，在开发阶段实时反馈给对应的开发人员，保证线上无异常。懒加载创建PFLImageMaker，保证相同的图片Url只有一个ImageMaker对象，防止重复创建，占用内存。

**PFLImageMaker：**主要是图片处理各个Action类统一初始化的地方，保证收口在同一个地方，跟扩展类解耦，方便以后随时加入别的Action。

**PFLImageMakerDeviceManager：**该类主要是提供升降级的管理类，处理一些升降级的逻辑。

**PFLResizeImageMaker：**图片缩放的Action

**PFLCornersImageMaker：**圆角矩形的Action

**PFLCropImageMaker：**自定义裁剪的Action

**PFLCircleImageMaker：**内切圆的Action

## 升降级策略
在组件内部引入升降级策略，对升降级策略添加了云控开关进行控制，一旦线上出现问题，也可进行直接关闭，不影响线上运行。

* 监听网络情况，如果网络情况不好，进行降级
* 监听内存使用情况，如果内存使用比较紧张则进行降级
* 监听设备电量，如果电量不足则进行降级

## 相关用法：

#### Resize(图片缩放)

```
maker.resize.w(168).h(125).mode(PFLImageResizeFill).rstUrl();
```

#### Corner(圆角矩形)

```
maker.corners.radius(12).rstUrl();
```

#### Crop(自定义裁剪)

```
maker.crop.x(100).y(90).rstUrl();
```

#### Circle(内切圆)

```
maker.circle.radii(20).rstUrl();
```

## 优化成果

绘本首页2000张瀑布流图片：

#### 模拟用户真是使用场景

优化前 | 2x设备 | 3x设备 | 收益
--------- | ------------- | --------- | -----
723M | 156M | 282M | 60%+

优化前：723M
![-w416](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16300571536786.jpg)


优化后
3x   优化后 282M
![-w406](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16300565695828.jpg)


2x  156M
![-w389](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16300585041285.jpg)

#### 模拟用户手机性能不好场景（完全降级）

优化前 | 2x设备 | 3x设备 | 收益
--------- | ------------- | --------- | -----
723M | 53M | 154M | 70%+


3x 优化后:     154M
![-w393](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16303910281066.jpg)


2x 优化后：   53M
![-w391](https://cenzhijun.oss-cn-beijing.aliyuncs.com/2021/09/27/16303915768647.jpg)


## 总结

图片处理是伴鱼各个业务线都需要接触和处理的重要业务，目前图片处理组件已上线一个月左右，线上运行正常，在伴鱼绘本主端各个业务线引入，同时伴鱼国际化团队的同事也有在用。满足业务开发方便的同时，也解决了业务开发中的痛点。

## 作者介绍

* 岑志军 伴鱼 iOS 工程师，负责伴鱼绘本客户端研发，图片性能优化等工作