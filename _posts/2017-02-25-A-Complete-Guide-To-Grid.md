---
layout: post
title:  "Grid 完全指南"
date:   2017-02-24 21:46:00
categories: play
comments: true
---

CSS Grid 布局（也叫“Grid”），是一个二维的基于网格的布局系统，旨于彻底改变我们设计基于网格的 UI 的方式。CSS 一直被用于排版网页，但是它一直不是很好用。一开始我们用 table，后来是 float，positon 和 inline-block，但这些方法本质上都是 hack，并且忽略了一些重要功能（比如垂直居中）。flexbox 有一定的帮助，但是它是用于简单的一维布局，和不复杂的二维布局（Flexbox 和 Grid 其实可以很好的配合）。Grid 是第一个用来解决过去我们在做网页中遇到的排版问题的 CSS 模块。

<!--more-->

### CSS Grid Layout 发展过程

- 2010 由微软提出，最早在 IE 10 实施。
- 2011 年 4 月首次公开草案。
- 2015 年 3 月 2 日 Chrome 支持。
- 2016 年 9 月 29 日成为 W3C 候选标准。

有两个东西激发了我写这个指南。第一个是 Rachel Andrew 的书 [Get Ready for CSS Grid Layout][2]。这本书清晰而透彻的解读了 Grid，它也是这篇文章的基础。我强烈推荐你买来看看。第二个是 Chris Coyier 的 [A Complete Guide to Flexbox][3]，我用 Flexbox 的时候首先会查看它。那篇文章帮助了很多人，从它是 Google 搜索  “flexbox” 排第一 就知道了。

我写这个指南是为了按照规范中最新的版本来介绍 Grid。因此我不会介绍过时的 IE 语法，并且随着规范日渐成熟，我会尽力更新这篇指南。

## Grid 基础以及浏览器支持

用 Grid 是很简单的。只需要用`display: grid`定义一个容器作为 grid，`grid-template-columns`和`grid-template-rows`设置列数和行数，然后用`grid-column`和`grid-row`放置它的子元素。跟 flexbox 类似，grid item 的顺序放置顺序并不重要。用 CSS 可以把它们按照任何顺序排列，这个特性可以在媒体查询中重排你的 grid。想象一下给你的整个网页定义版式，然后完全适配不同的屏幕，这些都只要少量几行代码。Grid 是最强大的 CSS 模块。

**值得注意的是 Grid 还不能用在生产环境中**。目前它还是一个 [W3C 草案][4]，还没有被任何一个浏览器正确支持。IE 10 和 11 支持 Grid，但是是以一种过时的语法来实现的。今天为了尝试 Grid，最好的选择是 Chrome，Opera 或者 Firefox，并且打开一些特殊的 flag。在 Chrome 中，输入`chrome://flags/`并打开 experimental web platform features。这种方法在 Opera 同样适用。Firefox 的话，设置 layout.css.grid.enabled 的 flag。

下面这个浏览器支持表格我会持续更新：

| Chrome      |    Safari | Firefox  |Opera  |IE  |Android  |iOS  |
| :------: | :-----:| :--: |
| 29+ (Behind flag)  |不支持 |  40+ (Behind flag)  |28+ (Behind flag) |10+ (Old syntax) |不支持|不支持 |

除了微软，浏览器厂商似乎不想在规范成熟以前让 Grid 开放使用。这是一件好事，意味着我们不需要学习多种语法。

在生成环境中使用 Grid 只是时间问题。但是现在已经是时候来学习了。

## 重要的术语

在研究 Grid 以前，搞懂其中的概念也很重要。因为这里提到的术语在概念上是相似的，如果你一开始不是按照 Grid 规范上的定义来记忆的话很容易混淆。但也不用担心，这些术语并不多。

### Grid 容器

应用`display: grid`的元素，是 grid item 的直接父元素。在这个例子中`container`就是 grid 容器。

~~~vbscript-html
<div class="container">
  <div class="item item-1"></div>
  <div class="item item-2"></div>
  <div class="item item-3"></div>
</div>
~~~

### Grid 项目

grid 容器的子元素。在这个例子中`item`是 grid 项目，但`sub-item`不是。

~~~vbscript-html
<div class="container">
  <div class="item"></div> 
  <div class="item">
    <p class="sub-item"></p>
  </div>
  <div class="item"></div>
