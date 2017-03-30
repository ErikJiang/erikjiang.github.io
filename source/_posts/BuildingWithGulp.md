title: "使用 Gulp 构建工程"
date: 2016-01-29 17:00:27
categories: "技术"
tags: [gulp]
---

gulp 是什么？

如果你和我一样，曾做过 Linux C 相关开发，一定知道对于 C 一类的工程，都不可缺少一个编译构建工具 Automake ，我们通过在工程中编写 makefile 脚本文件，实现工程代码的检查、编译以及安装等等。

而在如今大前端工程化的背景下，愈来愈强调前端的工作流程，如果有一种工具，能够如 automake 一般，能够帮助我们处理检测、编译、自动发布以及文件压缩、文件Hash、等等事宜，那一定能简化不少工作，而 gulp 即是我所说的这样一种工具。

<!--more-->

---

Gulp 是一个基于 Node.js 的流式构建工具，有别与 Grunt 基于 I/O 的构建方式，Grunt 在处理过程中会产生一些中间态的临时文件，而任务会最终基于这些临时文件生成构建后的结果文件。在这个过程中 Grunt 会进行大量的 I/O 读写操作，那么如果遇到大型项目工程，Grunt 的构建处理速度将大大拖长，这也就是为什么 Gulp 渐渐取代 Grunt 的原因之一，如果想了解更多，可以试着阅读 [《 gulp vs grunt 》](http://segmentfault.com/a/1190000002491282 "gulp vs grunt") 这篇文章。

由于 Gulp 通过代码优于配置的策略不仅使得任务易于编写，同时也易于维护和阅读。Gulp 基于的 NodeJS 的流，使用管道 Pipe 的概念，这一级的输出作为下一级的输入，中间无需在硬盘上写入任何临时文件和目录，让任务构建的更加迅速。



###  一、Gulp 的基本安装 ：

这里我默认您已经安装了 Node.js 以及 NPM 包管理工具，首先需要全局安装 Gulp 工具：

	$ npm install gulp -g
	
然后，需要在已有的项目中安装 gulp 的依赖包（前提是项目中已存在 package.json 文件）：
	
	$ npm init	#如果项目中没有 package.json 文件，需要执行此命令生成，有则忽略；
	$ npm install gulp --save-dev	#安装 gulp 依赖包
	
一切顺利，将会发现 package.json 文件中的 devDependencies 项里多了一个 gulp 子项，且 node_modules 中也多了一个 gulp 的依赖包；



###  二、Gulp 插件的安装 ：

要使用 Gulp 中的功能，必须依赖于 Gulp 的插件，在 Gulp 中每个插件完成一个具体的功能任务，而实际的构建工作往往是多个任务的流程，所以就要涉及到多个插件的安装，那么具体要用到哪些插件呢？这里我大概列出了常用的几项，仅供参考。

* Sass compile (gulp-ruby-sass)		# css 预处理器编译
* Autoprefixer (gulp-autoprefixer)	# css 浏览器兼容前缀处理
* Minify CSS with cssnano (gulp-cssnano)	# css 文本压缩
* JSHint (gulp-jshint)	# JS 代码检测
* Concatenation (gulp-concat)	# 代码文本合并
* Uglify (gulp-uglify)	# 代码文本压缩
* Compress images (gulp-imagemin)	# 图片资源压缩
* LiveReload (gulp-livereload)	# 文件变动页面重载
* Caching of images so only changed images are compressed (gulp-cache)	# 图片资源缓存
* Notify of changes (gulp-notify)	# 任务完成通知提示
* Clean files for a clean build (del)	# 清理构建目录及文件

安装方式与安装 Gulp 依赖包命令一致：

	$ npm install [package-name] --save-dev

###  三、Gulp 任务文件的配置 ：
	
这里我直接使用本人 Github 上的一个聊天室项目 [node_chat](https://github.com/JiangInk/node_chat "node_chat") 来进行讲解。
	
	// 加载插件
	var del = require('del'),
		gulp = require('gulp'),
		cache = require('gulp-cache'),
		jshint = require('gulp-jshint'),
		concat = require('gulp-concat'),
		uglify = require('gulp-uglify'),
		rename = require('gulp-rename'),
		notify = require('gulp-notify'),
		cssnano = require('gulp-cssnano'),
		imagemin = require('gulp-imagemin'),
		livereload = require('gulp-livereload'),
		autoprefixer = require('gulp-autoprefixer');

	// 设置源路径
	var srcPaths = {
		scripts: ['./public/javascripts/*.js'],
		images: './public/images/*',
		styles: './public/stylesheets/*.css'
	};

	// 设置目标路径
	var destPaths = {
		all: './build/**',
		scripts: './build/script',
		images: './build/image',
		styles: './build/style'
	};

	// 任务：脚本代码验证
	gulp.task('analysis', function() {
		return gulp.src(srcPaths.scripts)
			.pipe(jshint())
			.pipe(jshint.reporter('default'))
			.pipe(notify({message: 'Analysis task complete.'}));
	});

	// 任务1. 脚本合并与压缩
	gulp.task('scripts', ['analysis'], function() {
		return gulp.src(srcPaths.scripts)
			.pipe(concat('main.js'))
			.pipe(gulp.dest(destPaths.scripts))
			.pipe(rename({suffix: '.min'}))
			.pipe(uglify())
			.pipe(gulp.dest(destPaths.scripts))
			.pipe(notify({message: 'Scripts task complete.'}));
	});

	// 任务2. 资源图片压缩
	gulp.task('images', function() {
		return gulp.src(srcPaths.images)
			// 压缩 imagemin 并缓存 cache 图片
			.pipe(cache(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true })))
			.pipe(gulp.dest(destPaths.images))
			.pipe(notify({message: 'Images task complete.'}));
	});

	// 任务3. 样式表的压缩
	gulp.task('styles', function() {
		return gulp.src(srcPaths.styles)
			.pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
			.pipe(gulp.dest(destPaths.styles))
			.pipe(rename({suffix: '.min'}))
			.pipe(cssnano())
			.pipe(gulp.dest(destPaths.styles))
			.pipe(notify({message: 'Styles task complete.'}));
	});

	// 任务：清理构建目录
	gulp.task('clean', function(cb) {
		// 清空 build 目录
		return del([destPaths.scripts, destPaths.images, destPaths.styles], cb);
	});

	// 默认任务
	gulp.task('default', ['clean'], function() {
		gulp.start('scripts', 'images', 'styles');
	});

	// 任务：监听文件变动
	gulp.task('watch', function() {
		// 监听文件变动
		gulp.watch(srcPaths.scripts, ['scripts']);
		gulp.watch(srcPaths.images, ['images']);
		gulp.watch(srcPaths.styles, ['styles']);
		// 创建 livereload 监听服务
		livereload.listen();
		// 若有改变自动重载刷新
		gulp.watch([destPaths.all]).on('change', livereload.changed);
	});
	
---
