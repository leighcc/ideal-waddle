---
layout: post
title:  "Learn Gulp"
date:   2016-07-04 20:42:00
categories: play
comments: true
---

没有太多概念解析的文字，用法也不是特别 fashion。记录一下从零开始建立一个 gulp workflow 的过程。

<!--more-->

## 基础
进入项目目录，输入 npm init ，会提示输入 name 等选项，按 Enter
采用默认值，此操作会在项目根目录生成一个 package.json 文件，记录项目依赖。在 package.json 文件中最底下输入

~~~javascript
"devDependencies": {
  	"gulp": "*"
}
~~~

记得在上一项的后面加上逗号。* 表示安装 gulp 的最新版本。在命令行中输入 npm install ，node 就会自动扫描 package.json 中的信息并安装依赖。安装完成后，目录会生成一个 node_modules 目录，并且会有 gulp 。

在根目录中新建一个 gulpfile.js 。为了将项目文件和依赖隔离开，可以新建一个 public 文件夹，将项目文件放进去。

![directory](http://7xrci6.com1.z0.glb.clouddn.com/dir.png)
在 gulpfile 中输入

~~~javascript
var gulp = require('gulp');

// modifies styles
gulp.task('styles', function () {
	console.log('starting styles');
});

// modifies scripts
gulp.task('scripts', function () {
	console.log('starting scripts');
});

//runs all gulp tasks
gulp.task('default', function () {
	gulp.start('styles', 'scripts');
});
~~~

 require 会将指定 mode_modules 里的模块包含进来。在命令行中输入 gulp [任务] 会执行特定的任务，输入 gulp 会执行 default 任务，也就是先执行 styles 任务，再执行 scripts 任务。

接下来安装第一个 gulp 插件。命令行输入 npm install gulp-uglify --save-dev 。这样 node 不会扫描 package.json 文件，而是直接安装名叫 gulp-uglify 的插件， --save-dev 则会在 package.json 中添加 gulp-uglify 的依赖信息。 package.json 的意义在于，如果想在另一个项目中安装同样的依赖，不需要将 node_modules copy 过去，copy json 文件，然后命令行 npm install 即可。

gulp-uglify 安装完成以后，在 gulpfile 中引入它， uglify = require('gulp-uglify') ， scripts 任务改为：

~~~javascript
gulp.task('scripts', function () {
	console.log('starting scripts');
	gulp.src('public/js/main.js')
		.pipe(uglify())
		.pipe(gulp.dest('public/build/js'));
});
~~~

此任务会将压缩后的 js 文件输出到 build/js 文件夹中。记得要将 html 文件中引入的文件改为压缩后的 js 文件。

## 监听和实时更新

接下来建一个静态服务器。在根目录下新建一个 server.js 文件，并安装 connect 插件。

~~~javascript
var http = require('http'),
	connect = require('connect'),
	serveStatic = require('serve-static');
	app = connect();

app.use(serveStatic(__dirname + '/public'));

http.createServer(app).listen(3000, function () {
	console.log('server is live on port 3000!');
});
~~~
 http 模块是包含在 node 里的，所以不需要额外安装。 app 代表了服务器， connect 方法会启动一个可配置的 mini connect 应用，放在服务器上。 app.use 告诉 connect 应用我们想要以 public 目录建立一个静态服务器。 __dirname 代表本文件所在的目录。服务器启动后会输入 server is live on port 3000! 。由于高版本的 NodeJS 不支持 connect ，所以还需要引入 serve-static 。命令行输入 node server.js 执行里面的内容。

接下来我们需要监听 js 文件，一旦它发生修改，马上将其压缩。在 gulpfile 中添加以下内容：

~~~javascript
gulp.task('watch', function () {
	gulp.watch('public/js/main.js',function () {
		gulp.start('scripts');
	});
});
~~~
命令行输入 gulp watch 即可监听。

实时更新。命令行 npm install gulp-livereload --save-dev 。修改 watch 和 scripts 任务。

~~~javascript
// modifies scripts
gulp.task('scripts', function () {
	console.log('starting scripts');
	gulp.src('public/js/main.js')
		.pipe(uglify())
		.pipe(gulp.dest('public/build/js'))
		.pipe(livereload());
});

// watches files
gulp.task('watch', function () {
	livereload.listen();

	gulp.watch('public/js/main.js', ['scripts']);

	gulp.watch('public/index.html')
		.on('change', livereload.changed);
});
~~~

livereload.listen(); 表示启动一个本地的 livereload 服务器。另外 gulp.watch 第二个参数可以是一个包含任务名的数组。在 index.html 中还需要引入一个 js 文件以保证浏览器和 livereload 服务器连接起来：
 `<script type="application/javascript" src="http://localhost:35729/livereload.js"></script>`
 
> 如果对 js 文件的刷新也像 html 文件那样， gulp.watch('public/js/main.js', ['scripts']).on('change', livereload.changed); ，会造成先刷新页面再压缩 js 文件。

## CSS 和 SCSS

下一步我们对 CSS 文件也进行处理（压缩和实时刷新）。在这里用到的插件是 gulp-minify-css ，同样作为依赖安装，引入， styles 任务变成：

~~~javascript
// modifies styles
gulp.task('styles', function () {
	console.log('starting styles');
	gulp.src('public/css/styles.css')
		.pipe(minifyCSS())
		.pipe(gulp.dest('public/build/css/'))
		.pipe(livereload());
});
~~~

记得在 watch 任务中添加对 CSS 文件的监听，以及在 html 中引入 build 文件夹中的 CSS。

接下来介绍的是自动给 CSS 属性添加兼容前缀。用到的插件为 gulp-autoprefixer ,跟前面一样，安装插件，引入。 styles 任务变成：

~~~javascript
// modifies styles
gulp.task('styles', function () {
	console.log('starting styles');
	gulp.src('public/css/styles.css')
		.pipe(prefix('chrome 29'))
		.pipe(minifyCSS())
		.pipe(gulp.dest('public/build/css/'))
		.pipe(livereload());
});
~~~

 autoprefixer 使用时需要输入参数，以指定目标兼容浏览器，参数的写法参照[这里](https://github.com/ai/browserslist#queries)。

假如有一些第三方的 CSS 文件需要引入，为了减少 http 的请求数，需要把它们合并成一个文件。用到的插件是 gulp-concat-css 。把第三方的文件放在 public 文件夹下的 vendor 文件夹。新建任务如下：

~~~javascript
// modifies vendor styles
gulp.task("styles:vendor", function () {
	gulp.src('public/vendor/css/**/*.css')
		.pipe(concatCSS('vendor.min.css'))
		.pipe(minifyCSS())
		.pipe(gulp.dest('public/build/css/'))
		.pipe(livereload());
});
~~~

并在 watch 任务里增加监听 vendor 的文件。注意在 html 文件里引入新的 CSS 文件，并且要放在自己写的 CSS 文件上方。

接下来我们将 SCSS 文件编译成 CSS 文件。插件名字叫做 gulp-sass （由于在 windows 和 mac 中安装 gulp-sass 太麻烦[参考这里][1]，我改用了 gulp-ruby-sass ）

~~~javascript
gulp.task('styles', function () {
	console.log('starting styles');
	return sass('public/css/styles.scss', {stopOnError: false})
		.on('error', sass.logError)
		.pipe(prefix('chrome 29'))
		.pipe(minifyCSS())
		.pipe(gulp.dest('public/build/css/'))
		.pipe(livereload());
});
~~~

 gulp-ruby-sass 需要安装 ruby 和 sass >= 3.4。除此之外 win 7 还需要添加一个环境变量：控制面板 > 系统和安全 > 系统 > 更改设置 > 高级 > 环境变量... 添加一个变量名为 path，值为 D:\Ruby22-x64\bin 的环境变量。最后重启 node.js 命令提示行。注意 sass 中的选项，可使 SCSS 文件中出现错误时，输出错误信息，而且不中断 gulp 的执行。

## JavaScript

 gulp-jshint 是一款用于检查 JS 文件中的错误的工具。

~~~javascript
gulp.task('scripts', function () {
	console.log('starting scripts');
	gulp.src('public/js/main.js')
		.pipe(jshint())
		.pipe(jshint.reporter('default'))
		.pipe(jshint.reporter('fail'))
		.on('error', function () {
			this.emit('end');
		})
		.pipe(uglify())
		.pipe(gulp.dest('public/build/js/'))
		.pipe(livereload());
});
~~~

第一行只是检查 JS 文件中的错误，第二行用默认的 reporter 将错误信息在命令行中输出。第四行在出错时中止 jshint，而不是中止 gulp，使得不用重启 gulp。

用自定义的 reporter 可以美化 jshint 输出的信息。插件名字叫 jshint-stylish 。只需将 reporter 里的 default 字符串换成对应的插件变量即可。

~~~javascript
.pipe(jshint.reporter(stylish))
~~~

跟 CSS 一样，需要把几个 JS 文件连接起来，插件名字叫 gulp--concat 。注意要在 jshint 后面使用。参数为输出的文件名称。

~~~javascript
.pipe(concat('scripts.min.js'))
~~~


  [1]: https://github.com/cycold/cycold.github.io/issues/29
