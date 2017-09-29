---
title: vue+cesium的项目搭建
date: 2017-09-26 18:09:08
tags:
---
## 前言
上周有一个在大屏幕上展示作业进展情况的需求，需要在一个地球上动态展示作业分布，以及一些其他的图表显示。我们打算采用流行的vue前端框架，地球的显示采用Cesium库，前端工程使用es6语法来写，使用webpack进行构建。  
由于新接触Vue框架以及Cesium库，因此在搭建前端工程的时候遇到了几个典型的问题，在这里进行总结记录。

## vue框架搭建

### 软硬件环境
* 操作系统：win2010
* 开发工具：vs code
* 软件环境：nodejs-7.7.3，npm-4.1.2

### 工程搭建
我们使用vue-cli命令行工具, 搭建使用webpack来构建的工程。

1. 安装vue-cli命令行工具：

        npm install -g vue-cli

2. 创建vue工程：

        vue init webpack my-proj
    
    > 上述命令中， `vue init` 是命令，`webpack` 表示使用webpack工程模版，`my-proj` 是自定义的工程名称。  
    > vue-cli命令行工具就不详细介绍了，网上的相关文章很多，[npm官方连接](https://www.npmjs.com/package/vue-cli)。使用 vue help命令，也能得到详细的帮助信息。  
    
    执行上述命令后，会有很多yes/no的选择，以及一些简单的输入。主要涉及到工程名称、描述等，是否引入测试框架等等，主要用于生成工程的package.json，根据实际需要选择和输入即可。  
    执行完所有输入后，vue-cli会将工程初始化出来，创建相应的目录和文件。目录结构如下：

        my-proj
        ├─build          // webpack配置文件以及用于构建的js文件
        ├─config         // webpack配置和构建文件中用到的配置信息文件
        ├─src            // 源代码目录
        │  ├─assets      // 静态文件目录，如图片等
        │  ├─components  // vue组件目录
        │  └─router      // vue-router文件目录
        ├─static         // 外部的静态文件，webpack会在构建的时候把这里的文件拷贝到构建目录中
        └─test           // 测试相关的文件目录，在创建工程时引入了测试框架的才会有
            └─unit
                └─specs
        // 下边其实还有babel、eslint、package.json等配置文件，window的tree命令没列出来。。。
    
    此时，依赖的包只是被写入到了package.json中，并没有安装。进入工程的根目录，安装依赖的包：

        npm install

3. 测试工程：  
    执行如下命令：
    
        npm run dev

    启动一个express服务器，自动打开默认浏览器，会看到一个vue的欢迎页面，说明工程创建成功。

4. webpack构建过程介绍
    下面是webpack构建用到的配置文件的介绍

        my-proj
        │  package.json                // 工程的npm配置文件
        │  
        ├─build
        │      build.js                // 构建产品版程序的启动文件
        │      check-versions.js       // 检查node、npm以及依赖包的版本
        │      dev-client.js           // 这个没研究是干啥的
        │      dev-server.js           // 构建开发版程序，启动express服务器的启动文件
        │      utils.js                // 提供一些处理目录等的工具函数
        │      vue-loader.conf.js      // vue加载器的配置文件，没仔细研究
        │      webpack.base.conf.js    // webpack的公共配置
        │      webpack.dev.conf.js     // webpack的开发版配置，会包含base.conf.js
        │      webpack.prod.conf.js    // webpack的产品版配置，会包含base.conf.js
        │      webpack.test.conf.js    // webpack的测试版配置，会包含base.conf.js
        │      
        ├─config
        │      dev.env.js              // 开发版的环境变量配置
        │      index.js                // 主配置文件，配置了构建的目录、项目的相对路径、静态文件子目录等，会在webpack.conf文件中引用
        │      prod.env.js             // 产品版的环境变量配置
        │      test.env.js             // 测试版的环境变量配置
        │     

  开发版的构建和启动流程
  1. 执行 `npm run dev` 启动开发版构建；
  2. npm调用node执行 build/dev-server.js(见package.json中scripts的配置)；
  3. 调用webpack，使用webpack.dev.conf.js配置进行构建；
  4. 构建好之后，根据config/index.js中配置的端口等启动express服务器；
  5. 调用默认浏览器，显示构建好的页面；

    > 注意：无论是express还是下边要讲的webpack-dev-server（应该也是express的一种），webpack构建出来的目录对他们来说都只有静态文件有用，程序文件如index.html、app.js都是直接在server的内存中的，不是直接使用构建目录下的文件。  
    > 以上是我个人的理解，例证就是：对于这里的express服务器，构建完成之后，就没有生成新的目录和文件。而对于webpack-dev-server，虽然生成了构建目录目录和文件，但是如果构建出来的js文件名是带hash值的，可以在Chrome浏览器的开发者工具中看到，页面加载的js的hash值，与构建目录下的js的hash值是不一样的。

  产品版的构建流程
  1. 执行 `npm run build`；
  2. 调用node执行 build/build.js；
  3. 引入rimraf组件，删除构建目录中的已有文件；
  4. 删除完成后，启动webpack使用webpack.prod.conf.js配置执行构建；

  > 这里对webpack的配置和使用就不做详细说明了，如果不清楚的可以看我之前了解webpack时写的[webpack资料总结](http://www.jianshu.com/p/cfa9e5240a23)。

## 引入Cesium库
在引入Cesium库的过程中，我们找到了以下几种方案，分别说明：
1. Cesium开源版（org网站）的[官方指导](https://cesiumjs.org/tutorials/cesium-up-and-running/)
    
    这个方案介绍的是在Cesium工程的基础上进行开发的流程，我们是想向vue工程中引入cesium，因此不太适用。
    > 当然如果想用传统方案，通过下载cesium整个包到工程目录中，然后在html里通 `<script>` `<link>` 来直接引用相关的js和css也是没有问题的。  
    这里我们想用webpack进行包管理和自动化构建，因此不采用此方案。

2. [webpack-cesium](https://www.npmjs.com/package/webpack-cesium),这个是后来找到的，还没使用，不做说明；
3. Cesium商用版（com网站）中找到的[Cesium and webpack](https://cesium.com/blog/2016/01/26/cesium-and-webpack/)
    
    这里介绍的通过webpack引入cesium方案正是我们想要的。下面就详细介绍引入cesium的辛酸过程。。。。

### 安装Cesium包
官方文档里给的例子中，是下载Cesium包，然后拷贝到工程的lib目录中，我们当然不会这么老实。。。在[这里](https://www.npmjs.com/package/cesium)我们知道cesium是可以通过npm安装的，太好了！执行下面命令：

        npm install cesium -S

安装完成后，会在工程的node_modules中找到cesium文件夹，目录说明如下：

        ├─Build                     // 预编译好的文件目录
        │  ├─Cesium                 // 预编译且压缩的文件
        │  └─CesiumUnminified       // 预编译未压缩的文件
        └─Source                    // 源代码目录

### 代码中引入
按照官方文档说明，在我们工程的vue地球组件中加入以下代码：

        window.CESIUM_BASE_URL = './static/Cesium';  // 构建好之后，我们要把Cesium的文件放在项目下的static/Cesium/文件夹中，因此要这样配置

        // 注意：这里要用Source下的文件，原因下边有说明
        let Cesium = require('../../node_modules/cesium/Source/Cesium.js');
        require('../../node_modules/cesium/Source/Widgets/widgets.css');

        // html元素，以及页面初始化viewer 不再列出

这里有三个地方与官方文档不一样：
1. `CESIUM_BASE_URL` 路径    
    我们因为按照vue-cli创建的工程中，默认要把webpack构建的出来的项目种，静态文件要放在static二级目录下，所以要这样配置。
2. 引用了Source下的文件  
    官方文档里建议引用预编译好的文件，即Build下的Cesium或者CesiumUnminified，我们这里引用的是Source下的文件。  
    因为如果引用Build下的文件，构建好的项目启动后，会在浏览器的控制台报出一个错误，导致页面不能显示：

            Uncaught Error: Cesium missing ThirdParty/pako_inflate

    解决方案就是换成Source下的文件，解决方案来源于 [这里](https://github.com/AnalyticalGraphicsInc/cesium/issues/5417) 的负一楼。
3. 这里没有采用`let Cesium = window.Cesium` 的写法  
    如果采用官方推荐的写法，当使用 `Cesium` 对象初始话地图的时候，页面中会报 `Cesium`对象undefined的问题。

### 启动开发服务器
此时，我们披荆斩棘，已经把代码写好了，该是启动服务器验证一下页面的时候了。启动开发服务器的命令：

        npm run dev 或者 npm start

#### 第一次出现的错误
这时我们发现构建过程中报了一个错误：

        ERROR  Failed to compile with 1 errors

        This dependency was not found:

        * fs in ./~/cesium/Source/ThirdParty/crunch.js

        To install it, you can run: npm install --save fs

在检查了package.json以及node_modules下文件夹后，我们确定是已经安装了fs包的。尽管如此，我们还是按照提示，再一次执行安装 `npm install -S fs` , 安装了之后再编译，还是报同样的错误。。。  
只能google了，在 [这里](https://github.com/AnalyticalGraphicsInc/cesium/issues/4838) 找到了解决方法，我们修改工程中 `build/webpack.base.conf.js` 配置文件，代码如下：
        
        ***
        module: {
            ***
        },
        // 配置的最外层
        externals: {
            'fs': true,
        }
> 注意：这里要修改base配置文件，因为dev、prod、test的都会merge这个文件，只用改一处地方就可以了

#### 第二次出现的错误

重新启动开发服务器（`npm run dev`），构建没有报错，页面打开了，但是只有个黑黑的背景以及报错信息。f12打开Chrome的控制台，我们发现有几个相同的报错：

        Error: Cannot find module "."

继续google，在 [这里](https://github.com/mmacaula/cesium-webpack/issues/4) 找到了解决方法。继续修改 `build/webpack.base.conf.js` 配置文件，代码如下：

        ***
        module: {
            rules: [
                ***
            ],
            ***,
            // module选项内
            unknownContextCritical: false,
            unknownContextRegExp: /^.\/.*$/
        },
        // 配置的最外层
        externals: {
            'fs': true,
        }

#### 第三次出现的错误

再次启动开发服务器（`npm run dev`），构建没有报错，页面打开了，出现了黑黑的地图背景，还有地图工具栏，但是等等，为什么有很多图片加载失败？  
官方文档中其实已经说过了：像Cesium这么复杂的库，你想只通过 `require('Cesium')` 一句代码就引入到你的工程中去，那就图样图森破了。在代码中写的 `window.CESIUM_BASE_URL` 就是用来配置指向其他依赖的文件的根目录的。  
既然想到了这个，那就有办法了，我们把编译好的Cesium依赖文件，拷贝到webpack构建的项目的指定目录下就行了吧，也就是指定给 `window.CESIUM_BASE_URL` 变量的目录。  
但是还有个问题，我们发现执行完 `npm run dev` 后，webpack根本就没有生成配置文件定义的dist目录，猜测它是直接把构建好的文件扔到了express服务器的内存中去执行了，没有构建目录，我们往哪儿拷贝，怎么让express服务器知道Cesium依赖的静态文件在哪儿？研究了半天，答案是 **没办法** ，我也很无奈啊！

#### 用tomcat验证方案

我们可以换个思路，但是首先我们想验证下我之前想到的Cesium依赖文件的想法是否是正确的，我是这样验证的：
1. 用 `npm run build` 执行产品级的构建，在工程下构建出了 *dist* 目录；
2. 从 *node_modules/cesium/Build* 中，拷贝Cesium目录到 *dist* 目录下, 你可以可以拷贝未压缩的，或者Source下的，应该都可以；
3. 拷贝 *dist* 目录到 *tomcat/webapps/* 中；
4. 启动tomcat，打开 http://localhost:8080/dist 来查看；

这个时候我们发现，页面会报js路径找不到，原因是js路径不对，变成 http://localhost:8080/static/js/xxx.js 了，发现是 *dist* 那层被吃了。。。这肯定构建的路径有问题（可以打开看构建好的index.html的源代码，js的引用都是src="/static/js/xxxxx.js"，这样就是从tomcat的web根目录来找了）。  
我们发现webpack配置中的output引用的是 *config/index.js* 中build的assetsPublicPath属性，这里赫然配置着 `"/"` ， 改成 `"./"` , 然后重新执行上述步骤， 哈哈，成功啦！

### webpack-dev-server
如果每次修改一段代码想看下效果，都得手工构建，然后重新部署到tomcat中看，我就问你烦不烦？  
一般人儿都烦，如果你骨骼惊奇，想法奇特，就当我没说，全文到此over。。。  
下面的内容是给一般人看的：
#### 安装webpack-dev-server  

        npm install -D webpack-dev-server

#### 添加配置文件
可以有几种方案：
+ 直接修改 *build/webpack.dev.conf.js* ，增加server配置，然后在package.json中配置启动server命令时，指定为该配置文件  
这个比较简单，但是配置的server启动命令会比较长。。。
+ 模仿dev-server.js，用server的api接口来启动server  
这个需要学习webpack-dev-server的api接口，比较麻烦，但是后续想增加功能时，更灵活
+ 与第一个方案类似，将 *build/webpack.dev.conf.js* 拷贝到工程根目录，改名，然后加server的配置  
这个是webpack-dev-server的默认方式，server会默认查找工程根目录下的webpack.config.js, webpack本身启动时，也是找个名称的配置文件的，所以一定要把文件名称改对了。

我们项目就是一个小页面，所以采用最后一个方案。
> 其他方案也都类似, 这里不再赘述。

#### 配置webpack.config.js
1. 将 *build/webpack.dev.conf.js* 拷贝到工程根目录，改名为 webpack.config.js；
2. 增加server配置

        module: {
            ***
        },
        ***,
        devServer: {
            contentBase: config.dev.assetsRoot,  // 代理的文件目录，直接使用构建的目录           
            port: config.dev.port,    // 服务端口
            historyApiFallback: true, // 不跳转
            inline: true,             // 实时刷新
            hot: true,                // 热加载
        },
3. 虚拟项目名称

    这个如果要用webpack-dev-server的热加载实时刷新功能（hmr，通俗讲就是启动服务器后，你一修改代码保存后，页面自动刷新，体现出你的修改，爽不？），就不能配置为 `/` 或者 `./`, 必须给个字符串，比如我们这里的 `assets` , 我也不知道为什么，网上就这么说，实践也确实是这样。。。  
    具体的配置，有两种方式：
    + webpack.config.js
    
            output: {
                path: config.dev.assetsRoot,
                publicPath: 'assets', // webpack-dev-server的hmr必须指定一个代理目录
            },
    + config/index.js
    
            dev: {
                ***,
                assetsPublicPath: 'assets', // 默认的是'/'
                ***
            }

    这里我们采用第一种方式，不修改config下的默认配置，使用自定义的配置来覆盖默认的。
4. 自动拷贝静态文件配置
    
    还记得上边步骤中，需要把Cesium的依赖文件拷贝到webpack构建的目录中吗？这里webpack可以在构建阶段帮你做。需要安装 `copy-webpack-plugin` 插件:

        npm install -D CopyWebpackPlugin

    在webpack.config.js中引入插件：

            const CopyWebpackPlugin = require('copy-webpack-plugin'); 
            ***
            plugins: [
                ***,
                // copy custom static assets
                new CopyWebpackPlugin([{
                    from: path.resolve(__dirname, './static'),
                    to: config.build.assetsSubDirectory,
                    ignore: ['.*'],
                }]),
            ],

    从 */my-proj/node_modules/cesium/Build* 中，拷贝Cesium目录到 */my-proj/static* 目录下, ，并将这个目录加入版本管理工具。
    > 这个拷贝在webpack.prod.conf.js是默认有的，因此不用再给其配置。

5. 配置启动命令
在package.json中，加入如下配置：

        "scripts": {
            ***,
            "webpack": "webpack",
            "server": "webpack-dev-server --open",
            "start": "webpack && webpack-dev-server --open"
        },

6. 开发环境测试

        npm start

    构建成功。。。  
    服务器启动成功。。。  
    浏览器自动打开。。。  
    页面显示正确。。。。

以上，大功告成！！！