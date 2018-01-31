## webpack配置详解
> 在解决webpack配置之前，先回答一个问题：**为什么使用npm  run 命令就能启动某个服务？**  

> 这就关于node的npm scripts ；详见 阮一峰老师的 [npm scripts 使用指南](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

### npm scripts (npm 脚本) 使用指南
1. 什么是npm脚本？
> npm 允许在 **package.json** 文件里面，使用**scripts**字段定义脚本命令  

EG：  
```
 {
    "scripts": {
        "dev": "",
        "build": "node build.js"
    }
 }

```
上述代码是**package.json**文件的一个片段，里面的**script**是一个对象，每一个属性就对应着一段脚本。  
命令行使用**npm run** 命令，就可以执行这段脚本
```
npm run build
// 等同于
node build.js
```
定义在**packjson**中的脚本就称为 npm 脚本。
> 优点  
- 项目相关的脚本可以集中在一个地方
- 不同项目的脚本命令，只要功能相同，就可以有同样的对外接口。用户不需要知道怎么测试你的项目，只要运行npm run test 即可。
- 可以利用npm 提供的很多辅助功能。
> 我好像只知道第一个功能 。。。

2. 原理
> 当执行 **npm run** 时，就会自动新建一个shell在这个shell里面执行制定的脚本命令。因此只要shell(bash)可以运行的命令，都可以写在npm 脚本里面。  

==**npm run** 新建的这个shell，会将当前目录的 node_modules/.bin子目录加入**PATH**变量，执行结束后，**PATH**变量恢复原样==  
> 这意味着当前目录的**node_modules/.bin**子目录中的所有脚本，都可以直接用脚本名调用，而不必加路径。  

EG ：当前项目的依赖项中有Mocha
```
// package.json
{
    "script": {
        "test": "mocha test"
    }
}
// 而不用写成
"test": "./node_modules/mocha test" 
```
3. 通配符
> 由于npm脚本本来就是shell脚本，因此可以使用shell通配符
```
// package.json
{
    "script": {
        "lint": "jshint  *.js", // * 表示任意文件名
        "lints": "jshint **/*.js"// * 表示任意一层子目录
    }
}
```
4. 传参  
向npm脚本传入参数，要使用 **--** 标明

5. 执行顺序
如果npm脚本里面需要执行多个任务，那么需要名曲他们的执行顺序。  
> 如果时并行执行(即同时的平行执行)，可以使用**&**符号
```
"build": "rimraf dist && cross-env NODE_ENV='production' webpack --config webpack.config.js --progress --profile --bail"
```
> 如果是继发执行（即只有前一个任务成功，才执行下一个任务），可以使用**&&**符号。
```
$ npm run script1.js && npm run script2.js
```
6. 默认值  
一般来说，npm 脚本由用户提供。但是，npm 对两个脚本提供了默认值。也就是说，这两个脚本不用定义，就可以直接使用。


    "start": "node server.js"，
    "install": "node-gyp rebuild"

上面代码中，npm run start的默认值是node server.js，前提是项目根目录下有server.js这个脚本；npm run install的默认值是node-gyp rebuild，前提是项目根目录下有binding.gyp文件。

7. 钩子  
npm 脚本有pre和post两个钩子。举例来说，build脚本命令的钩子就是prebuild和postbuild。


    "prebuild": "echo I run before the build script",
    "build": "cross-env NODE_ENV=production webpack",
    "postbuild": "echo I run after the build script"

用户执行npm run build的时候，会自动按照下面的顺序执行。


    npm run prebuild && npm run build && npm run postbuild

因此，可以在这两个钩子里面，完成一些准备工作和清理工作。下面是一个例子。


    "clean": "rimraf ./dist && mkdir dist",
    "prebuild": "npm run clean",
    "build": "cross-env NODE_ENV=production webpack"

8. ==简写形式==  
四个常用的 npm 脚本有简写形式。

        npm start是npm run start
        npm stop是npm run stop的简写
        npm test是npm run test的简写
        npm restart是npm run stop && npm run restart && npm run start的简写

npm start、npm stop和npm restart都比较好理解，而npm restart是一个复合命令，实际上会执行三个脚本命令：stop、restart、start。具体的执行顺序如下。

        prerestart
        prestop
        stop
        poststop
        restart
        prestart
        start
        poststart
        postrestart


9. 变量  
npm 脚本有一个非常强大的功能，就是可以使用 npm 的内部变量。

首先，通过npm_package_前缀，npm 脚本可以拿到package.json里面的字段。比如，下面是一个package.json。


    {
      "name": "foo", 
      "version": "1.2.5",
      "scripts": {
        "view": "node view.js"
      }
    }

那么，变量npm_package_name返回foo，变量npm_package_version返回1.2.5。


    // view.js
    console.log(process.env.npm_package_name); // foo
    console.log(process.env.npm_package_version); // 1.2.5

上面代码中，我们通过环境变量process.env对象，拿到package.json的字段值。如果是 Bash 脚本，可以用$npm_package_name和$npm_package_version取到这两个值。

npm_package_前缀也支持嵌套的package.json字段。


      "repository": {
        "type": "git",
        "url": "xxx"
      },
      scripts: {
        "view": "echo $npm_package_repository_type"
      }

上面代码中，repository字段的type属性，可以通过npm_package_repository_type取到。


```
const { resolve } = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require("webpack")

module.exports = (options = {}) => {
  return {
    /*
    配置页面入口js文件
    这里entry我们改用对象来定义
    属性名在下面的output.filename中使用, 值为文件路径
    */
    entry: {
      index: './src/app',
      vendor: './src/vendor'
    },

    // 配置打包输出相关
    output: {
      // 打包输出目录
      path: resolve(__dirname, 'dist'),

      /*
      entry字段配置的入口js的打包输出文件名
      [name]作为占位符, 在输出时会被替换为entry里定义的属性名, 比如这里会被替换为"index"
      [chunkhash]是打包后输出文件的hash值的占位符, 把?[chunkhash]跟在文件名后面可以防止浏览器使用缓存的过期内容,
      这里, webpack会生成以下代码插入到index.html中:
      <script type="text/javascript" src="/assets/index.js?d835352892e6aac768bf"></script>
      这里/assets/目录前缀是output.publicPath配置的
      options.dev是命令行传入的参数. 这里是由于使用webpack-dev-server启动开发环境时, 是没有[chunkhash]的, 用了会报错
      因此我们不得已在使用webpack-dev-server启动项目时, 命令行跟上--env.dev参数, 当有该参数时, 不在后面跟[chunkhash]
      */
      filename: options.dev ? '[name].js' : '[name].js?[chunkhash]',
    },

    module: {
      /*
      配置各种类型文件的加载器, 称之为loader
      webpack当遇到import ... 时, 会调用这里配置的loader对引用的文件进行编译
      */
      rules: [{
          /*
          使用babel编译ES6/ES7/ES8为ES5代码
          使用正则表达式匹配后缀名为.js的文件
          */
          test: /\.js$/,

          // 排除node_modules目录下的文件, npm安装的包不需要编译
          exclude: /node_modules/,

          /*
          use指定该文件的loader, 值可以是字符串或者数组.
          这里先使用eslint-loader处理, 返回的结果交给babel-loader处理. loader的处理顺序是从最后一个到第一个.
          eslint-loader用来检查代码, 如果有错误, 编译的时候会报错.
          babel-loader用来编译js文件.
          */
          use: ['babel-loader', 'eslint-loader']
        },

        {
          // 匹配.html文件
          test: /\.html$/,
          /*
          使用html-loader, 将html内容存为js字符串, 比如当遇到
          import htmlString from './template.html'
          template.html的文件内容会被转成一个js字符串, 合并到js文件里.
          */
          use: 'html-loader'
        },

        {
          // 匹配.css文件
          test: /\.css$/,

          /*
          先使用css-loader处理, 返回的结果交给style-loader处理.
          css-loader将css内容存为js字符串, 并且会把background, @font-face等引用的图片,
          字体文件交给指定的loader打包, 类似上面的html-loader, 用什么loader同样在loaders对象中定义, 等会下面就会看到.
          */
          use: ['style-loader', 'css-loader']
        },

        {
          /*
          匹配各种格式的图片和字体文件
          上面html-loader会把html中<img>标签的图片解析出来, 文件名匹配到这里的test的正则表达式,
          css-loader引用的图片和字体同样会匹配到这里的test条件
          */
          test: /\.(png|jpg|jpeg|gif|eot|ttf|woff|woff2|svg|svgz)(\?.+)?$/,

          /*
          使用url-loader, 它接受一个limit参数, 单位为字节(byte)
          当文件体积小于limit时, url-loader把文件转为Data URI的格式内联到引用的地方
          当文件大于limit时, url-loader会调用file-loader, 把文件储存到输出目录, 并把引用的文件路径改写成输出后的路径
          比如 views/foo/index.html中
          <img src="smallpic.png">
          会被编译成
          <img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAA...">
          而
          <img src="largepic.png">
          会被编译成
          <img src="/f78661bef717cf2cc2c2e5158f196384.png">
          */
          use: [{
            loader: 'url-loader',
            options: {
              limit: 10000
            }
          }]
        }
      ]
    },

    /*
    配置webpack插件
    plugin和loader的区别是, loader是在import时根据不同的文件名, 匹配不同的loader对这个文件做处理,
    而plugin, 关注的不是文件的格式, 而是在编译的各个阶段, 会触发不同的事件, 让你可以干预每个编译阶段.
    */
    plugins: [
      /*
      html-webpack-plugin用来打包入口html文件
      entry配置的入口是js文件, webpack以js文件为入口, 遇到import, 用配置的loader加载引入文件
      但作为浏览器打开的入口html, 是引用入口js的文件, 它在整个编译过程的外面,
      所以, 我们需要html-webpack-plugin来打包作为入口的html文件
      */
      new HtmlWebpackPlugin({
        /*
        template参数指定入口html文件路径, 插件会把这个文件交给webpack去编译,
        webpack按照正常流程, 找到loaders中test条件匹配的loader来编译, 那么这里html-loader就是匹配的loader
        html-loader编译后产生的字符串, 会由html-webpack-plugin储存为html文件到输出目录, 默认文件名为index.html
        可以通过filename参数指定输出的文件名
        html-webpack-plugin也可以不指定template参数, 它会使用默认的html模板.
        */
        template: './index.html'
      }),
      /*
      使用CommonsChunkPlugin插件来处理重复代码
      因为vendor.js和index.js都引用了spa-history, 如果不处理的话, 两个文件里都会有spa-history包的代码,
      我们用CommonsChunkPlugin插件来使共同引用的文件只打包进vendor.js
      */
      new webpack.optimize.CommonsChunkPlugin({
        /*
        names: 将entry文件中引用的相同文件打包进指定的文件, 可以是新建文件, 也可以是entry中已存在的文件
        这里我们指定打包进vendor.js
        但这样还不够, 还记得那个chunkFilename参数吗? 这个参数指定了chunk的打包输出的名字,
        我们设置为 [id].js?[chunkhash] 的格式. 那么打包时这个文件名存在哪里的呢?
        它就存在引用它的文件中. 这就意味着被引用的文件发生改变, 会导致引用的它文件也发生改变.
        然后CommonsChunkPlugin有个附加效果, 会把所有chunk的文件名记录到names指定的文件中.
        那么这时当我们修改页面foo或者bar时, vendor.js也会跟着改变, 而index.js不会变.
        那么怎么处理这些chunk, 使得修改页面代码而不会导致entry文件改变呢?
        这里我们用了一点小技巧. names参数可以是一个数组, 意思相当于调用多次CommonsChunkPlugin,
        比如:
        plugins: [
          new webpack.optimize.CommonsChunkPlugin({
            names: ['vendor', 'manifest']
          })
        ]
        相当于
        plugins: [
          new webpack.optimize.CommonsChunkPlugin({
            names: 'vendor'
          }),
          new webpack.optimize.CommonsChunkPlugin({
            names: 'manifest'
          })
        ]
        首先把重复引用的库打包进vendor.js, 这时候我们的代码里已经没有重复引用了, chunk文件名存在vendor.js中,
        然后我们在执行一次CommonsChunkPlugin, 把所有chunk的文件名打包到manifest.js中.
        这样我们就实现了chunk文件名和代码的分离. 这样修改一个js文件不会导致其他js文件在打包时发生改变, 只有manifest.js会改变.
        */
        names: ['vendor', 'manifest']
      })
    ],

    /*
    配置开发时用的服务器, 让你可以用 http://127.0.0.1:8080/ 这样的url打开页面来调试
    并且带有热更新的功能, 打代码时保存一下文件, 浏览器会自动刷新. 比nginx方便很多
    如果是修改css, 甚至不需要刷新页面, 直接生效. 这让像弹框这种需要点击交互后才会出来的东西调试起来方便很多.
    */
    devServer: {
      // 配置监听端口, 因为8080很常用, 为了避免和其他程序冲突, 我们配个其他的端口号
      port: 8100,

      /*
      historyApiFallback用来配置页面的重定向
      SPA的入口是一个统一的html文件, 比如
      http://localhost:8010/foo
      我们要返回给它
      http://localhost:8010/index.html
      这个文件
      配置为true, 当访问的文件不存在时, 返回根目录下的index.html文件
      index 值指定index.html文件的url路径
      */
      historyApiFallback: {
        index: '/src/'
      }
    },

    // 性能提示
    performance: {
      hints: options.dev ? false : 'warning'
    }
  }
}
```