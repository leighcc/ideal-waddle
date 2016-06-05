---
layout: post
title:  "移动端开发经验小结"
date:   2016-06-04 21:51:00
categories: play
comments: true
---

已经好久没有写博客了呢:relieved:，其实是因为我不想把博客当成笔记本，所以很多学习记录都写在印象笔记了。Whatever，还是太久没在这里写，那就随便记录一下后面再更新吧:relaxed:。

<!--more-->

## 像素

首先要知道两种像素。**逻辑像素**，设计中所用到的像素就是逻辑像素。**设备像素**，设备厂商规格表上写着的就是设备像素。它们之间存在一定的换算关系，这个系数就叫**dpr**，比如 iPhone 4 以上的 iOS 设备 dpr 为 2，到 iPhone 6s 甚至是 3。要得到 dpr，就要知道**ppi**，而 ppi 是设备像素和尺寸计算出来的，这里就不放计算方法了。在 chrome Dev Tools 里面可以直接看到 dpr 的。

而 dpr 可以理解为设备用 dpr 个设备像素去表现 1 个逻辑像素，所以给 iPhone 6 设计的时候要自动把它的设备像素除以 2，也就是 375 ＊ 677。当然设计稿的尺寸都是按照设备像素来设计的，这样放到设备上才有高清的效果。

除了 Retina 屏，浏览器在缩放视图的时候 dpr 也会改变，例如 200% 的时候 dpr 为 2。

## viewport

{% highlight HTML %}
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">
{% endhighlight %}
这一行语句就是用来设置 viewport 的。跟像素一样，viewport 也有两个。layout viewport，比如 iPhone 的值是980px。visual viewport，有时候是屏幕实际的宽度。浏览器欺骗网页，说自己是一个 980px 宽度的设备，网页按照这个尺寸布局好以后，再缩小到 visual viewport 上展现出来。上面的语句就是首页把 layout viewport 缩小到跟实际屏幕一样，网页在这个小屏幕上布局。
{% highlight HTML %}
<meta content="yes" name="apple-mobile-web-app-capable">
<meta content="black" name="apple-mobile-web-app-status-bar-style">
<meta content="telephone=no" name="format-detection">
<meta content="email=no" name="format-detection">
{% endhighlight %}
比较通用的`meta`设置还有这几个，从上到下的作用分别是，当网站添加到主屏幕快速启动方式，可隐藏地址栏，仅针对ios的safari；将网站添加到主屏幕快速启动方式，仅针对ios的safari顶端状态条的样式；忽略将页面中的数字识别为电话号码；忽略Android平台中对邮箱地址的识别。

## 自适应

主要是要活用 rem 和百分比。只需：
{% highlight CSS %}
/* 320px布局 */
html {
	font-size: 100px;
}
/* iPhone6* /
@media (min-device-width: 375px) and (max-device-width : 667px) and (-webkit-min-device-pixel-ratio : 2) {
	html {
		font-size: 117.1875px;
	}
}
{% endhighlight %}
在 iPhone 6 下字体就会变成 iPhone 5 的 1.17 倍。

## 工具

### Brower-sync

虽然 chrome 支持模拟移动端，但要考虑地址栏高度等问题，还需要在真机上查看。Brower-sync 可以实现在多终端上查看网页，滚动页面等操作都是同步的。使用方法：

1. 进入代码所在路径。
2. `browser-sync start --server --files "css/*.css, *.html"`，监听`css`和`html`文件，如果文件比较散落，可以执行`browser-sync start --server --files "**/*.css, **/*.html"`。
3. PC 和手机打开`http://localhost:3000`即可。

### Autoprefixer

这个不是移动端专用的了，属于`css`后处理器，处理兼容前缀。可以设定需要兼容的浏览器版本，Autoprefixer 再根据 Can I Use 里面的数据，自动添加前缀，还可以删除多余的前缀。从此可以跟 SCSS 的`mixin`告别了。