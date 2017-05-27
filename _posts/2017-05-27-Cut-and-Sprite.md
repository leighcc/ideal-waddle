---
layout: post
title:  "切图及自动生成 sprite"
date:   2017-05-26 23:20:00
categories: play
comments: true
---

切图是前端的基础技能之一，然而之前我一直用的是原始人的切图方法，后来觉得实在太浪费时间，而且还有误差，因此踩了一遍坑，把这个流程整理出来。

首先将需要切出来的图层全部拖到一个 PSD 文件，并把需要合并的图层`Ctrl + E`合并。然后，选择*文件 - 脚本 - 将图层导出到文件...*将所有图层导出为单个 PNG 图片。由于图层的名称最终会成为类名，因此这个时候就应该把图层命名好。

<!--more-->

## cutterman

如果需要切二倍图、三倍图之类的，可以用 cutterman（需要注册使用），窗口 - 扩展功能 - Cutterman，选择需要切的图层，导出即可。Photoshop cc 2014 安装这个插件的时候有点问题，只能安装[旧版本][1]（其实可以装，方法在下面），因此切完以后还需要给文件名添加`@2x`的后缀，网上找的批处理代码如下：

~~~bash
@echo off
setlocal enabledelayedexpansion
echo "请输入所要添加的标题前缀[不添请回车]"
set /p str1=
echo "请输入所要添加的标题后缀[不添请回车]"
set /p str2=

:chose
echo "是否应用到子文件夹中(Y/N)"
set /p cho=
if "%cho%"=="Y" goto 1
if "%cho%"=="y" goto 1
if "%cho%"=="N" goto 2
if "%cho%"=="n" (goto 2) else (goto chose)

:1
for /f "delims=" %%i in ('dir /a-d/b/s') do (if /i not "%%~fi"=="%~f0" ren "%%i" "%str1%%%~ni%str2%%%~xi")
goto 3

:2
for /f "delims=" %%i in ('dir /a-d /b *.*') do (if /i not "%%~fi"=="%~f0" ren "%%i" "%str1%%%~ni%str2%%%~xi")
goto 3

:3
pause
~~~

### Photoshop cc 2014 安装新版本 cutterman

下载官网的安装程序默认将插件放在`C:\Users\Administrator\AppData\Roaming\Adobe\CEP\extensions`路径下，把 cutterman 文件夹放到`C:\Users\Administrator\AppData\Roaming\Adobe\CEPServiceManager4\extensions`即可。

## [gulp-css-spriter][2]

> gulp.spritesmith 的功能强大一点，star 也多一点，建议使用 gulp.spritesmith。

    npm install gulp-css-spriter

可将 CSS 文件中引用的图片合并为 sprite，并给出相应的`background-position`值。

`gulpfile.js` 文件：

~~~javascript
gulp.task('css', function() {
	// 引用了单个图片的 CSS 文件
	return gulp.src('public/css/test.css')
		.pipe(spriter({
			// 在该路径下保存将要生成的 sprite
			'spriteSheet': 'public/img/spritesheet.png',
			// 需要处理的 CSS 文件到 sprite 的路径
			'pathToSpriteSheetFromCSS': '../img/spritesheet.png'
		}))
		// 输出的 CSS
		.pipe(gulp.dest('public/build/css'));
});
~~~

### 只转换带后缀的图片

有时候不是所有背景图片都想用 sprite，可以设置图片地址带`?__bg`后缀的才转为 sprite。修改`node_modules\gulp-css-spriter\lib\map-over-styles-and-transform-background-image-declarations.js`：从第 42 行开始，

~~~javascript
// background-image always has a url
if(transformedDeclaration.property === 'background-image') {
    return cb(transformedDeclaration, declarationIndex, declarations);
}
// Background is a shorthand property so make sure `url()` is in there
else if(transformedDeclaration.property === 'background') {
    var hasImageValue = spriterUtil.backgroundURLRegex.test(transformedDeclaration.value);

    if(hasImageValue) {
        return cb(transformedDeclaration, declarationIndex, declarations);
    }
}
~~~

替换为：

