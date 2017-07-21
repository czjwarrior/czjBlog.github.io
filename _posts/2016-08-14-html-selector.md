---
bg: "tools.jpg"
layout: post
title:  "HTML5选择器"
crawlertitle: "HTML5选择器"
summary: "HTML"
date:   2016-08-14 12:22:47 +0700
categories: blog
tags: ['2016-08']
author: 岑志军
---
最近刚开始学习HTML5，记录一下自己学习的笔记，方便以后查阅： 
首先选择器的常用分类：
 
- 标签选择器

```
div{
       color: red;
   }
```
- 类选择器

```
.one{
       color: yellow;
    }
```
- id选择器，注意id是唯一的标示，不能用于其他标签



```
#main{
       font-size: 40px;
     }
```
- 后代选择器

```
#test1 div{
            color: black;
            font-size: 50px;
        }
```

- 属性选择器

```
div[name]{
            color: blue;
        }
```

Demo练习：

```
<body>
    <div id="main">11111111111111</div>
    <div>11111111111111</div>
    <div>11111111111111</div>

    <p>2222222222222</p>

    <div class="one">2222222222</div>
    <p class="one">4444444444444</p>

    <div id="test1">
        <p>
            <div>3434343424234</div>
        </p>
    </div>

    <div name="cen">98989898</div>
    <div>77777777777</div>
</body>
```





