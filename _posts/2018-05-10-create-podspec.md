---
layout: post
title: "使用Cocoapods创建私有库"
date: 2018-05-10
tags: iOS
---

> 五一之后，公司要求对代码进行整理，同时进行代码管理、自动化打包等标准化流程，这些东西一直是我想搞的，这次有了公司的支持，操作起来也更顺利了，代码管理、自动化打包会找时间写一篇博客，这次主要记录利用Cocoapods将多个项目中共用的代码抽离出私有库，方便其他项目的引用，也算是组件化的第一步吧。抽离出私有库的时候，参考了很多的博客，遇到了很多的问题，主要参考了[这篇博客](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)。

## 为什么要进行代码抽离

很多公司不止有一个产品，当项目达到两个及以上的时候，就需要考虑代码的共用（理想情况下）。由于我们公司特殊情况（懒），原来做项目的时候没有考虑这么多，由于公司新项目与原来的项目有大量功能相似，当时我们就采用了创建新分支的形式创建了新项目，导致代码耦合性非常强，平常开发中遇到很多问题，比如：

* 有些代码可能这个项目需要，那个项目不需要
* 分支切换太过频繁
* 创建了大量分支
* 在这个分支上开发的时候，测试需要另一个项目的包，需要来回切换
* 。。。。等等

