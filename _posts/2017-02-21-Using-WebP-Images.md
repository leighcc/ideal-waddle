---
layout: post
title:  "使用 WebP 图片"
date:   2017-02-21 21:46:00
categories: play
comments: true
---

很多网站都有一些吸引眼球的大尺寸图片，然而这些图片实在是太大了。在缓慢的移动端网速下，甚至可以看到这些图片在你面前展开。你突然感受到那些被拨号上网支配的恐惧。

<!--more-->

这是一个问题，因为对于典型的网站来说图片代表了下载内容的很大一部分。图片是昂贵的工具，它们比文字更有说服力。平衡视觉丰富的内容和快速的传递它是一个挑战。

对于这个困境解决方法是多种的。有很多压缩图片的技术，根据发出请求的设备的能力发送不同的图片。关于这方面的技术可以写成一本书了，但是这篇文章的侧重点是：Google 的 webP 图片格式，以及在维持图片的保真度基础上，获得更小的文件大小。现在就开始吧！

## 什么是 WebP，为什么需要它

WebP 是 Google 在 2010 年开发并首次发布的图片格式。它允许将图片有损或者无损编码，这使得它对于任何视觉媒体类型都能通用，也是 PNG 或者 JPEG 的一个很好的替代方案。WebP 是视觉质量通常都比得上常用的格式。下面是一组有损 WebP 图片和 JPEG 图片的对比：

![你能说出它们的区别吗？（提示：右边的是 WebP 图片）][1]

你能说出它们的区别吗？（提示：右边的是 WebP 图片。）

在上面的例子中，视觉上的区别几乎可以忽略，然而它们的文件大小区别却很大。左边的 JPEG 图片大小是 56.7 KB，右边的 WebP 图片大小几乎小了三分之一，为 38 KB。还不错，尤其是当你想到这两者视觉效果是相当的。

那么下一个问题，当然也就是“浏览器支持怎么样？”。没有你想的那么差。由于 WebP 是 Google 的技术，对于它的支持是固化在基于 Blink 的浏览器的。这些浏览器占了全球用户的一大部分，意味着截止写这篇文章的时间，[将近 73%][2] 在使用的浏览器都支持 WebP。如果你可以对三分之二的用户提升你网站的速度，你会拒绝吗？我想答案应该是不会。

然而，要记住，WebP 并不是完全取代 JPEG 和 PNG 图片。它是一个可以提供给支持它的浏览器的格式，但是你也应该要为那些不支持的浏览器保留旧的图片格式。这是 web 开发的本质：为支持的浏览器提供 Plan A，为那些不支持的浏览器提供 Plan B（或者是 Plan C）。

免责声明到此结束。让我们开始优化！

## 将你的图片转换为 WebP

如果你对 Photoshop 很熟悉，最容易尝试 WebP 的方法就是用 [WebP Photoshop 插件][3]。安装以后，你可以在 Save As 选项（不是 Save For Web！）中从格式下拉菜单中选择 WebP 或者
无损 WebP。

它们两者的区别是什么呢？想象 JPEG 和 PNG 的区别，WebP 和无损 WebP 的区别就差不多是这样。JPEG 是有损的，PNG 是无损的。当你转换 JPEG 图片的时候请选用普通 WebP 格式。当你转换 PNG 的时候请选用无损 WebP 格式。

当你用这个 Photoshop 插件保存无损 WebP 格式的时候，你不会得到任何提示。它会把所有事情搞定。当你选择有损的 WebP，你会得到类似的提示：

![有损 WebP 配置对话框][4]

有损 WebP 配置对话框

有损 WebP 的设置对话框能灵活的配置输出。你可以通过拖动从 0 到 100 的滑块来调整图片的质量（类似 JPEG），通过设置过滤模板的强度来得到更小的文件大小（会牺牲视觉质量），调节噪点过滤和锐化。

WebP Photoshop 插件有两点值得吐槽：没有一个 Save for Web  接口，你不能预览设置后的图片。如果你需要保存一堆图片，需要创建一个批处理。第二点，如果你喜欢 Photoshop 的批处理，对你来说可能不是个问题，但对于程序员来说，更倾向于用 Node 这类工具来一次性转换多张图片。

## 用 Node 将图片转换为 WebP

