---
layout: post
title: "fastlane实现自动化打包"
date: 2018-04-26
tags: iOS
---

>正常产品开发完成之后，我们都需要给测试人员打包，又是测试包，又是生产包的，打一次包需要浪费十几分钟的时间，甚至有时候，你刚打完包，产品过来告诉你某个地方需要微调一下（麻蛋，这个时候是不是想弄死他），但是没办法，只好改完bug，继续打包，就这样可能一上午或者一下午就这样浪费了，所以有一个能够自动化打包的工具不仅能够为我们节省大量的时间，**还可以让我们能够装逼**。。。。。（这是重点）

其实自动化打包的工具有很多，比较流行的有`Jenkins`和`fastlane`，原来尝试过`Jenkins`,感觉这个工具比较麻烦，需要配置的东西非常多，还需要仓库地址等等很多信息，不像`fastlane`感觉是傻瓜式的，非常简单，目前[Github上已经超过两万star了](https://github.com/fastlane/fastlane)，而且团队人员众多，下面步入正题！！！

## 安装前的准备工作

1. 首先确认是否安装了ruby，终端查看下ruby版本

```
ruby -v
```

2. 确认是否安装了Xcode命令行工具

```
xcode-select  --install
```

如果出现
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247254544910.jpg)
表示已经安装成功

如果出现：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247255157977.jpg)

就点击安装就行了。。。

下面就正式开始安装fastlane了

## 安装步骤

1. 安装fastlane

```
sudo gem install fastlane
```

2. 切换到工程目录初始化

```
fastlane init
```

初始化的过程中会出现下面的选项：
![0280EF1C30306802E173DFDFD724032A](http://otogtitz7.bkt.clouddn.com/2018-04-26-0280EF1C30306802E173DFDFD724032A.jpg)

第一个选项的意思是：自动截屏。这个功能能帮我们自动截取APP中的截图，并添加手机边框（如果需要的话）
第二个选项的意思是：自动发布beta版本用于TestFlight
第二个选项的意思是：自动发布到AppStore
第二个选项的意思是：手动设置

我在这里选的是第四个（大家可根据自己需要选择），截图如下：

![ACA6B6119DF13E2F16A1A6512563A32F](http://otogtitz7.bkt.clouddn.com/2018-04-26-ACA6B6119DF13E2F16A1A6512563A32F.jpg)

紧接着一直点击`enter`键，知道安装成功会出现如下截图

![D51E2EB4B604605AE7C5B86951DCE6E8](http://otogtitz7.bkt.clouddn.com/2018-04-26-D51E2EB4B604605AE7C5B86951DCE6E8.jpg)

安装成功之后，会在我们的工程目录生成一个`fastlane`文件夹：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247263009632.jpg)

然后此时，我们需要自己编辑`Appfile`和`Fastfile`两个文件:

首先看`Appfile`文件，我的如下：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247264551130.jpg)

然后是`Fastfile`文件：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247266906282.jpg)

其中的`firim`是指定到上传到`fir`的，如果只是单纯的想把包打出来可以不写哪一行。

这样的话就可以顺利打包了。。。
执行打包命令：

```
fastlane betaDebug
```

打包成功截图如下：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247289369146.jpg)


## 自动上传至`fir`或者蒲公英

但是，如果想将自己打好的包直接上传到`fir`或者蒲公英等平台，请看下面的步骤：


执行如下命令安装`fir`插件：

```
fastlane add_plugin fir
```

自动上传到`fir`还需执行如下命令：

```
gem install fir-cli
```

如果是蒲公英平台，安装如下插件：

```
fastlane add_plugin pgyer
```

此时`fastlane`文件夹会变成如下结构：
注意：`package`文件夹是在第一次打包的时候生成的
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247270913064.jpg)

此时执行打包命令,就可以自动打包，并上传至`fir`了。

安装完插件之后`Pluginfile`文件内容如下：（**注意**：你安装了什么插件，就会在该文件中显示）
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247278181731.jpg)


上传`fir`成功截图如下：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247271925792.jpg)

生成的`ipa`包和`dysm`文件如下：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247276440903.jpg)

至此，自动化打包安装过程结束，下面记录下我遇到的坑。

## 遇到的坑

* **错误一**

![908AA24D21E5040D79E27747E6D621AD](http://otogtitz7.bkt.clouddn.com/2018-04-26-908AA24D21E5040D79E27747E6D621AD.jpg)

我遇到这个问题的原因是，证书没有匹配对，修改`Fastfile`文件，仔细查看下`export_method`参数是否配对就行了。。。

* **错误二**

![D7ADA2F6C1CCA094D0E07D81EF414DE3](http://otogtitz7.bkt.clouddn.com/2018-04-26-D7ADA2F6C1CCA094D0E07D81EF414DE3.jpg)

错误指出的很明显，请一定要记住`:`后面一定要紧跟自己写的名称

* **错误三**

打包成功了，但是上传至`fir`一直失败
忘记截图了，大概报错说明如下：

```
Could not find action, lane or variable 'firim'
```

我原先看文档，看到有人将`Gemfile`和`Gemfile.lock`文件拖到`fastlane`文件夹里面了，但是自动生成的话是在这个文件夹外面的，但是我想着放到一个文件夹里面方便管理，就这样报错了，所以记住，它生成在哪你就放在哪就行。

报错原因是，没有找到`firim`这个action，可以在终端下面查看是否安装了这个`action`

```
fastlane actions [firim]
```

如果安装了，会显示如下：
![](http://otogtitz7.bkt.clouddn.com/2018-04-26-15247287834633.jpg)

如果没有安装，会提示没找到，这个时候重新安装下插件就好了。

## shell脚本打包

除了借助一些开源框架外，我原来也用过`shell`脚本打包，无非是自己写一个脚本，里面包含很多的打包命令，但是还是感觉没有`fastlane`简单方便，有兴趣的可以参考[GitHub上的这个](https://github.com/stackhou/AutoPacking-iOS)，写的比较详细

## 总结

至此，利用`fastlane`自动化打包就算告一段落了，但是[fastlane官网](https://docs.fastlane.tools/)还提供了很多的语法说明，感兴趣的可以参考下，另外说明下，由于我是最近才开始用，所以一般给测试人员打包的时候我都是用`fastlane`，真正要上线提交AppStore的时候，我还是用Xcode，毕竟放心。提交到App Store还没用过，有什么坑我也不知道，如果有人实践过，欢迎评论区互相交流（另外，觉得写得不错的，请点赞❤️❤️❤️！！！哈哈）

***

我的博客即将搬运同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=1denjdl5jafn










