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