[Node.js][5] 很酷，而对于杂学不精的人比如我来说，对比将 JavaScript 带给服务端，它更是我构建网站的生产工具。在这篇文章中，我们将用一个叫 [imagemin][6] 的 Node 包同时将 JPEG 和 PNG 转换为 WebP 图片。

`imagemin`是图片处理的瑞士军刀，但我们现在只关注如何用它来将 JPEG 和 PNG 转换为 WebP 图片。但先别急！即使你之前没有用过 Node，这篇文章会告诉你所有步骤，如果你不想用 Node，那么直接用插件吧，这部分可以跳过。

第一步是[下载 Node.js][7] 并且安装它。这只需要几分钟。安装好以后，打开终端，进入你的项目的根目录。在那里，用 npm 安装`imagemin`和`imagemin-webp`插件：

    npm install imagemin imagemin-webp
    
安装过程需要大概一分钟。完成以后，打开你的文本编辑器，在项目根目录创建一个叫`webp.js`的文件。将以下代码放到文件中：

    var imagemin = require("imagemin"),    // The imagemin module.
      webp = require("imagemin-webp"),   // imagemin's WebP plugin.
      outputFolder = "./img",            // Output folder
      PNGImages = "./img/*.png",         // PNG images
      JPEGImages = "./img/*.jpg";        // JPEG images
    
    imagemin([PNGImages], outputFolder, {
      plugins: [webp({
          lossless: true // Losslessly encode images
      })]
    });
    
    imagemin([JPEGImages], outputFolder, {
      plugins: [webp({
        quality: 65 // Quality setting from 0 to 100
      })]
    });
    
这个脚本会处理`img`文件夹里的所有 JPEG 和 PNG 图片，并将它们转换为 WebP。当转换 PNG 图片，我们将`lossless`选项设为`true`。转换 JPEG 图片的时候，`quality`选项设为`65`。你可以调整这些参数得到不同的输出。在 [imagemin-webp 插件页面][8]中你还可以获得其他配置说明。

这个脚本假设你所有的 JPEG 和 PNG 图片都在`img`文件夹中。如果不是这样，可以改变`PNGImages`和`JPEGImages`变量。这个脚本同样假设你要将 WebP 输出到`img`文件夹。如果你不想这样，改变`outputFolder`的值。搞定以后，运行以下命令：

    node webp.js
    
这会处理所有的图片，并将 WebP 副本放到`img`文件夹中。文件大小能减小多少决定于你转换的图片。对于我来说，一个总大小为 2.75 MB 的 JPEG 文件夹在不损失视觉质量的情况下能减小到 **1.04 MB**。相当于毫不费劲就减小了**62%**！现在所有图片的转换完了，你可以开始使用它们了。让我们用起来！

## 在 HTML 中使用 WebP

在 HTML 中用 WebP 图像就跟用其他格式的图像一样，对吧？只要把它塞到`<img />`标签的`src`属性里，走你！

    <!-- Nothing possibly can go wrong with this, right? -->
    <img src="img/myAwesomeWebPImage.webp" alt="WebP rules." />
    
这样是 OK 的，但只是对于支持的浏览器来说。对于那些不幸的用户就悲剧了，如果你用的全是 WebP：

![就是这样][9]

很糟糕，是的，但这就是前端（微笑），因此振作起来。一些功能就是不可以在所有浏览器中通用，而且这种现状也不是马上可以改变。最简单的方法就是用`<picture>`元素来指定一系列的回退：

    <picture>
      <source srcset="img/awesomeWebPImage.webp" type="image/webp">
      <source srcset="img/creakyOldJPEG.jpg" type="image/jpeg"> 
      <img src="img/creakyOldJPEG.jpg" alt="Alt Text!">
    </picture>
    
这可能是保持最大兼容性的最佳实践了，因为对于每个浏览器都可以用，而不只是支持`<picuture>`元素的浏览器。这是因为不支持`<picture>`的浏览器只会显示`img`指定的图片。如果你需要全面的`<picture>`支持，可以参考[这篇文章][10]。

## 在 CSS 中用 WebP

如果你想在 CSS 中用 WebP 图片，情况会变得更复杂。不像 HTML 里面的`<picture>`元素在所有浏览器中可以优雅降级到`<img>`元素，CSS 没有最佳的內建图片降级方案。多背景的解决方式在某些情况下会下载所有资源。最佳的解决方案应该是特性检测。