</div>
~~~

### Grid 线

组成 grid 结构的分割线。网格线可以是垂直的（“列线”）也可以是水平的（”行线”），他们位于行的一边或者列的一边。在这里黄色的线就是一条列线。

![列线][5]

### Grid 路径

相邻两条 grid 线之间的空间称为 grid 路径。你可以把它们当做 grid 的列或行。这个例子表示了第二和第三行线之间的 grid 路径。

![grid 路径](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-track.png)

### Grid 单元格

两条相邻的行线和两条相邻的列线之间的空间。它是 grid 的一个“单位”。在这个例子中展示了行线 1、2 和列线 2、3 之间的 grid 单元格。


![grid 单元格](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-cell.png)

### Grid 区域

四条 grid 线围起来的整个空间。一个 grid 区域可以由任意数量的 grid 单元格组成。这个例子中展示了行线 1 和 3，列线 1 和 3 之间的 grid 区域。

![grid 区域][6]

## Grid 属性目录

### grid 容器属性

#### display

将元素定义为 grid 容器并为里面的内容建立定义新的 grid 格式上下文。

值：

- **grid** - 生成一个块级 grid。
- **inline-grid** - 生成一个行级 grid。
- **subgrid** - 如果你的 grid 容器同时也是个 grid 项目（也就是嵌套 grid），你可以用这个属性来指定它的行、列宽继承父级元素而不需要另外指定。

~~~css
.container{
  display: grid | inline-grid | subgrid;
}
~~~

注意：`column`/`float`/`clear`和`vertical-align`对 grid 容器不起作用。

#### grid-template-columns grid-template-rows

用空格隔开的一串值来定义行和列的大小。这些值表示路径的大小，它们之间的空间则表示了行线。

值：

- **`<track-size>`** - 可以是长度、百分比或者表示 grid 剩余空间的几分之几（用`fr`作单位）
- **`<line-name>`** - 任意取的名字

~~~css
.container{
  grid-template-columns: <track-size> ... | <line-name> <track-size> ...;
  grid-template-rows: <track-size> ... | <line-name> <track-size> ...;
}
~~~

例子：
当你在路径值之间留下空格，grid 线会被自动分配数字作为名字：

~~~css
.container{
  grid-template-columns: 40px 50px auto 50px 40px;
  grid-template-rows: 25% 100px auto;
}
~~~

![例子][7]

你也可以给网格线指定名字。注意名字要放在方括号里面：

~~~css
.container{
  grid-template-columns: [first] 40px [line2] 50px [line3] auto [col4-start] 50px [five] 40px [end];
  grid-template-rows: [row1-start] 25% [row1-end] 100px [third-line] auto [last-line];
}
~~~

一条线可以有多个名字。例如这里第二条线就有两个名字：row1-end 和 row2-start：

~~~css
.container{
  grid-template-rows: [row1-start] 25% [row1-end row2-start] 25% [row2-end];
}
~~~

如果你的定义包含重复的部分，可以用`repeat()`来组织：

~~~css
.container{
  grid-template-columns: repeat(3, 20px [col-start]) 5%;
}
~~~

等价于：

~~~css
.container{
  grid-template-columns: 20px [col-start] 20px [col-start] 20px [col-start] 5%;
}
~~~

用`fr`单位可以将路径设为 grid 容器剩余空间的几分之几。下面的例子中会将每个项目设为容器宽度的三分之一：

~~~css
.container{
  grid-template-columns: 1fr 1fr 1fr;
}
~~~

剩余空间是指非弹性项目剩下的空间。本例中用`fr`指定的剩余空间不包括 50px：

~~~css
.container{
  grid-template-columns: 1fr 50px 1fr 1fr;
}
~~~

#### grid-template-areas

通过引用在`grid-area`中指定网格区域的名字，定义一个网格模板。重复区域的名字可以填充单元格。一个句号表示一个空的单元格。这个语法在视觉上就能识别出网格的结构。

值：

- **`<grid-area-name>`** - 用`grid-area`指定的 grid 区域名字。
- **`.`** - 句号代表一个空的单元格
- **`none`** - 没有网格区域被定义

~~~css
.container{
  grid-template-areas: "<grid-area-name> | . | none | ..."
                       "..."
}
~~~

