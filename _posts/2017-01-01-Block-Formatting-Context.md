---
layout: post
title:  "Block Formatting Context"
date:   2017-01-01 15:33:00
categories: play
comments: true
---

BFC（Block Formatting Context，块级格式化上下文）是至少满足以下条件之一的一个盒子：
- `float`的值不是`none`。
- `overflow`的值不是`visible`。
- `display`的值是`table-cell`，`table-caption`，或者`inline-block`。
- `position`的值不是`static`或者`relative`。

<!--more-->

> 当提到[可视格式化模型](http://www.w3.org/TR/CSS21/visuren.html)的时候（此模型用于用户代理为视觉[媒体](http://www.w3.org/TR/CSS21/media.html)处理[文档树](http://www.w3.org/TR/CSS21/conform.html#doctree)），BFC 起到很大的作用。因此对于 CSS 使用者来说，要理解 BFC 和 flow，floats，clear 和 margins 的关系是很重要的。

## 规范是怎么写的……

### 块级格式化上下文如何流动（flow）

BFC 所属的[定位方法](http://www.w3.org/TR/CSS21/visuren.html#positioning-scheme)是常规流（normal flow）。因此，BFC 的“块”在页面流中正如你预期的那样，是跟[块](http://www.w3.org/TR/CSS21/visuren.html#block-box)级盒子，[行内格式化](http://www.w3.org/TR/CSS21/visuren.html#inline-formatting)的[行内](http://www.w3.org/TR/CSS21/visuren.html#inline-box)盒子，[相对定位](http://www.w3.org/TR/CSS21/visuren.html#relative-positioning)的块级或者行内盒子，[插入盒子](http://www.w3.org/TR/CSS21/visuren.html#run-in)一起定位的。简而言之，他们是页面流的一部分。

### 什么会触发 BFC

9.4.1 节写着以下的情况会建立新的 BFC：

- 浮动
- 绝对定位元素
- 行内块级元素
- 单元格
- 表格标题
- 任何带非`visible`的`overflow`值的元素
- 任何带`display: flex`或者`inline-flex`样式的元素

但是根据 [CSS 3 级规范](http://www.w3.org/TR/css3-box/#block-level0)，一个 BFC 在下面的情况中也会被创建：

- `position`的值不是`static`或者`relative`。

这定义意味着`position:fixed`也会建立一个 BFC。这并不是规范的失误，固定定位是绝对定位（9.6.1）的子类，并且绝对定位元素规范参考中写着，元素的定位属性有“absolute”和“fixed”的值。

注意`display:table`本身不会生成 BFC。但是因为它能够生成[匿名盒子](http://www.w3.org/TR/CSS21/tables.html#anonymous-boxes)，它们中的一个（带`display:table-cell`属性的）会建立一个新的 BFC。也就是说，触发 BFC 的是匿名盒子，而不是`display:table`。这一点需要大家注意，因为即使两种样式都会建立新的 BFC（显示或者隐式），`clear`用在`display:table`和`display:table-cell`也会不一样。

最后一个会触发 BFC 的是`fieldset`元素。很奇怪的是，在 [HTML5](http://www.w3.org/TR/html5/the-xhtml-syntax.html#the-fieldset-element-0) 规范以前，www.w3.org 上并没有关于这种行为的说明。有些浏览器 bug（webkit、Mozilla）会提到它，但不是官方的。实际上，即使`fieldset`元素在大多数浏览器中会建立 BFC，按照 3.2 节，不能把这个当成完全被支持的特性：

> CSS 2.1 没有定义可以应用给表单控件和框架（frames）的属性、或者 CSS 可以如何给它们设定样式。用户代理可能会应用 CSS 属性给这些元素。开发者应该把它当成实验性质。未来级别的 CSS 可能有这样的功能。

### BFC 做了什么

因为它们的流动方式，BFC 包含浮动元素。并且根据 9.4.1 节，BFC 阻止外边距收缩并且不会与浮动元素重叠：

> 在一个 BFC 中，盒子在垂直方向上从容器的顶部开始被一个接一个的排列。两个相邻的盒子的垂直距离被`margin`属性决定。一个 BFC 内相邻的两个块级盒子
之间的外边距会[收缩](http://www.w3.org/TR/CSS21/box.html#collapsing-margins)。

> 在一个 BFC 中，每个盒子的左外边缘触碰容器的左边缘（对于从右到左格式，触碰的是右边缘）。这对于浮动元素也是成立的（尽管一个盒子的线框会因为浮动而收缩），除非这个盒子建立了新的 BFC（这种情况下盒子本身可能会因为浮动而[变得更窄](http://www.w3.org/TR/CSS21/visuren.html#bfc-next-to-float)）。

## 规范到此为止，这些有什么实际用途？

BFC 行为上很像块级盒子，除了下面这些：

### BFC 阻止外边距收缩

垂直方向上的两个相邻的盒子间的外边距[收缩](http://www.w3.org/TR/CSS21/box.html#collapsing-margins)，但只有它们在同一个 BFC 中的时候。换句话说，如果两个相邻的盒子不属于同一个 BFC，它们的外边距不会收缩。

例子：

<div style="background:skyBlue">
<p style="margin:20px"> This is a paragraph inside a DIV with a blue background, styled with <code>margin:20px</code>. </p></div>

<div style="background:skyBlue">
<p style="margin:20px"> This is a paragraph inside a DIV with a blue background, styled with <code>margin:20px</code>. </p></div>

<div class="gainLayout" style="background:skyBlue;overflow:hidden">
<p style="margin:20px"> This is a paragraph inside a DIV with a blue background, it is styled with <code>margin:20px</code>, The parent DIV is styled with <code>overflow:hidden;zoom:1</code>. </p></div>

上面的两个蓝色盒子，段落的上外边距和底外边距收缩了（距离是 20px，而不是 40 px）。因为最后的 DIV 建立了一个新的 BFC，第三段的外边距不会收缩，因此它们不会“突出”到容器外面而是会成为容器的一部分。

**注意**：在 IE 6 中，没有显式的外边距，DIV 会包不住外边距。

涉及到外边距收缩，新建一个 BFC 跟给元素应用`border`或者`padding`是一样的。

### 包含浮动元素的 BFC

你肯定听说过“浮动元素总是能包含浮动元素”，或者是 FNE（float nearly everything [浮动一切](http://orderedlist.com/our-writing/blog/articles/clearing-floats-the-fne-method/)）方法。这些方法的根据**是** BFC，因此更确切的表达方式是“BFC 总能包含浮动元素”。

例子：

<div style="background:skyBlue">
<p style="float:left;margin:20px"> This paragraph is a float inside a DIV with a blue background, it is styled with <code>margin:20px</code></p></div>

<div class="gainLayout" style="background:skyBlue;overflow:hidden;clear:left">
<p style="float:left;margin:20px"> This paragraph is a float inside a DIV with a blue background, it is styled with <code>margin:20px</code>. The parent DIV is styled with <code>overflow:hidden;zoom:1</code>. </p></div>

第一段浮动了，因此脱离了文档流，父元素因此**收缩**，因此父元素的背景没有显示出来。

第二段也浮动了，但被新建了 BFC 的 DIV 包裹起来，因此容器包含了子元素的外边距盒子。你也应该注意到，尽管第一段浮动了，但第二段却不需要清除浮动。这通常被称为“自清除”，把 BFC 当成文档流的一部分会比较好理解。

**注意**：`clear`只清除了同一个 BFC 里面的浮动。

### BFC 不会跟浮动元素重叠

这是[我最喜欢](http://www.ez-css.org/)的一点。根据规范，BFC 的边框盒子一定不能与同一个 BFC （也就是它本身）内的浮动元素的外边距盒子重叠。这意味着浏览器在 BFC 上创建了隐式的外边距以防止它们与浮动元素的外边距盒子重叠。因为这个原因，负外边距对浮动元素旁边的 BFC 无效（然而 webkit 和 IE 6 不是这样，[测试用例](http://www.tjkdesign.com/lab/bfc/test.html)）。

例子：

<div style="background:skyBlue;float:left;width:180px"><pre style="background: transparent;">.sideBar { 
background: skyBlue; 
float: left; 
width: 180px; 
}</pre></div>
<div style="background:yellow;float:right;width:180px"><pre style="background: transparent;">.sideBar { 
background: yellow; 
float: right; 
width: 180px; 
}</pre></div>
<div class="gainLayout" style="background:pink;overflow:hidden;border:5px solid teal"><pre style="background: transparent;">#main { 
background: pink; 
overflow: hidden; 
zoom: 1; 
border: 5px solid teal; 
} </pre></div>

因为这个行为是附属于“边框盒子”（而不是“外边距盒子”）的，要在粉红色盒子和它的毗邻元素之间建立间隙的话，方法有两个：

- 给浮动元素设置 20px 的外边距
- 在粉红色盒子上设置比浮动元素宽带大的外边距（比如 margin:0 220px）

是的，你要用`220px`，而不是`20px`。因为是边框盒子试图在浮动元素之间适应，不是外边距盒子。我说它*试图*，是因为如果两个浮动元素之间没有足够的空间，中间的容器会掉到下面去。

换句话说，如果粉红色盒子是`400px`宽，如果父元素比`770px`（180px + 180px + 400px + 10px）窄它就会掉下去。注意，极少数情况下，这种行为在 firefox 中会被打破（例如，当以上的结构是`body`元素的第一个子元素，参照[测试用例](http://www.tjkdesign.com/lab/bfc/test.html)）。

**注意**：在 IE 6 中，粉红色盒子和两个浮动盒子的距离有一个 [3px 的 bug](http://www.positioniseverything.net/explorer/threepxtest.html)。

## hasLayout 对 BFC

你可能会注意到，所有之前的例子都是用`overflow:hidden;zoom:1`来设置样式的。前面的声明在现代浏览器中新建了一个 BFC，而后者在 IE 5.5/6/7 中触发了 hasLayout。这是因为这些渲染是非常接近的（[CSS 规范的相似](http://www.satzansatz.de/cssd/onhavinglayout.html#engineer)）。像 BFC 那样，元素被赋予一个会阻止外边距收缩、会包含浮动元素以及不覆盖浮动元素的特性。