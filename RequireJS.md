## 简介
[RequireJS](http://requirejs.org/docs/download.html) 是 [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD-(%E4%B8%AD%E6%96%87%E7%89%88)) 规范的一种实现  
所解决的问题如下：
1. 实现JS文件的异步加载，避免网页失去响应  
    说明：因为HTML的解析是从上往下的，在加载JS时，如果出现阻塞，这将会影响页面的显示
2. 管理模块之间的依赖性，便于代码的编写和维护
3. 避免变量污染全局环境

### 使用
在页面加载RequireJS
```
<script src="RequireJS路径" defer async="true" data-main="js/main"></script>
// async 代码此文件异步加载，避免网页失去响应。由于IE不支持async属性 只支持defer。
// data-main  指定 主模块加载位置
```
- define API
定义一个模块
```
// math.js
// 每个定义的模块最终由RequireJS处理后，所导出的
define(function(){
    var add = function (x,y){
        return x+y;
    };
    return {
        add : add
    }
})

// main.js 加载 math.js 模块
require(['math'],function(math){
    alert(math.add(1,1))
})

```
==注意：  
在主模块中，如果没有使用require.config()方法进行初始化定义，那么require的第一个参数(依赖项数组)中的元素会被RequireJS认为是路径去查找到对应的文件上述代码，会让RequireJS查找main.js同级的目录中的math.js  
但是如果有使用require.config()  
RequireJS会先查找paths中的键进行匹，若查找不到，则按路径的方式查找==  
#### 其他的地方使用依赖项也应该如上的过程

定义一个有依赖项的模块
```
// 第一个参数为：数组 指明改模块的依赖项
define(['myLib'],function(myLib){
    
})
// 当require()函数加载上面定义的模块时，会先加载myLib.js文件
```
- require API
此api用于模块的加载  
使用require.config()方法对模块的加载行为进行自定义。require.config()卸载主模块(main.js)的头部，参数是一个==对象==。该对象的paths属性指定各个模块的加载路径。
```
require.config({
    paths:{
        "jquery"："js/jquery.min",
        // 其实也可以写成
        // "jquery":["http://libs.baidu.com/jquery/2.0.3/jquery","js/jquery.min"]
        // 这样写，RequireJS首先会加载百度的jquery库，如果加载不成功，则会加载本地jquery库
        "underscore":"js/underscore.min"
    }
})
或
require.config({
    baseUrl: 'lib/',
    pathcs:{
        "jquery":"base/jquery.min",
        "underscore":"base/underscore.min"
    }
})
```
==并非所有的库都符合RequireJS规范==  
shim可以帮助我们加载不符合AMD规范的库
```
require.config({
    baseUrl: 'lib/',
    pathcs:{
        "jquery":"base/jquery.min",
        "underscore":"base/underscore.min"
    },
    shim:{
        'angular':{
            exports:'angular'
        },       
        'angular-ui-router':{
            deps:['angular']
        },
        'angular-sanitize':{
            deps:['angular']
        },
        'angular-file-upload':{
        	deps:['angular']
        }
    }
    // shim：
    //① exports值(输出的变量名)，表示这个模块外部调用时的名称 
    //② deps(type：数组) 表明该模块的依赖项
})
```
为什么要使用shim?
```
// hello.js
function hello(){
    alert("hello world")
}
// 这样的写法是不符合AMD规范的
若我们仍使用平常的方式进行使用

// main.js
require.config({
    paths: {
        hellos: 'hello'
    }
});
require(['hello'],function(hello){
    hello()
})
// 代码会报错：uncaught TypeError:undefined is not a function
// 这是因为在调用hello()的时候，hello是个undefined
// 这说明，虽然我们依赖了一个js库(它会被载入),但RequireJS无法获取到代表这个js库的对象，注入进来供我们使用
```
上述情况就需要我们，使用shim，将载入的JS库中的某个全局变量暴露给RequireJS，将这个全局变量，当作这个库本身的引用。
```
require.config({
    paths: {
        hello : 'hello'
    },
    shim: {
        hello: {exports : 'hello'}
        // 将hello模块中的hello全局变量暴露出来
    }
})
require(['hello'],function(hello){
    hello();// hello world
})
```
 上面代码 exports: 'hello' 中的 hello ，是我们在 hello.js 中定义的hello 函数。当我们使用 function hello() {} 的方式定义一个函数的时候，它就是全局可用的。如果我们选择了把它 export 给requirejs，那当我们的代码依赖于hello 模块的时候，就可以拿到这个 hello 函数的引用了。所以： exports 可以把某个非requirejs方式的代码中的某一个全局变量暴露出去，当作该模块以引用。
 
==暴露多个全局变量==
```
// hello.js
function hello() {
  alert("hello, world~");
}
function hello2() {
  alert("hello, world, again~");
}

// main.js
requirejs.config({
  baseUrl: '/public/js',
  paths: {
    hello: 'hello'
  },
  shim: {
    hello: {
      init: function() {
        return {
          hello: hello,
          hello2: hello2
        }
      }
    }
  }
});
requirejs(['hello'], function(hello) {
  hello.hello1();
  hello.hello2();
});
```