[Modernizr][11] 是一个著名的特性检测库，可以检测浏览器可用的特性。WebP 支持也在它的检测范围内。更好的是，你可以在 [https://modernizr.com/download](https://modernizr.com/download) 创建只有 WebP 检测的自定义的 Modernizr，让你以很少的成本就能完成 WebP 的支持检测。

如果你是通过`<script>`标签来添加这个自定义的构造，它会自动给`<html>`元素添加两个类中的一个：

1. 如果浏览器支持 WebP，就会添加`webp`类。
2. 如果不支持，添加`no-webp`类。

有了这些类以后，就能根据浏览器的兼容用 CSS 来加载背景图片：

    no-webp .elementWithBackgroundImage {
      background-image: url("image.jpg");
    }
    
    .webp .elementWithBackgroundImage{
      background-image: url("image.webp");
    }
    
就这样。可以用 WebP 的浏览器会得到 WebP。不能用的就会回退到支持的图片类型。这就是双赢！除了……

## 如果用户禁用了 JavaScript 呢？

如果你要用 Modernizr，你要考虑那些禁用 JavaScript 的用户。对不起，事情就是这样的。如果你要用特性检测来将一部分用户蒙在鼓里，你就要检测 JavaScript 是否被禁用。使用上面的特性检测类，禁用 JavaScript 的浏览器甚至不会显示背景图片。这是因为被禁用的脚本从来都没有添加检测类到`<html>`元素。

为了应付这种情况，我们要先给`<html>`标签添加一个`no-js`类：

    <html class="no-js">
    
然后再写一点行内脚本放在其他`<style>`或者`<link>`标签前面：

    <script>
      document.documentElement.classList.remove("no-js");
    </script>
    
这行脚本在被解析的时候会去掉`<html>`上的`no-js`类。

那么这样做会有什么好处？当 JavaScript 被禁用了，这行脚本就不会运行，`no-js`就会留在元素上。这意味着我们可以添加另外一条规则来提供最广泛支持的图片类型：

    .no-js .elementWithBackgroundImage {
      background-image: url("image.jpg");
    }
    
这样就能覆盖大多数情况。如果 JavaScript 可用，行内脚本就会运行并且在 CSS 被解析之前就去掉了`no-js`类，支持 WebP 的浏览器就不会下载那张 JPEG 图片。如果 JavaScript 禁用了，那个类就不会去掉，兼容性更强的图片就会被使用。

这就是我们所做的一切，下面是我们可以预想的一些使用情况：

1. 可以用 WebP 的会得到 WebP。
2. 不能用 WebP 的会得到 PNG 或者 JPEG 图片。
3. JavaScript 禁用的会得到 PNG 或者 JPEG 图片。

## 总结

WebP 是一个全能的图片格式，我们可以用它来取代 PNG 和 JPEG（如果支持的话）。它能大大减小你网站图片的大小，众所周知，传输更少的数据能减小网页的加载时间。

有弊端吗？有一些。最大的一个就是你要管理两套图片以保证最大的兼容，如果你的网站有大量的图片需要转成 WebP 那可能就行不通了。还有一点就是如果你要在 CSS 中使用 WebP，就要用上一点 JavaScript。还有一点值得注意的就是用户如果将你的图片保存到本地，可能缺少一个查看 WebP 的应用程序。

对比省下来的流量所作的付出显然是相对少的，更快的加载可以提升你的网站的用户体验。对于用移动网络来浏览的用户更是如此。那么现在就开始用 WebP 吧！

  [1]: https://cdn.css-tricks.com/wp-content/uploads/2016/08/tacos-2x.jpg
  [2]: http://caniuse.com/#search=webp
  [3]: http://telegraphics.com.au/sw/product/WebPFormat#webpformat
  [4]: https://cdn.css-tricks.com/wp-content/uploads/2016/08/webp-plugin-2x.png
  [5]: https://nodejs.org/
  [6]: https://www.npmjs.com/package/imagemin
  [7]: https://nodejs.org/en/download
  [8]: https://www.npmjs.com/package/imagemin-webp
  [9]: https://cdn.css-tricks.com/wp-content/uploads/2016/08/broken-webp-2x.png
  [10]: https://github.com/scottjehl/picturefill
  [11]: http://modernizr.com/