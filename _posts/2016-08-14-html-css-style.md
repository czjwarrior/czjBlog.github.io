---
layout: post
title: "HTML5-css样式分类"
date: 2016-08-14
tag: iOS
---
- 行内样式

```
<body>
<!--行内样式-->
    <div style="font-size: 30px; color: red; background-color: blue;">121221323231221</div>
    <div style="font-size: 30px; color: red; background-color: blue;">121221323231221</div>
    <div style="font-size: 30px; color: red; background-color: blue;">121221323231221</div>
    <div style="font-size: 30px; color: red; background-color: blue;">121221323231221</div>
    <div style="font-size: 30px; color: red; background-color: blue;">121221323231221</div>
    <div style="font-size: 30px; color: red; background-color: blue;">121221323231221</div>

    <p>r43fdjvbdflvnadkjvadjvndfajkvndavnavnadfkvnadjk</p>
</body>
```
显然这种样式是非常麻烦的，代码非常冗余，真正开发中基本不用。

- 页内样式

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <!-- 单个文件当中使用-->
    <title></title>
    <style>
        div{
            color: green;
            font-size: 40px;
            background-color: red;
        }
    </style>
</head>
<body>
    <div>111111111111111111</div>
    <div>111111111111111111</div>
    <div>3333333333333333</div>
    <div>111111111111111111</div>
    <div>555555555555555</div>

    <p>222222222222222222</p>
</body>
</html>
```
- 外部样式。(*这种样式在开发中比较常见)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title></title>
    <link rel="stylesheet" href="css/index.css">
</head>
<body>
    <div>3322222223232332323</div>
    <p>2222222222222222222222</p>
</body>
</html>
```
另外需要单独建立一个css文件，可供整个工程使用

```
div{
    color: brown;
    font-size: 50px;
}

p{
    background-color: yellow;
    color: green;
    font-size: 39px;
}
```