例子：

~~~css
.item-a{
  grid-area: header;
}
.item-b{
  grid-area: main;
}
.item-c{
  grid-area: sidebar;
}
.item-d{
  grid-area: footer;
}

.container{
  grid-template-columns: 50px 50px 50px 50px;
  grid-template-rows: auto;
  grid-template-areas: "header header header header"
                       "main main . sidebar"
                       "footer footer footer footer"
}
~~~

上面的代码创建了一个四列宽，三行高的网格。上面一行由`header`区域组成。中间一行由两个`main`区域，一个空的单元格，以及一个`sidebar`区域组成。最后一行由`footer`组成。

![例子][8]

每一行要有相同数量的单元格。

你可以用任意数量相邻的句号来声明一个单独的空单元格。只要句号之间没有空格，它们就表示一个单元格。

注意不能用这个语法来命名网格线，只能命名区域。当你用这个语法区域两头的网格线会被自动命名。如果网格区域的名字是`foo`，区域前面的行线和列线会叫做`foo-start`，后面的行线和列线会叫做`foo-end`。这意味着部分网格线会有多个名字，比如上面例子中最左边的列线就有三个名字，`header-start`，`main-start`和`footer-start`。

#### grid-column-gap grid-row-gap

指定网格线的宽度。你可以把它当成设置栏距。

值：

- **`<line-size>`** - 一个长度值

~~~css
.container{
  grid-column-gap: <line-size>;
  grid-row-gap: <line-size>;
}
~~~

例子：

~~~css
.container{
  grid-template-columns: 100px 50px 100px;
  grid-template-rows: 80px auto 80px; 
  grid-column-gap: 10px;
  grid-row-gap: 15px;
}
~~~

![例子][9]

栏距只会在列之间或者行之间创建，不会出现在外边缘。

#### grid-gap

`grid-column-gap`和`grid-row-gap`的简写。

值：

- **`<grid-column-gap> <grid-row-gap>`** - 长度值

~~~css
.container{
  grid-gap: <grid-column-gap> <grid-row-gap>;
}
~~~

例子：

~~~css
.container{
  grid-template-columns: 100px 50px 100px;
  grid-template-rows: 80px auto 80px; 
  grid-gap: 10px 15px;
}
~~~

如果没有指定`grid-row-gap`的值，就默认等于`grid-column-gap`的值。

#### justify-items

在行轴上对齐 grid 项目里面的内容（`align-items`则是用于在列方向上对齐）。这个值会应用到容器内所有的 grid 项目。

值：

- **start** - 将内容对齐到网格区域的左边
- **end** - 将内容对齐到网格区域的右边
- **center** - 将内容对齐到网格区域的中间
- **stretch** - 填满网格区域的整个宽度（默认）

~~~css
.container{
  justify-items: start | end | center | stretch;
}
~~~

| start      |    end | center  | stretch  |
| :--------: | :--------:| :--: | :--: |
| ![例子][10]  | ![例子][11] |  ![例子][12]   |![例子][13]|

每个 grid 项目上的对齐可以通过`justify-self`来设计。

#### align-items

在列方向上对齐 grid 项目里面的内容（作为对比，`justify-items`是用于对齐行方向上的内容）。这个值对容器内的所有项目生效。

值：

- **start** - 将内容对齐到网格区域的上方
- **end** - 将内容对齐到网格区域的底部
- **center** - 将内容对齐到网格区域的中间
- **stretch** - 将内容填充网格区域的整个高度（默认）

~~~css
.container{
  align-items: start | end | center | stretch;
}
~~~

| start      |    end | center  | stretch  |
| :--------: | :--------:| :--: | :--: |
|![例子][14]|![例子][15]|![例子][16] |![例子][17]|

要设置单个 grid 项目中的对齐方式可以用`align-self`。

#### justify-content

有时候所有的 grid 项目加起来都比 grid 容器小。如果所有的 grid 项目都被设为非弹性例如用`px`作单位时就会这样。这个时候你可以设置这些项目在容器中的对齐方式。这个属性设置的是行方向的对齐（作为对比`align-content`设置的是列方向上的对齐）。

值：