~~~javascript
//background-imagealwayshasaurl且判断url是否有?__bg 后缀
if(transformedDeclaration.property === 'background-image'&&/\?__bg/i.test(transformedDeclaration.value)){
    transformedDeclaration.value = transformedDeclaration.value.replace('?__bg','');
    return cb(transformedDeclaration,declarationIndex,declarations);
}
//Backgroundisashorthandpropertysomakesure`url()`isinthere且判断url是否有?__bg 后缀
else if(transformedDeclaration.property === 'background'&&/\?__bg/i.test(transformedDeclaration.value)){
    transformedDeclaration.value = transformedDeclaration.value.replace('?__bg','');
    var hasImageValue = spriterUtil.backgroundURLRegex.test(transformedDeclaration.value);
    if(hasImageValue){
        return cb(transformedDeclaration,declarationIndex,declarations);
    }
}
~~~

## [gulp.spritesmith][3]

    npm install gulp.spritesmith

spritesmith 不需要先准备一个实现写好`background`的 CSS，它直接将文件夹下的图片合并成 sprite，并生成 CSS，对比 gulp-css-spriter，`background-size`的参数也写好了。生成的 CSS 类型是以`icon + 文件名`来命名的。

~~~javascript
var gulp = require('gulp');
var spritesmith = require('gulp.spritesmith');

gulp.task('sprite', function () {
  var spriteData = gulp.src('public/cut/*.png').pipe(spritesmith({
    imgName: 'sprite.png',
    cssName: 'sprite.css'
  }));
  return spriteData.pipe(gulp.dest('public/build/css/'));
});
~~~

可以生成 SCSS，添加选项：

    cssFormat: 'scss'

使用的时候`@include`即可（bar 为图片名称）：

    .bar {
      @include sprite($bar);
    }

### 添加图片、CSS 压缩功能

~~~javascript
var gulp = require('gulp');
var buffer = require('vinyl-buffer');
var csso = require('gulp-csso');
var imagemin = require('gulp-imagemin');
var merge = require('merge-stream');
var spritesmith = require('gulp.spritesmith');

gulp.task('sprite', function () {
	var spriteData = gulp.src('public/cut/*.png').pipe(spritesmith({
		imgName: 'sprite.png',
		cssName: 'sprite.css',
	}));

	var imgStream = spriteData.img
		// DEV: We must buffer our stream into a Buffer for `imagemin`
		.pipe(buffer())
		.pipe(imagemin())
		.pipe(gulp.dest('public/img/'));

	// Pipe CSS stream through CSS optimizer and onto disk
	var cssStream = spriteData.css
		.pipe(csso())
		.pipe(gulp.dest('public/build/css/'));

	// Return a merged stream to handle both `end` events
	return merge(imgStream, cssStream);
});
~~~

其中`vinyl-buffer`将 vinyl 转换为 user buffer。`gulp-csso`用于压缩 CSS，`gulp-imagemin`压缩图片。

### 重设 retina 图片大小

由于 retina 图片的尺寸必须为 1 倍图的两倍，否则插件会[报错][5]，因此还需增加一步将尺寸为奇数的图片补全为偶数。首先需要安装 [GraphicsMagick][6]，安装成功以后，需要重启电脑（Windows）。

~~~javascript
var configs = {
	spritesSource: 'public/cut/*.png',
	spriteMithConfig: {
		imgName: 'sprite.png',
		cssName: 'sprite.css'
	}
}

gulp.task('sprite', function () {
	var spriteData = gulp.src(configs.spritesSource).pipe(spritesmith(configs.spriteMithConfig));

	var imgStream = spriteData.img
		// DEV: We must buffer our stream into a Buffer for `imagemin`
		.pipe(buffer())
		.pipe(imagemin())
		.pipe(gulp.dest('public/img/'));

	// Pipe CSS stream through CSS optimizer and onto disk
	var cssStream = spriteData.css
		.pipe(csso())
		.pipe(gulp.dest('public/build/css/'));

	// Return a merged stream to handle both `end` events
	return merge(imgStream, cssStream);
});
~~~

通过配置参数，还可以设置输出的样式文件类型（SCSS 等），sprite 的排列方式，是否在图片边缘留空，给类名添加前缀等。


[1]: http://blog.cutterman.cn/?p=46
[2]: https://github.com/MadLittleMods/gulp-css-spriter
[3]: https://github.com/twolfson/gulp.spritesmith
[4]: https://www.w3ctrain.com/2015/12/09/generating-sprites-with-gulp/
[5]: https://github.com/twolfson/gulp.spritesmith/issues/57
[6]: https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick-binaries/