先看下我们原来的代码结构，确实感觉很头疼：
![](http://otogtitz7.bkt.clouddn.com/2018-05-10-15259585288914.jpg)

综上所述，代码抽离迫在眉睫。。。。

## 1、创建私有`Spec Repo`

`Spec Repo`其实类似一个容器，里面装着所有的公开的Pods,当使用`Cocoapods`后，他就会被`clone`到本地的`~/.cocoapods/repos`目录下：
![](http://otogtitz7.bkt.clouddn.com/2018-05-10-15259592117791.jpg)

因此我们也需要创建一个私有的`Spec Repo`，因为是公司项目，所以我们搞一个私有库，这次是我单独的练习，`GitHub`上创建私有库是收费的，所以这次我采用了免费的`Git`服务，我用的是[Coding](https://coding.net/)，首先需要在`coding`上创建一个自己的`git`仓库，创建完成之后，在终端下执行如下命令

```
pod repo add ZJTestSpecs https://coding.net/u/cenzhijun/p/ZJTestSpecs/git
```

成功的话就会在`~/.cocoapods/repos`目录下看到`ZJTestSpecs`文件夹了，第一步完成，这一步通常只需要执行一次

## 2、创建`Pod`项目的文件

首先`cd`到你想创建项目的文件夹执行如下操作

**记住一定要创建一个单独的名字，否则以后`pod search <私有库>`会找到`Github`上跟你重名的项目**

```
pod lib create ZJPodPrivateTest
```

之后会出现下列问题：

![DA3D7B14F4D628E154A175EFFDE87B27](http://otogtitz7.bkt.clouddn.com/2018-05-10-DA3D7B14F4D628E154A175EFFDE87B27.jpg)

接下来就是在你的`ZJPodPrivateTest`文件夹下添加自己的内容，将自己的模块部分放在`ZJPodPrivateTest/Classes`下，然后`cd`到`Example`文件夹下执行`pod update`命令，之后打开项目，就能在`Development Pods/ZJPodPrivateTest`文件夹下看到自己添加的组件了，之后需要将项目推送到远端仓库，同样需要先自己去`git`服务商哪里创建一个私有仓库，然后`cd`到`ZJPodPrivateTest`目录，执行如下操作：

```
git add -A
git commit -a -m "init library"
git remote add origin https://git.coding.net/cenzhijun/ZJPodPrivateTest.git

git push origin master
```

这个时候执行`push`操作会报如下错误：

![C3672E624EF076875C02B836EFE87ED7](http://otogtitz7.bkt.clouddn.com/2018-05-10-C3672E624EF076875C02B836EFE87ED7.jpg)

提示你需要先`pull`下代码，这一步不能直接`pull`,需要执行如下命令：

```
git pull origin master --allow-unrelated-histories
```

![D6043786E0DA1A559CA1DA5F27EB3F90](http://otogtitz7.bkt.clouddn.com/2018-05-10-D6043786E0DA1A559CA1DA5F27EB3F90.jpg)

有可能会出现冲突，解决冲突之后`push`代码：

```
git push origin master 
```

因为`podspec`文件获取版本控制的项目需要`tag`号，所以还要搭上一个`tag`

```
git tag -m "first release" 0.1.0
git push --tags     #推送tag到远端仓库
```

做完这些之后开始编辑`podspec`文件，填上对应的信息。

编辑完之后，执行如下命令，验证是否有效，不能有`error`或者`warning`：

```
pod lib lint
```

当看到

![6CECC0B42712110AA9F952FFE1C2E179](http://otogtitz7.bkt.clouddn.com/2018-05-10-6CECC0B42712110AA9F952FFE1C2E179.jpg)

就说明验证通过

## 3、本地测试podspec文件

自己可以创建一个新项目，在`Podfile`中指定自己编辑好的`podspec`文件，如下：（两种方式填写一种就行）

```
pod 'ZJPodPrivateTest', :path => '~/Desktop/ZJPodPrivateTest'      # 指定路径
# pod 'PodTestLibrary', :podspec => '~/Desktop/ZJPodPrivateTest/ZJPodPrivateTest.podspec'  # 指定podspec文件

```

然后执行`pod install`命令安装，然后打开项目发现库文件已经被加载到`Pods`子项目中了，不过没有在`Pods`目录下，而是在`Development Pods/ZJPodPrivateTest`目录下，因为是本地测试项目，没有吧`podspec`文件添加到`Spec Repo`中的缘故
![EEBF626209D091109DC43B9298F96E26](http://otogtitz7.bkt.clouddn.com/2018-05-10-EEBF626209D091109DC43B9298F96E26.jpg)

<br>
![5B479CE603BFA168CD47132A8E8E5781](http://otogtitz7.bkt.clouddn.com/2018-05-10-5B479CE603BFA168CD47132A8E8E5781.jpg)

确认无误后，就可以提交`podspec`到`Spec Repo`中了

## 4、提交podspec

提交很简单，只需要一个命令：

```
pod repo push ZJTestSpecs ZJPodPrivateTest.podspec  #前面是本地Repo名字 后面是podspec名字
```

![481992FDCEF1AB9F310B905AF4D4A](http://otogtitz7.bkt.clouddn.com/2018-05-10-481992FDCEF1AB9F310B905AF4D4AC92.jpg)


没有错误之后，就可以在`~/.cocoapods/repos/ZJTestSpecs`目录下看到自己的私有库了，同时我们远程的`Spec Repo`也有一次提交，已经被自动`push`上去了

可以用`pod search ZJPodPrivateTest`查看自己的库了

![714E14C7AA77A9CDECE5C5334B28F057](http://otogtitz7.bkt.clouddn.com/2018-05-10-714E14C7AA77A9CDECE5C5334B28F057.jpg)

***一定要记住自己的创建的私有库一定不要跟Github上的第三方库重名，否则会搜不到，**我博客里面有的是`ZJPodPrivateTest`有的是`ZJPodTest`，就是因为`ZJPodTest`跟`Github`上的一个第三方库重名了，才会又重新建了`ZJPodPrivateTest`，按照我博客操作的时候`ZJPodPrivateTest`和`ZJPodTest`可以认为是同一个仓库，有的截图了，有的忘了，有不明白的，可以问我！！！

至此，自己的私有库就算制作好了

## 5、使用制作好的Pod

在`Podfile`文件中，内容如下：

```
source 'https://github.com/CocoaPods/Specs.git'  # 官方库
source 'https://git.coding.net/cenzhijun/ZJTestSpecs.git'   # 私有库

platform :ios, '8.0'

target 'TargetName' do
pod 'AFNetworking', '~> 3.0'    
pod 'ZJPodPrivateTest', '0.1.0'   #自己的私有库
end

```

至此就算大功告成了

## 6、更新维护podspec

[这里写得很详细，参考这个吧](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)

## 总结

创建私有库的时候，尽管我参考的这篇文章已经写得十分详细，但是还是有一个过时的操作，很导致操作错误，尤其是在本地仓库`push`到远程仓库那里出现问题，同时创建私有库不能和`GitHub`上存在的第三方库重名也是我摸索很久发现的，希望看到这篇文章的同学能够少走弯路❤️❤️❤️❤️