- **start** - 将项目对齐到容器的左边
- **end** - 将项目对齐到容器的右边
- **center** - 将项目对齐到容器的中间
- **stretch** - 调整项目的大小以填充容器的整个宽度
- **space-around** - 在项目之间放置相等大小的间隙，两端的间隙是项目间间隙的一半。
- **space-between** - 在项目之间放置相等大小的间隙，两端没有间隙。
- **space-evenly** - 在项目之间放置相等大小的间隙，两端的间隙跟项目间的间隙相等。

~~~css
.container{
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;  
}
~~~

| start      |    end | center  | stretch  | space-around  | space-between  | space-evenly  |
| :--------: | :--------:| :--: | :--: |
|![1][18]|![2][19]|![3][20] |![4][21]|![5][22]|![6][23]|![7][24]|

#### align-content

跟`justify-content`类似。

| start      |    end | center  | stretch  | space-around  | space-between  | space-evenly  |
| :--------: | :--------:| :--: | :--: |
|![1][25]|![2][26]|![3][27] |![4][28]|![5][29]|![6][30]|![7][31]|

#### grid-auto-columns grid-auto-rows

指定任意自动生成的 grid 路径（也称为隐式 grid 区域）的大小。当你通过`grid-template-rows`/`grid-template-columns`在网格范围以外放置行或列的时候，隐式 grid 路径就会被创建。

值：

- **`<track-size>`** - 可以是一个长度值，一个百分比或者用`fr`表示的占网格剩余空间的一部分。

~~~css
.container{
  grid-auto-columns: <track-size> ...;
  grid-auto-rows: <track-size> ...;
}
~~~

要描述隐式 grid 路径是怎么被创建的，请看一下例子：

~~~css
.container{
  grid-template-columns: 60px 60px;
  grid-template-rows: 90px 90px
}
~~~

![网格][32]

创建了一个 2 x 2 的网格。

然后用 `grid-column` 和 `grid-row` 去定位网格项目：

~~~css
.item-a{
  grid-column: 1 / 2;
  grid-row: 2 / 3;
}
.item-b{
  grid-column: 5 / 6;
  grid-row: 2 / 3;
}
~~~

![定位网格项目][33]

我们告诉 .item-b 从列线 5 开始到列线 6 结束，但列线 5 和 6 从来未被定义过。因为我们引用了不存在的网格线，宽度为 0 的隐式路径就会被创建来填补这些空缺。可以用 `grid-auto-columns` 和 `grid-auto-rows`来指定这些隐式路径的宽度：

~~~css
.container{
  grid-auto-columns: 60px;
}
~~~

![隐式路径][34]

#### grid-auto-flow

如果有一些 grid 项目没有明确定位到网格中，自动定位算法就会生效以放置那些项目。这个属性控制的是自动定位算法如何工作。

值：

- **row** - 告诉自动定位算法按顺序逐行填充，如有必要则添加新行
- **column** - 告诉自动定位算法按顺序逐列填充，如有必要则添加新列
- **dense** - 告诉自动定位算法尝试填充排在前面的网格。

~~~css
.container{
  grid-auto-flow: row | column | row dense | column dense
}
~~~

注意`dense`可能会打乱项目的顺序。

~~~vbscript-html
<section class="container">
    <div class="item-a">item-a</div>
    <div class="item-b">item-b</div>
    <div class="item-c">item-c</div>
    <div class="item-d">item-d</div>
    <div class="item-e">item-e</div>
</section>
~~~

定义一个两行五列的网格，并设置`grid-auto-flow`为`row`（默认值）。

~~~css
.container{
    display: grid;
    grid-template-columns: 60px 60px 60px 60px 60px;
    grid-template-rows: 30px 30px;
    grid-auto-flow: row;
}
~~~

只指定了其中两个项目的位置：

~~~css
.item-a{
    grid-column: 1;
    grid-row: 1 / 3;
}
.item-e{
    grid-column: 5;
    grid-row: 1 / 3;
}
~~~

因为`grid-auto-flow`的值为`row`，我们的网格看起来会像是下面那样。注意没有指定位置的三个项目（item-b, item-c 和 item-d）是如何在可用的行中流动的：

![例子][35]

如果将`grid-auto-flow`设为`column`，item-b, item-c 和 item-d 会在列中流动：

