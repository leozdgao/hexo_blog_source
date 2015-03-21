title: "CSS Reset及相关思考"
date: 2015-03-17 20:55:13
categories: HTML/CSS
tags: css
---

有一个专门讲CSS Reset的站点。 [site](http://www.cssreset.com/)

由于各大浏览器的user agent样式（即默认样式）不同，为了尽可能避免浏览器之间样式的差异，重置了css样式，让浏览器之间可以由一个基本相同的样式基准，并在该基准的基础上进行开发。

## 引入CSS Reset样式

引入CSS Reset，意味着告诉浏览器将所有元素应用上reset的样式，再使用针对页面或业务的特定样式。对于页面加载而言，这不算一个最优的方式，但对于组织css代码而言是一个好方法，因为页面针对不同浏览器都在一个统一的基准上开发。

而通常来说CSS Reset网上有许多现成的（可以去上面的链接里找），但直接引入或者直接复制黏贴不是推荐的做法，这里我直接引用：

> The reset styles given here are intentionally very generic. I don’t particularly recommend that you just use this in its unaltered state in your own projects. It should     be tweaked, edited, extended, and otherwise tuned to match your specific reset baseline. Fill in your preferred colors for the page, links, and so on. In other words, this is a starting point, not a self-contained black box of no-touchiness.

简单地说，引入bootstrap后，即使html中没有定义任何class或者inline style，依然可以呈现出优雅的页面，即推荐大家根据自己的需求定制自己的CSS Reset，而不是直接拷贝（虽然确实省事）。

<!-- more -->

## CSS设计

在为一个站点写css样式时，我认为可以分为三层：全局层，模块层，页面逻辑层。

全局层代表的是应用在所有页面上样式，模块层值的是应用在各个具有相同样式规则的元素上的，页面逻辑层是每个页面都不共享的一些特殊样式。而CSS Reset则可以作为全局层的一部分让所有页面具有相同的样式基准，对于模块层和页面逻辑层，举个例子：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
    /*全局层*/
    * { margin: 0px; padding: 0px; }

    /*模块层*/
    .list {
        margin-left: -10px;
        margin-right: -10px;
        font-size: 16px;
        width: 100%;
    }
    .list .list-item {
        padding-left: 10px;
        padding-right: 10px;
        float: left;
        background: #ccc;
    }
    .list:after, .clearfix:after {
        content: "";
        display: table;
        clear: both;
    }

    /*页面逻辑层*/
    .testlist {
        color: #3c3c3c;
    }
    </style>
</head>
<body>
    <div class="list">
        <div class="list-item testlist">a</div>
        <div class="list-item testlist">a</div>
        <div class="list-item testlist">a</div>
        <div class="list-item testlist">a</div>
        <div class="list-item testlist">a</div>
    </div>
</body>
</html>
```

list模块包括了样式`.list`、`.list-item`，可应用于一些需要呈现列表的情景，而`testlist`样式仅包含应用于特定场景的特殊样式。不难发现，CSS中的注释对代码维护有着重大意义。

## 关于使用Universal Selector(*)做CSS Reset

使用全局选择器或者叫通配选择器(*)，是一种最简便的CSS Reset的方式，可能会是这样子的：

```
* {
    margin: 0px;
    padding: 0px;
}
```

或许会在这个基础上再加上其他style的重设，比如`border: 0;`或者是`outline:0;`这样子。但这样子维护CSS Reset是有许多弊端的：

- 没有明确的指出哪个元素是被reset的了，这个算是语义上的问题
- 样式继承将无法作用于任何元素，因为全局选择器会匹配所用的元素，其样式权值为`0,0,0,1`，而继承样式的权值为`<0,0,0,1`，这也导致了子元素的样式需要重复的逐个定义（个人认为是主要隐患）
- 由于该选择器将匹配所有元素，可能会增加页面的加载时间（这个是有争议的观点）
- IE6、IE7不支持该选择器

由于使用这种reset的方式虽然简便，但可能会带来一些隐患，使用它时需要清楚的知道里面的样式规则是否会带来丢失继承的隐患，其实`Tripoli CSS Reset`中也是有部分样式是使用这个选择器的，大家根据需求自己权衡。

## 小提示

在写CSS Reset的时候应该要注意，比如我设置：

```
a {
    color: #333;
}
```

那么导致的问题是需要每次手动设置a标签的`:focus`、`:hover`等样式，因为对a标签应用该样式会覆盖所有a标签的默认样式，包括伪类（伪类也被认为是应用于a标签的样式，伪类样式权值为`0,0,1,0`）。同样的问题也会出现在像`outline`这样的样式上。
