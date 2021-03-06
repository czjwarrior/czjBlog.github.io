---
layout: post
title: "iOS逆向-ipa包重签名及非越狱手机安装多个微信"
date: 2018-05-29
tags: iOS
---

> 前一段时间学了点儿逆向相关的一些东西，但是都是基于越狱手机上的操作，给视频类应用去广告之类的。随着苹果生态圈的逐渐完善、及苹果对自身系统的保护越来越严格，导致现在的iPhone手机并不像以前那样存在大量的越狱用户。


前段时间我自己申请了个微信小号，申请小号的目的就是原来微信号好友中乱七八糟的人实在太多，感觉自己的朋友圈都是一些无关紧要的垃圾信息，曾经关闭了一段时间的朋友圈，但是最近遇到了好多技术上很强的同行，还想了解大佬们的动态。于是我就想着申请了个小号，但是麻烦来了，iPhone手机并不像安卓手机那样存在着微信多开之类的应用，将自己手机越狱吧成本太高，于是就想着通过技术手段安装多个微信，下面步入正题：

## 为什么要重签名

其实我们平时开发的App，程序运行主要就是加载一个`Mach-o`可执行文件。当我们将程序打包成`ipa`文件，上传到App Store的时候，期间就是进行了一些加壳操作，比如：数字证书签名等。重签名的目的就是将别人的程序重新签上我们的证书信息。也可以简单理解为将别人的加密文件解密，加上我们自己的加密算法。

#### 逆向当中的一些专业术语

* 加壳：利用特殊算法（iOS中数字证书），对可执行文件的编码进行改变，以达到保护程序代码的目的
* 脱壳：摘掉壳程序，将未加密的可执行文件`Mach-o`还原出来

#### 查看应用是否加壳

将下载好的`ipa`包解压缩之后，拿到里面的`Mach-o`文件，`cd`到所在目录，执行如下命令：