~~~css
.container{
    display: grid;
    grid-template-columns: 60px 60px 60px 60px 60px;
    grid-template-rows: 30px 30px;
    grid-auto-flow: column;
}
~~~

![例子][36]

#### grid

grid 是以下所有属性的简写：`grid-template-rows`, `grid-template-columns`, `grid-template-areas`, `grid-auto-rows`, `grid-auto-columns`, 和`grid-auto-flow`。这个属性同样会将`grid-column-gap`和`grid-row-gap`设置为初始值，即使它们不能被显式的设置。

值：

- **none** - 设置所有子属性为初始值
- **`<grid-template-rows> / <grid-template-columns>`** - 分别设置`grid-template-rows`和`grid-template-columns`的值，其他的子属性设为初始值。
- **`<grid-auto-flow> [<grid-auto-rows> [ / <grid-auto-columns>] ]`** - 分别设置它们三个的值。如果`grid-auto-columns`的值忽略不写，就被设为跟`grid-auto-rows`的值一样。如果两个都省略，则设为初始值。

~~~css
.container{
    grid: none | <grid-template-rows> / <grid-template-columns> | <grid-auto-flow> [<grid-auto-rows> [/ <grid-auto-columns>]];
}
~~~

下面两种写法是等价的：

~~~css
.container{
    grid: 200px auto / 1fr auto 1fr;
}
~~~

~~~css
.container{
    grid-template-rows: 200px auto;
    grid-template-columns: 1fr auto 1fr;
    grid-template-areas: none;
}
~~~

下面两种写法也是等价的：

~~~css
.container{
    grid: column 1fr / auto;
}
~~~

~~~css
.container{
    grid-auto-flow: column;
    grid-auto-rows: 1fr;
    grid-auto-columns: auto;
}
~~~

grid 属性也接受一种更复杂但也很快捷的语法去设置所有的值。你可以指定`grid-template-areas`, `grid-auto-rows`和`grid-auto-columns`的值，其他所有的子属性会被设为初始值。只需要在同一行内指定行线名和 grid 区域以及对应的路径大小
。用例子来说明：

~~~css
.container{
    grid: [row1-start] "header header header" 1fr [row1-end]
          [row2-start] "footer footer footer" 25px [row2-end]
          / auto 50px auto;
}
~~~

等价于：

~~~css
.container{
    grid-template-areas: "header header header"
                         "footer footer footer";
    grid-template-rows: [row1-start] 1fr [row1-end row2-start] 25px [row2-end];
    grid-template-columns: auto 50px auto;    
}
~~~

### grid 项目属性

#### grid-column-start grid-column-end grid-row-start grid-row-end

通过指定特定的 网格线，决定 gird 项目在 grid 里面的位置。`grid-column-start`/`grid-row-start`是网格项目开始的地方，而`grid-column-end`/`grid-row-end`是网格项目结束的地方。

值：

- **`<line>`** - 可以是一个代表网格线的数字，或者是名字。
- **span `<number>`** - 项目会扩展指定数目的网格路径。
- **span `<name>`** - 项目会一直扩展直到触碰到指定的网格线。
- **auto** - 表示自动放置，自动扩展，或者默认扩展一格。

~~~css
.item{
  grid-column-start: <number> | <name> | span <number> | span <name> | auto
  grid-column-end: <number> | <name> | span <number> | span <name> | auto
  grid-row-start: <number> | <name> | span <number> | span <name> | auto
  grid-row-end: <number> | <name> | span <number> | span <name> | auto
}
~~~

例子：

~~~css
.item-a{
  grid-column-start: 2;
  grid-column-end: five;
  grid-row-start: row1-start
  grid-row-end: 3
}
~~~

![grid item](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-start-end-a.png)

~~~css
.item-b{
  grid-column-start: 1;
  grid-column-end: span col4-start;
  grid-row-start: 2
  grid-row-end: span 2
}
~~~

