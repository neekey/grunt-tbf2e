# grunt-tbf2e

**基于Grunt的前端的Build工具集合**

关于Grunt的详细使用方法请参考：
 * [grunt](https://github.com/cowboy/grunt)
 * [getting_started](https://github.com/cowboy/grunt/blob/master/docs/getting_started.md)

下面简单介绍`grunt-tbf2e`的使用，包含如果安装和配置Grunt并安装grunt-tbf2e作为其插件来打包您的项目。

## 如何开始

### 安装Grunt

我们使用[nodeJS](http://nodejs.org)自带的包管理工具[NPM](http://npmjs.org/)来安装grunt。由于希望能全局进行命令的调用，因此我们添加`-g`参数：

```bash
$ (sudo) npm install grunt -g
```
注意，对于非windows用户，如果你直接使用`npm install grunt -g`命令时提示`permition denied`，那么请在前面添加`sudo`命令来启用管理员权限。

等待安装完毕后，我们在控制台中验证是否安装成功：

```bash
$ grunt --version
```
如果正确输出了版本，比如`grunt v0.3.17`，那么说明安装成功了。如果你想了解更多关于grunt命令的使用，推荐使用`grunt --help`查看。

#### 注意

如果你是*windows*用户，请特别注意，你需要使用`grunt.cmd`来代替`grunt`来执行grunt命令，比如查看grunt帮助，windows用户需要在`CMD`中输入：

```bash
$ grunt.cmd --help
```

### 安装grunt-tbf2e

*Grunt非常棒*，但是有一点不够方便的是，所有的插件都必须安装在项目目录中，而不能像`grunt`本身一样安装在全局来发挥作用（或许以后会改进这一点^_^）。

因此，要在你的项目中使用`grunt-tbf2e`，你需要在控制台中`cd`到你的项目目录（或者说是你希望你的grunt配置文件出现的地方，或者是你希望执行打包操作的目录）。

比如我有一个项目的结构如下，并且我希望将配置文件放置在项目目录的根目录中：

```bash
	path/to/project/app
					├	build
					├	source
							├	js
							├	css							
```
于是我在控制台中`cd path/to/project/app`，到达项目目录根目录中。OK，接下来我们安装`grunt-tbf2e`到当前目录:

```
$ npm install grunt-f2e
```
注意到没有使用参数`-g`，这样才能讲模块安装到当前目录下！经过一段等待后npm会告诉你安装成功，这时候你再查看目录，你会发现多了一个`node_modules`目录（对nodeJS很熟悉的朋友请跳过这块啰嗦的说明^_^），目录如下：

```bash
	path/to/project/app
					├	node_modules
							├	grunt-tbf2e
					├	build
					├	source
							├	js
							├	css							
```
说明grunt-tbf2e安装完成。

### 配置我们的grunt.js

grunt很强大，但是它并不知道我们的项目中哪些文件需要压缩，哪些文件需要打包，所有我们需要编写一个`grunt.js`来告诉它做哪些事情，添加一个`grunt.js`在当前目录下：

```bash
	path/to/project/app
					├	node_modules
							├	grunt-tbf2e
					├	build
					├	source
							├	js
							├	css	
					├	grunt.js					
```

`grunt.js`如何编写呢？要想把所有细节说清楚需要写几篇文章！已经厌烦了大堆的文字说明了？好吧，作为程序员，你肯定在心中大喊：`show me the code!` Fine!!!直接上代码：

```js
/*global module:false*/
module.exports = function (grunt) {

    grunt.initConfig({

        meta:{
            /* 发布版本号 */
            publish: 20120925
        },
        watch:{
            ksp:{
                files:['source/js/**' ],
                tasks:['ksp:dev', 'base-build' ]
            },
            compass:{
                files:['source/sass/**'],
                tasks:['compass:dev', 'compass:release' ]
            }
        },
        // 对base.js进行压缩
        min: {
            base: {
                src: ['source/js/common/base.js'],
                dest: 'release/<%= meta.publish %>/js/base-min.js'
            }
        },
        // 复制base.js到release目录
        copy: {
            base: {
                files: {
                    'release/<%= meta.publish %>/js/': 'source/js/common/base.js'
                }
            }
        },
        // SASS -> CSS
        compass: {
            dev: {
                src: 'source/sass/',
                dest: 'source/css'
            },
            release: {
                src: 'source/sass/',
                dest: 'release/<%= meta.publish %>/css'
            }
        },
        ksp:{
            dev:{
                "name":"v2",
                "pub":'<config:meta.publish>',
                "main":[
                    "source/js/msg_center.js",
                    "source/js/list.js"
                ],
                "output":"release/{{pub}}/js/{{filename}}.js",
                "compress":'-min',
                "unicode":true,
                "wrapConfig":true
            }
        }
    });

    grunt.loadNpmTasks('grunt-tbf2e');

    // Build 先对js文件进行打包，然后对sass进行编译，并赋值css到release文件夹下
    grunt.registerTask('default', 'ksp compass base-build');

    // 对base.js的压缩 已经复制
    grunt.registerTask('base-build', 'copy:base min:base');

};

```

上面就是我前一个项目的配置文件，可以看到里面做了很多事情，比如文件的`copy`，压缩`min`，JS文件的打包`ksp`，动态样式文件的编译`compass`，甚至你还看到我使用了一些变量定义`<config:meta:publish>`。对，相信聪明的你肯定能猜到这些配置的含义，但是我还是要简单地啰嗦两句：

* 我们通过initConfig来配置任务，这些配置过的任务不一定被执行
* 我们通过`grunt.loadNpmTasks('grunt-tbf2e');`来告诉grunt我们需要使用`grunt-tbf2e`这个插件
* 我们通过grunt.registerTask来定义和组合任务

比如:

```js
grunt.registerTask('default', 'ksp compass base-build');

```

定义了`default`方法，也就是如果我们在控制台中直接输入:

```
$ grunt
```
它默认会执行的方法。

而`base-build`的定义，则使得我们可以通过在控制台中：

```
$ grunt base-build
```
来依次执行`copy:base min:base`两个任务。在这里简单地说明下`copy:base`的意思：执行`copy`任务中的一个子任务`base`。

注意到initConfig方法中，copy部分的定义下面有个`base`了吧？或许你会以为这个`base`有特殊的含义，其实它只是我随便取的一个子任务的名字，你可以根据自己的喜好定义多个子任务，比如：

```

grunt.initConfig({
	
	copy: {
		taskOne: {
		},
		taskTwo: {
		}
	}
});

```
注意，不是所有的功能，都支持多任务配置，具体要参考各个任务的具体说明和文档哦~


## 包含的插件

除了grunt的内置组件：

* [concat](/gruntjs/grunt/blob/master/docs/task_concat.md) - Concatenate files.
* [init](/gruntjs/grunt/blob/master/docs/task_init.md) - Generate project scaffolding from a predefined template.
* [lint](/gruntjs/grunt/blob/master/docs/task_lint.md) - Validate files with [JSHint][jshint].
* [min](/gruntjs/grunt/blob/master/docs/task_min.md) - Minify files with [UglifyJS][uglify].
* [qunit](/gruntjs/grunt/blob/master/docs/task_qunit.md) - Run [QUnit][qunit] unit tests in a headless [PhantomJS][phantom] instance.
* [server](/gruntjs/grunt/blob/master/docs/task_server.md) - Start a static web server.
* test - Run unit tests with [nodeunit][nodeunit].
* watch - Run predefined tasks whenever watched files change.

以外，`tbf2e`提供了以下额外组件：

#### [`copy`](https://github.com/gruntjs/grunt-contrib-copy/)
复制文件到制定目录.

#### [`less`](https://github.com/gruntjs/grunt-contrib-less/)
将LESS文件编译为CSS.

#### [`compass`](https://github.com/kahlil/grunt-compass)
在你的项目中使用Compass(SASS)

#### [`mincss`](https://github.com/gruntjs/grunt-contrib-mincss/)
对CSS文件进行压缩.

#### [`ksp`](https://github.com/neekey/grunt-ksp)
对遵循KISSY 1.2 的包规范的脚本文件进行打包


## License
Copyright (c) 2012 neekey  
Licensed under the MIT license.