```
otool -l 可执行文件路径 | grep crypt
```
![](http://otogtitz7.bkt.clouddn.com/2018-05-29-15276021774171.jpg)

其中`cryptid`代码是否加壳，`1`代表加壳，`0`代表已脱壳。我们发现打印了两遍，其实代表着该可执行文件支持两种架构`armv7`和`arm64`.

#### 查看应用支持哪种架构

终端下执行如下命令查看架构信息

```
lipo -info 文件路径
```

![](http://otogtitz7.bkt.clouddn.com/2018-05-29-15276023773959.jpg)

除了查看架构信息，还可以利用该指令导出某种特定架构、合并多种架构：

* 导出特定架构


```
lipo 文件路径 -thin 架构类型 -output 输出文件路径
```

* 合并多种架构


```
lipo 文件路径1 文件路径2 -output 输出文件路径
```

#### 怎么给应用脱壳

给应用脱壳有两种途径：

* 一、直接从一些第三方应用商店里面下载你想脱壳的应用，例如：`PP助手`、`iTools`等
* 二、自己脱壳，利用`GitHub`上开源的一些工具，常用的有[Clutch](https://github.com/KJCracks/Clutch/releases)、[dumdecrypted](https://github.com/stefanesser/dumpdecrypted/)。具体如何使用，请自行Google
* 

**前期准备工作：**

* 一台iPhone，越不越狱都行
* 开发者证书或者企业证书（个人账号也行，但是应用安装上之后，有效期只有7天）
* 电脑安装 [iOS App Signer](https://dantheman827.github.io/ios-app-signer/)

其实重签名的方式有很多，比如：可以利用`sigh resign`命令，在终端下操作，还可以借助一些逆向相关的重签名工具，本文采用`iOS App Signer`

了解以上基本概念之后，下面正式开始史上最详细的重签名过程，以微信为例：

## 第一步：准备好脱壳后的微信App

我是直接从`PP助手`上下载的，感兴趣的可以自己手动脱壳

![CC5D0564776F714701F45AE5846D3464](http://otogtitz7.bkt.clouddn.com/2018-05-29-CC5D0564776F714701F45AE5846D3464.jpg)

## 第二步：将对用的`ipa`文件解压，修改一些东西

注意：个人证书不能重签`Extension`文件，所以要删除`ipa`包中包含的相应文件，包括`Watch`里面的`Extension`，为了方便一般直接将`Watch`文件删除：

![C00C57F8EB9509B60FDCD8006DF3E719](http://otogtitz7.bkt.clouddn.com/2018-05-29-C00C57F8EB9509B60FDCD8006DF3E719.jpg)

![34293DFAD48B9B855A066721DC0D358B](http://otogtitz7.bkt.clouddn.com/2018-05-29-34293DFAD48B9B855A066721DC0D358B.jpg)

## 第三步：利用`iOS App Signer`给微信重签名

![D063FFD5907B3090DE0494AF05978BDB](http://otogtitz7.bkt.clouddn.com/2018-05-29-D063FFD5907B3090DE0494AF05978BDB.jpg)

* 第一项：对应的`.ipa`或者`.app`路径
* 第二项：我们自己的签名证书
* 第三项：证书对应的`Profile`文件，默认项`Re-Sign Only`是无效的，选择证书下存在的`Profile`文件）
* 第四项：重签名之后的`Bundle identifier`（选择了`Profile`文件，一般会自动填写）
* 下面几项可以随便写

![25441B7BB39F6BADE92ECA6187C0F6AA](http://otogtitz7.bkt.clouddn.com/2018-05-29-25441B7BB39F6BADE92ECA6187C0F6AA.jpg)

签名完毕之后对应的文件夹下会生成重签名之后的`ipa`包

![F2804E382DD16002B001BAA1F927AA3](http://otogtitz7.bkt.clouddn.com/2018-05-29-F2804E382DD16002B001BAA1F927AA3C.jpg)

**注意：**利用`iOS App Signer`重签名，在删除掉相应的`Extension`，选择路径的时候，一定要选择`Payload`文件夹下对应的`.app`文件，否则会报找不到`Payload`文件夹的错误：

![4C2E14FA5CA3D81E4BEC2F44D9D93647](http://otogtitz7.bkt.clouddn.com/2018-05-29-4C2E14FA5CA3D81E4BEC2F44D9D93647.jpg)

## 安装重签名之后的微信

可以用`PP助手安装`，也可以用`Xcode`安装，我采用`Xocde`安装：

![install-wechat](http://upload-images.jianshu.io/upload_images/423503-33808e5a3de12398.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


不出意外地话，第二个微信就成功的安装到了你的手机上。如果装不上的话，基本上大部分原因就是证书不对。。。

## 最终效果

多个证书可以多次重新签名，安装多个相同的应用

![wechat_hook_01](http://otogtitz7.bkt.clouddn.com/2018-05-29-wechat_hook_01.jpg)


![wechat_hook_02](http://otogtitz7.bkt.clouddn.com/2018-05-29-wechat_hook_02.jpg)




这篇文章图有点儿多。。。。。

**注意：重签名方式安装的微信，是对微信APP的一种破解，会被官方认定为非安全软件，有被封号的危险**。但是这种方式对破解各种其他软件都是有用的，利用逆向相关的知识，我们可以利用这种知识做很多我们想做的事儿！！！（不要做非法的事情哈！）

## 遇到的坑

* 错误一：
![988D211E9B6C6BEF7659DAA8854AAFBD](http://otogtitz7.bkt.clouddn.com/2018-06-04-988D211E9B6C6BEF7659DAA8854AAFBD.jpg)

**解决办法：**证书不对，仔细检查下证书

* 错误二
![BD2A4E9CB954AB66E6598ED5CCA85F8B](http://otogtitz7.bkt.clouddn.com/2018-06-04-BD2A4E9CB954AB66E6598ED5CCA85F8B.jpg)

**解决办法：**删除`ipa`包里面的`watch`相关的文件



## 总结

过程其实很简单，我始终认为借助一些工具能完成的东西，都是很简单的，因为不需要敲代码。总算给自己iPhone装上了多个微信，原来还打算买个安卓手机呢，哈哈，给自己省了一大笔钱。。。




