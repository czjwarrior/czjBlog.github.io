---
layout: post
title: "CocoaPods升级"
date: 2016-12-13
tag: iOS
---

前提是你以前已经安装过CocoaPods 
1、查看当前pod版本

```
pod --version
```

2、命令行安装

```
// 先更新gem
sudo gem update --system    // 需要漫长的等待
```

3、依次执行命令

```
brew install ruby
```

```
gem sources --remove https://rubygems.org/
```

```
gem sources -a https://ruby.taobao.org/
```

```
gem sources -l
```

```
*** CURRENT SOURCES ***

https://ruby.taobao.org/
```

```
sudo gem install cocoapods      // 安装cocoapods
```

```
pod setup
```

##大功告成！！！




