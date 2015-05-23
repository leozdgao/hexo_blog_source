title: "BOM中计算元素相关尺寸或偏移量的方式汇总"
date: 2015-05-14 17:49:17
tags: BOM
categories:
---

##  元素尺寸

获取元素的尺寸（Dimension），该尺寸为该元素border-box的大小
```
var rect = element.getBoundingClientRect();  // return ClientRect object
var height = rect.bottom - rect.top;
var width = rect.right - rect.left;
```


| 属性 | 说明 |
|------------|-----------------------------------------------------|
| clientHeight / clientWidth | 用户可见高度 / 宽度（元素的padding-box高度 / 宽度 - 滚动条宽度）|
| scrollHeight / scrollWidth | 元素的内容高度 / 宽度（包括元素的溢出部分）+padding |
| offsetHeight / offsetWidth | 元素的border-box高度 / 宽度（包括元素的padding+border+滚动条+正文高度 / 宽度）|

<!-- more -->

其中`scrollWidth`或者`scrollHeight`由于各浏览器滚动条实现的差异，可能会出现返回值不同的情况

Chrome及IE7下（不限于）

![0](http://7sbm5t.com1.z0.glb.clouddn.com/cssdimension0.png)


Firefox及IE8+（不限于）

![1](http://7sbm5t.com1.z0.glb.clouddn.com/cssdimension1.png)


**jQuery中获取元素尺寸的方法**

| 方法 | 说明 |
|------------|-----------------------------------------------------|
| innerWidth() / innerHeight() | 获取元素padding-box的尺寸 |
| width() / height() | 根据元素的box-sizing的设置返回相应box的尺寸 |
| outerWidth(includeMargin) / outerHeight(includeMargin) | includeMargin为true时则返回margin-box的尺寸，includeMargin为false则返回border-box的尺寸，includeMargin默认值为false|


**获取window的视口（viewport）大小**
```
// generic js
function getViewportDimension () {
  var quirksMode = document.compatMode == 'BackCompat';
  var rect = quirksMode ? document.body : document.documentElement; 
      
  return {
    width: rect.clientWidth,
    height: rect.clientHeight
  };
}

// in jQuery
$('body').innerWidth();
```

标准模式下： `document.documentElement`表示window视口大小，`document.body`获取的是body元素的大小
怪异模式下： `document.documentElement`获取的值为0，`document.body`获取的window视口大小

`document.documentElement.clientWidth`获取的值**不包括滚动条宽度**
`window.innerWidth`，`window.innerHeight`支持大多数主流浏览器以及IE9+，它们获取的值**包括了滚动条宽度**

## 滚动条相关

滚动条出现在元素的content-box外，包括在元素padding-box内，这意味着如果滚动条出现，元素的content-box被减小，padding-box大小不变

判断是否出现滚动条：
```
var hasVerticalScroll = element.scrollHeight > element.clientHeight;
var hasHorizontalScroll = element.scrollWidth > element.clientWidth;
```

判断滚动条宽度：（OSX下滚动条宽度为0）
```
<style>
.scrollbar-measure {
    width: 100px;
    height: 100px;
    overflow: scroll;
    position: absolute;
    top: -9999px;
}
</style>
<script type="text/javascript">
var temp = document.createElement('div'), length;
temp.className = 'scrollbar-measure';
document.body.appendChild(temp);
length = temp.offsetWidth - temp.clientWidth;
document.body.removeChild(temp);

console.log(length);
</script>
```


检查滚动条是否划到底部：

```
element.scrollHeight - element.scrollTop == element.clientHeight;
```

## 元素偏移量相关


| 属性 | 说明 |
|------------|-----------------------------------------------------|
| clientTop（get） | 元素的border-top-width |
| clientLeft（get） | 元素的border-left-width，如果text-direction为rtl，则还需要算上出现的垂直滚动条的宽度 |
| scrollTop（get / set） | 元素正文上方被卷去的（scrolled）高度 |
| scrollLeft（get / set） | 元素正文左边被卷去的（scrolled）高度 |
| offsetTop（get） | 元素border-box的上边缘到其父元素margin-box的上边缘的宽度 |
| offsetLeft（get） | 元素border-box的左边缘到其父元素margin-box的左边缘的宽度 |


`element.offsetParent`返回的是离element最近的containing element或者说是element的父元素，如果element或其中一个上级元素的样式为`display:none`，则该属性返回`null`

| 属性 | 说明 |
|-----------|------------------------------------|
| window.scrollX（pageXOffset ） | 整个文档被水平滚动了的宽度 |
| window.scrollY（pageYOffset） | 整个文档被垂直滚动了的宽度 |


为保证兼容性，应该尽可能使用`pageXOffset`或`pageYOffset`，不过在IE9以下版本并不支持该属性：
```
var supportPageOffset = window.pageXOffset !== undefined;
var quirksMode = document.compatMode == 'BackCompat';
var rect = quirksMode ? document.body : document.documentElement;

var scrolledWidth = supportPageOffset ? window.pageXOffset : rect.scrollLeft;
var scrolledHeight = supportPageOffset ? window.pageYOffset : rect.scrollTop;
```

