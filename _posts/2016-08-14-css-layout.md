---
bg: "tools.jpg"
layout: post
title:  "HTML5-CSS布局"
crawlertitle: " HTML5-CSS布局"
summary: "HTML"
date:   2016-08-14 16:45:47 +0700
categories: blog
tags: ['2016-08']
author: 岑志军
---
一、默认情况下，所有的网页标签都在标准流布局中，从上到下，从左到右。
脱离标准流的方法有：

- float属性
- position属性和left、right、top、bottom属性

1、float属性的常用取值有：

- left：脱离标准流，浮动在父标签的最左边
- right：脱离标准流，浮动在父标签的最右边

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .one{
            background-color: red;
            width: 100px;

            float: right;
        }
        .two{
            background-color: yellow;
            width: 90px;
        }
    </style>
</head>
<body>
    <div class="one">11111111111</div>
    <div class="two">111111111111</div>
</body>
</html>
```

2、position属性取值


```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <style>
        .one{
            background-color: red;
            width: 500px;
            height: 400px;
            margin: 50px;
            position: relative;
        }
        .two{
            background-color: yellow;
            width: 100px;
            height: 90px;
            position: absolute;
            top: 20px;
            left: 20px;
        }
    </style>
</head>
<body>
    <div class="one">
        <div class="two">111111</div>
    </div>
</body>
</html>
```