![grid item](http://chris.house/images/grid-start-end-b.png)

如果没有指定`grid-column-end`/`grid-row-end`，项目会默认扩展一格。

项目可以互相重叠。你可以用`z-index`来控制堆叠的顺序。

#### grid-column grid-row

分别为`grid-column-start`+`grid-column-end`和`grid-row-start`+`grid-row-end`的缩写。

值：

- **`<start-line>` / `<end-line>`** - 两个都接受跟普通写法一样的值，包括 span。

~~~css
.item{
  grid-column: <start-line> / <end-line> | <start-line> / span <value>;
  grid-row: <start-line> / <end-line> | <start-line> / span <value>;
}
~~~

例子：

~~~css
.item-c{
  grid-column: 3 / span 2;
  grid-row: third-line / 4;
}
~~~

![grid item](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-start-end-c.png)

如果没有指定结束的网格线，项目默认扩展一格。

#### grid-area

给一个项目命名，这样就能在`grid-template-areas`属性里引用它。或者这个属性可以是`grid-row-start`+`grid-column-start`+`grid-row-end`+`grid-column-end`的缩写。

值：

- **`<name>`** - 你选的命名。
- **<`row-start>` / `<column-start>` / `<row-end>` / `<column-end>`** - 网格线的编号或者是名字。

~~~css
.item{
  grid-area: <name> | <row-start> / <column-start> / <row-end> / <column-end>;
}
~~~

例子：

给项目命名的一种方式：

~~~css
.item-d{
  grid-area: header
}
~~~

作为`grid-row-start`+`grid-column-start`+`grid-row-end`+`grid-column-end`的缩写：

~~~css
.item-d{
  grid-area: 1 / col4-start / last-line / 6
}
~~~

![grid item](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-start-end-d.png)

#### justify-self

在行方向上对齐网格项目内的内容（而`align-self`是在列方向上对齐）。这个值应用给单个网格项目里的内容。

值：

- **start** - 将内容对齐到网格区域的左端。
- **end** - 将内容对齐到网格区域的右端。
- **center** - 将内容对齐到网格区域的中间。
- **stretch** - 填满网格区域的整个宽度（默认值）。

~~~
.item{
  justify-self: start | end | center | stretch;
}
~~~

| start      |    end | center  | stretch  | 
| :--------: | :--------:| :--: | :--: |
|![start](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-self-start.png)|![end](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-self-end.png)|![center](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-self-center.png) |![stretch](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-self-stretch.png)|

要设置网格内所有项目的对齐方式，可以在网格容器上用`justify-items`来设置。

#### align-self

在列方向上对齐网格项目内的内容（而`align-self`是在行方向上对齐）。这个值应用给单个网格项目里的内容。

- **start** - 将内容对齐到网格区域的顶端。
- **end** - 将内容对齐到网格区域的底端。
- **center** - 将内容对齐到网格区域的中间。
- **stretch** - 填满网格区域的整个高度（默认值）。

| start      |    end | center  | stretch  | 
| :--------: | :--------:| :--: | :--: |
|![start](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-self-start.png)|![end](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-self-end.png)|![center](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-self-center.png) |![stretch](https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-self-stretch.png)|

要设置网格内所有项目的对齐方式，可以在网格容器上用`align-items`来设置。


  [1]: https://css-tricks.com/snippets/css/complete-guide-grid/
  [2]: http://abookapart.com/products/get-ready-for-css-grid-layout
  [3]: https://css-tricks.com/snippets/css/a-guide-to-flexbox/
  [4]: https://www.w3.org/TR/css-grid-1/
  [5]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-line.png
  [6]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-area.png
  [7]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-numbers.png
  [8]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-template-areas.png
  [9]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-column-row-gap.png
  [10]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-items-start.png
  [11]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-items-end.png
  [12]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-items-center.png
  [13]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-items-stretch.png
  [14]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-items-start.png
  [15]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-items-end.png
  [16]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-items-center.png
  [17]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-items-stretch.png
  [18]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-start.png
  [19]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-end.png
  [20]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-center.png
  [21]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-stretch.png
  [22]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-space-around.png
  [23]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-space-between.png
  [24]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-justify-content-space-evenly.png
  [25]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-start.png
  [26]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-end.png
  [27]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-center.png
  [28]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-stretch.png
  [29]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-space-around.png
  [30]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-space-between.png
  [31]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-align-content-space-evenly.png
  [32]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-auto.png
  [33]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/implicit-tracks.png
  [34]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/implicit-tracks-with-widths.png
  [35]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-auto-flow-row.png
  [36]: https://cdn.css-tricks.com/wp-content/uploads/2016/03/grid-auto-flow-column.png


