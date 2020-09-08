# 2019.4.18 CSSOM API

> 全称 CSS 对象模型，涉及两部分内容，第一部分和操作样式表相关，第二部分和元素尺寸相关

[TOC]

HTML 与 CSS，一个负责表示语义，一个负责表现，DOM 与 CSSOM 也是一样，一个负责语义，一个负责表现，例如 width、height 显示相关的显示

css 样式表的模型，也就是 cssom 本体

```
<style title="Hello">
a {
  color:red;
}
</style>
<link rel="stylesheet" title="x" href="data:text/css,p%7Bcolor:blue%7D">

```

可以以 dom 节点的方式去去获取这个节点的内容，属性，这次讲的`cssom`所以我们使用 cssom Api 来操作生成的样式表

## 获取文档中所有的样式表

```
document.styleSheets
```

![](https://raw.githubusercontent.com/HongHongMing/Images/master/15555551652821.jpg)

表示所有文档中所有的样式表，这是一个只读的列表，可以通过通过方括号访问与 item 的方法，length 属性代表样式表数量

里面获取的样式表只能通过 style 标签与 link 标签才创建

### 修改样式表的内容

```
document.styleSheets[0].insertRule("p { color:pink; }", 0)
document.styleSheets[0].removeRule(0)
```

### 获取样式表中的特定规则（Rule）

```
document.styleSheets[0].cssRules
```

和上面样式表访问方法一样，规则列表也可以通过方括号访问与 item 的方法，length 属性代表样式表数量

不过`Rules`的规则比较复杂，可以为 css 的`at-rule`也可以是普通的样式规则

at-rule 规则列表

- CSSStyle​Rule
- CSSCondition​Rule
- CSSGrouping​Rule
- CSSKeyframe​Rule
- CSSNamespace​Rule
- CSSMedia​Rule
- CSSPage​Rule
- CSSRule
- CSSSupports​Rule

### 简单介绍 CSSStyle​Rule

CSSStyleRule 属性有 `selectorText`与`style`

**CSSStyleRule.selectorText**
获取此规则的选择器的文本表示，例如 “h1，h2”

**CSSStyleRule.style**
返回规则的 CSSStyleDeclaration 对象。 只读。

[CSSStyle​Rule 详情](https://developer.mozilla.org/zh-CN/docs/Web/API/CSSStyleRule)

## 获取一个元素经过 css 计算后的属性

```
window.getComputedStyle(element, pseudoElt);
```

**element**
用于获取计算样式的 Element。

**pseudoElt** 可选
指定一个要匹配的伪元素的字符串。必须对普通元素省略（或 null）。

返回的 style 是一个实时的 CSSStyleDeclaration 对象，当元素的样式更改时，它会自动更新本身。

Window.getComputedStyle()方法返回一个对象，该对象在应用活动样式表并解析这些值可能包含的任何基本计算后报告元素的所有 CSS 属性的值。 私有的 CSS 属性值可以通过对象提供的 API

## cssom View

可以分为窗口部分、滚动部分、布局部分

### 窗口部分

用于操作浏览器窗口的位置、尺寸
window.moveTo(x, y)
window.moveBy(deltaX, deltaY)
window.resizeTo(aWidth, aHeight)
window.resizeBy(xDelta, yDelta)
**注：**
本函数按照指定的绝对位置移动当前窗口，而 window.moveBy 函数按照与当前位置相对的距离移动当前窗口。

从 Firefox 7 开始,如果符合下列情况,则普通网页中的 JavaScript 无法通过调用该函数来移动浏览器窗口

当前窗口或标签页不是由 window.open 方法创建的
当前标签页所在的窗口包含有多个标签页

### 滚动部分

**视口部分**
window.scrollX; 返回文档/页面水平方向滚动的像素值。pageXOffset 属性是 scrollX 属性的别名：

window.scrollY; 文档从顶部开始滚动过的像素值。 pageYOffset 属性是 scrollY 属性的别名：

window.scroll(x-coord, y-coord) 滚动窗口至文档中的特定位置。

- x-coord 值表示你想要置于左上角的像素点的横坐标。
- y-coord 值表示你想要置于左上角的像素点的纵坐标。

window.scrollBy(x-coord, y-coord); 在窗口中按指定的偏移量滚动文档。
window.scrollBy(options)

- X 是水平滚动的偏移量，单位：像素。
- Y 是垂直滚动的偏移量，单位：像素。

options 是一个包含三个属性的对象：

- top 等同于 y-coord
- left 等同于 x-coord
- behavior 表示滚动行为，支持参数：smooth (平滑滚动)，instant (瞬间滚动)，默认值 auto，效果等同于 instant

window.scrollBy 滚动指定的距离，而 window.scroll 滚动至文档中的绝对位置。另外还有 window.scrollByLines, window.scrollByPages

window.scrollTo 同样能高效地完成同样的任务。想要重复地滚动某个距离，请使用 window.scrollBy.

监听视口滚动

```
document.addEventListener("scroll", function(event){
  //......
})
```

**元素滚动**

**Element.scrollTop**
属性可以获取或设置一个元素的内容垂直滚动的像素数。

一个元素的 scrollTop 值是这个元素的顶部到视口可见内容（的顶部）的距离的度量。当一个元素的内容没有产生垂直方向的滚动条，那么它的 scrollTop 值为 0。

**Element.scrollLeft**
属性可以读取或设置元素滚动条到元素左边的距离。

注意如果这个元素的内容排列方向（direction） 是 rtl (right-to-left) ，那么滚动条会位于最右侧（内容开始处），并且 scrollLeft 值为 0。此时，当你从右到左拖动滚动条时，scrollLeft 会从 0 变为负数（这个特性在 chrome 浏览器中不存在）。

**Element.scrollWidth**
是只读属性，表示元素内容的宽度，包括由于滚动而未显示在屏幕中内容

scrollWidth 值等于元素在不使用水平滚动条的情况下适合视口中的所有内容所需的最小宽度。 宽度的测量方式与 clientWidth 相同：它包含元素的内边距，但不包括边框，外边距或垂直滚动条（如果存在）。 它还可以包括伪元素的宽度，例如::before 或::after。 如果元素的内容可以适合而不需要水平滚动条，则其 scrollWidth 等于 clientWidth

**Element.scrollHeight**
这个只读属性是一个元素内容高度的度量，包括由于溢出导致的视图中不可见内容。

scrollHeight 的值等于该元素在不使用滚动条的情况下为了适应视口中所用内容所需的最小高度。 没有垂直滚动条的情况下，scrollHeight 值与元素视图填充所有内容所需要的最小值 clientHeight 相同。包括元素的 padding，但不包括元素的 border 和 margin。scrollHeight 也包括 ::before 和 ::after 这样的伪元素。

属性将会对值四舍五入取整。如果需要小数值，使用 Element.getBoundingClientRect().

**element.scroll**
Element 接口的 scroll（）方法将元素滚动到给定元素内的特定坐标集。

element.scroll(x-coord, y-coord)
element.scroll(options)

x-coord 是您希望显示在左上角的元素水平轴上的像素。
y-coord 是您希望显示在左上角的元素的垂直轴上的像素。
——或
options 是一个 ScrollToOptions 字典。

**element.scrollBy**
元素接口的 scrollBy()方法按给定的数值滚动元素。

element.scrollBy(x-coord, y-coord);
element.scrollBy(options)

x-coord 是要滚动的水平像素值。
y-coord 是要滚动的垂直像素值。
——或
options 是一个 ScrollToOptions 字典。

**element.scrollIntoView**
方法的作用是:将调用元素的元素滚动到浏览器窗口的可见区域。

element.scrollIntoView();
element.scrollIntoView(alignToTop); // Boolean parameter
element.scrollIntoView(scrollIntoViewOptions); // Object parameter

[scrollIntoView 详情](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView)

监听事件

```
element.addEventListener("scroll", function(event){
  //......
})

```

## 布局 Api

### 全局布局

![](https://raw.githubusercontent.com/HongHongMing/Images/master/15555582539772.jpg)

### 元素布局

**getClientRects()**
method 返回一个 DOMRect 对象列表，表示该范围占用的屏幕区域。 这是通过聚合范围内所有元素的 Element.getClientRects（）调用结果而创建的。

**getBoundingClientRect()** 返回元素的大小及其相对于视口的位置。
返回的值是一个 DOMRect 对象，它是 getClientRects（）为元素返回的矩形的并集，即与该元素关联的 CSS 边框。 结果是包含整个元素的最小矩形，具有只读左，上，右，下，x，y，宽度和高度属性，用于描述整个边框（以像素为单位）。 宽度和高度以外的属性相对于视口的左上角。

[getBoundingClientRect 详情](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)

**获取相对坐标**

```
var offsetX = document.documentElement.getBoundingClientRect().x - element.getBoundingClientRect().x;
```
