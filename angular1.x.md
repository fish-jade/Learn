# 概念
- template 模版
- directive 指令
- model 模型(数据)：用户在视图中显示的数据、并与用户进行交互
- scope 作用域
- expressions 表达式
- compiler 编译器：解析模版、实例化指令和表达式
- filter 过滤器
- view 视图
- Data Binding 数据绑定
- controller 控制器
- Dependency Injection 创建并连接对象和函数
- Injector 注入器：依赖注入的容器
- module 模块：一个用于应用程序不同部分的容器，包括控制器、服务、过滤器、配置injector的指令
- service 服务：可重用的业务逻辑

## 模块(module)
模型 视图 控制器 过滤器 服务 
```
var app = angular.module("myApp",[])
// 注册一个模块
var app = angular.module("myApp")
// 获取一个模块，若不存myApp模块则报错
```
## 控制器
```

```
## 指令
==注意事项==  
在定一个这样的指令
```
myModule.directive('maskLayer', function() {
    return {
        restrict: 'EA'
        scope:{
            isShowBox: '='
        }
    }
}
```
采用驼峰的写法时，是使用时，需要使用 - 进行连接  
eg：<mask-Layer is-show-box="true"></mask-layer>

ng-bind ng-show ng-hide ng-class ng-model
- 模版指令
- 属性指令
- 结构指令
### 指令执行阶段
![](http://ww1.sinaimg.cn/large/9da83df8gy1flqw02sxttj20iw0bt443.jpg)
- 自定义compile
```
var myModule = angular.module("MyModule", []);
myModule.directive('repeater', function() {
    return {
        restrict: 'A',
        compile: function(el, attrs, transclude) {
            //这里开始对标签元素自身进行一些变换
            console.log("repeat...compile...");
            
            var tpl = el.children().clone();            
            for (var i = 0; i < attrs.repeater - 1; i++) {
                el.append(tpl.clone());
            }

            //返回link函数
            return function(scope, el, attrs, controller) {
                console.log("repeat...link...");
            }
        }
    }
});
```
==注意：一般不会同时出现compile与link，因为当出现compile时，需要返回一个函数作为link函数来直接调用，若再自己写一个link，是不会执行写了link方法==
- 自定义link 操作dom 绑定事件
```
var myModule = angular.module('MyModule',[]);
myModule.directive('hi',fucntion(){
    return {
        restrict: 'AE',
        template: "<div></div>",
        replace:true,
        link:function(scope,el,attrs,controller){
        // 数据,dom节点,dom上的属性,所需引入的指令controller
            
        }
    }
})
```
- compile与link的区别
![](http://ww1.sinaimg.cn/large/9da83df8gy1fls3n7bgqkj20kp0afgrg.jpg)

### 自定义指令
![](http://ww1.sinaimg.cn/large/9da83df8gy1flquzucm9kj20kz0atjvj.jpg)
```
var myModule = angular.module("MyModule", []);
// .run方法：当注射器加载完所有模块时，run方法会被执行一次
myModule.run(function($templateCach){
    $templateCache.put("index.html","<div>hello</div>")
    // 将模版缓存起来，供多个指令使用
})
myModule.directive("hello",function(){
    return{
        // restrict 指令匹配模式
        restrict: 'A' ,// 可选值：A属性 E元素 M注释 C样式
        template: "<div></div>",// 模版
        //或 建议使用templateUrl:"index.html"
        //或 template: $templateCache.get("index.html") 
        // 使用get方法将缓存模版取出来
        replace: true // 将注册使用的元素类型指令，内部的内容忽略
        // 如果需要将元素类型指令，内部的内容显示出来则使用transclude(类似vue的类容分发)
    }
})
```
transclude例子
```
var myModule = angular.module("MyModule", []);
myModule.directive("hello",function(){
    return{
        // restrict 指令匹配模式
        restrict: 'AE' ,// 可选值：A属性 E元素 M注释 C样式
        transclude:true,
        // transclude 会提示angular将双标签内部的内容放到<div ng-transclude></div>中去
        template: "<div>hello<div ng-transclude></div></div>"// 模版
        // 如果需要将元素类型指令，内部的内容显示出来则使用transclude(类似vue的类容分发)
    }
})
HTML
<hello><div>你好！</div></hello>
经angular编译后
<div>hello
    <div>你好！</div>
</div>
```
### 指令与控制器|指令与指令之间的交互方式
- 指令与控制器之间的交互  
需求：  
同一个指令，在不同的控制器下，调用不同控制器特有的方法  
```
HTML
<div ng-controller="crt1">
<loader howToLoad="load1()">滑动加载</loader>
</div>
<div ng-controller="crt2">
<loader howToLoad="load2()">滑动加载2</loader>
</div>

JS
var myModule = angular.module("MyModule", []);
myModule.controller('ctr1',['$scope',function($scope){
    $scope.load1=function(){
        console.log('111111')
    }
}])
myModule.controller('ctr2',['$scope',function($scope){
    $scope.load2=function(){
        console.log('2222222')
    }
}])
myModule.directive('loader',function(){
    return{
        restrict:"AE",
        link:function(scope,element,attr){
            element.bind("mouseenter",function()){
                scope.$apply(attr.howtoload)
                // 如果在元素指令中的属性写的是驼峰命名，使用attr时需要用小写
            }
        }
    }
})
```
- 指令与指令之间的交互
```
<superman strength></superman>
<superman strength speed></superman>
<superman strength speed ligth></superman>

JS

var myModule = angular.module("MyModule",[]);
myModule.directive("superman",function(){
    return{
        scope:{},// 创建独立scope作用域
        restrict:"AE",
        controller: function($scope){
        // 内部controller，作用：给本指令暴露一组public方法供外部调用
            $scope.abilities=[];
            this.addStrength = function(){
                $scope.abilits.push("strength")
            };
            this.addSpeed = function(){
                $scope.abilits.push("speed")
            };
            this.addLight = function(){
                $scope.abilits.push("light")
            };
        },
        link: function (scope,element,attrs){
            elements.addClass('btn btn-primary');
            elements.bind("mouseent",function(){
                console.log(scope.abilities)
            })
        }
    }
})
myModule.directive("strength",function(){
    return{
        require:'^superman',// 表示strength指令依赖于superman指令
        link: function(scope,element,attrs,supermanCtrl){
        // 第四个参数 supermanCtrl为superman指令中的controller
            supermanCtrl.addStrength()
        }
    }
})
myModule.directive("speed",function(){
    return{
        require:'^superman',// 表示speed指令依赖于superman指令
        link: function(scope,element,attrs,supermanCtrl){
        // 第四个参数 supermanCtrl为superman指令中的controller
            supermanCtrl.addSpeed()
        }
    }
})
myModule.directive("light",function(){
    return{
        require:'^superman',// 表示ligth指令依赖于superman指令
        link: function(scope,element,attrs,supermanCtrl){
        // 第四个参数 supermanCtrl为superman指令中的controller
            supermanCtrl.addLigth()
        }
    }
})
```

## 服务
将公共部分抽离成服务，供控制器调用  
- 使用$http服务
![](http://ww1.sinaimg.cn/large/9da83df8gy1flqzgse43mj20i50670w3.jpg)
- 创建自己的service
```
var myService = angular.module("MyService",[]);
// 使用.factory方法定义一个服务
myService.factory("userListService",['$http',function($http){
    
}])
```
==注意==  
在进行注入时，需要将自己写的服务，写在最后面。
```
myService.controller('serviceController',['$scope','$timeout','userListService',function($scope,$timeout,userListService){
    
}])
```
- service的特性  
![](http://ww1.sinaimg.cn/large/9da83df8gy1flr0aidu0vj20g009l40q.jpg)
- service、Fctory、Provider本质都是Provider
==provider其实是一种设计模式(策略模式与抽象工厂模式的混合体)==
- 其他的内置service
    - $compile(编译服务)
    - $filter(过滤器)
    - $interval(定时器)
    - $timeout(定时器)
    - $local(国际化)
    - $location(监控浏览器地址变化)
    - $log(日志)
    - $parse
    - $http(封装好的ajax)
## 过滤器(管道)
使用$filter服务(过滤器)
- 内置过滤器
- 自定义过滤器
```
var myModule = angular.module("Mymodule",[]);
myModule.filter('过滤器名',function(){
    return function(item){
        return item + "你好！"
    }
})
```
## $scope
- 树形结构(与DOM标签平行)
- 子$scope会继承父$scope
- $scpoe是表达式的执行环境(作用域)
- 事件
    - 向上传播  
    $emit
    - 向下传播  
    $broadcast
- 提供的工具方法
    - $watch()
    - $apply()
- 根$scpoe($rootscope,只有一个，位于ng-app上)
- 独立$scope(可不在$rootscope上创建$scope)  
```
类似于vue中的组件采用
data () {
    return{
        
    }
}
```
其作用是为了不让复用的指令(组件)使用同一个scope(变量)
- [scope绑定](https://www.imooc.com/video/3085)  
对指令scope与controller scope进行绑定
![](http://ww1.sinaimg.cn/large/9da83df8gy1flqymvkufuj20ly0abn0d.jpg)
- 通过angular.element($0).scope() 进行调试
- 生命周期
    - 创建
    - 注册
    - 模型变化
    - 模型脏检查
    - 销毁
## 路由
![](http://ww1.sinaimg.cn/large/9da83df8gy1flqqxejpgsj20d80730wl.jpg)
原生路由无法进行路由嵌套  
需要使用ui-route

# 核心原理
## $parse、$rootscope(两个重要的provider)
## 双向数据绑定
## 依赖注入
![](http://ww1.sinaimg.cn/large/9da83df8gy1fls20qr0kcj20js0an40v.jpg)
### 注入方式
- 内联式的注入
```
var myModule = angular.module("MyModule",[]);
myModule.controller("mycontroller",["$scope",function(scope1){
    scope1.text="你好！";
}])
```
- 推断注入
```
var myModule = angular.module("MyModule",[]);
myModule.controller("mycontroller",function($scope){
    
})
// 缺点：所使用的参数必须与所需要注入的对象相同
```
- 声明式注入
```
var myModule = angular.module("MyModule",[]);
myModule.$inject = ["$scope"]
myModule.controller("mycontroller",function(myName){
    myNmae.text = '你好！';
})

<div ng-app="MyModule" ng-contorller="mycontroller">
{{text}}
</div>
```
### provider 与 injector
- 直接使用$injector
$injector 是一个angular的内置对象  
其包含多个方法
    - invoke 方法 (用来调用一个服务/方法)
    ```
    // 当你定一个了一个服务，可以通过$injector.invoke方法进行实例化调用
    myModule.factory("game",function(){
        return {
            gameName:"魂斗罗"
        }
    });
    myModule.controller("myc",["$injector",function($injector){
        $injector.invoke(function(game){
        // 此处 angular 会将服务game的实力传递过来
            console.log(game.gameName)
            // 魂斗罗
        })
    }])
    ```
    - annotate 方法 (用来分析函数的参数)
    此方法是用来分析函数的参数  
    在推断注入时，就会使用到此方法
    ```
    $injector.annotate(function(arg1,arg2,arg3){})
    // ["arg1","arg2","arg3"]
    ```
    - get (获取一个服务)
    - has
    - instantiate (用来实例化一个对象)
- provider
创建一个服务的方法有很多，包括：constant、value、service、factory、provider、decorator  
==但是这些方法在源码中最终都是调用provider== 
==**service注入时就会被实例化，而factory注入时不会被实例化，需要使用new进行实例化使用**==  
    
    - provider
    
    ```
    var myModule = angular.module("MyModule",[]);
    myModule.provider("Hello",function(){
        return {
            $get:function(){
                
            }
        }
    })
    // 注意：使用provider必须定义$get方法，并且返回
    // 因为在源码中，有对该方法的返回进行检查，若无$get方法，则会报错
    ```
    - factory
    ```
    myModule.factory("hello",function(){
        return{
            
        }
    })
    // 对于factory没有特殊要求，但一定要返回一个对象
    // 被注入的factory需要实例化才可以使用
    ```
    - service
    ```
    myModule.service("hello",function(){
        
    })
    // 可以不返回
    // service 在注入时就会被实例化
    ```
    - constant和value  
    ```
    var app = angular.module('app', []);
 
    app.config(function ($provide) {
      $provide.value('movieTitle', 'The Matrix')
    });
     
    app.controller('ctrl', function (movieTitle) {
      expect(movieTitle).toEqual('The Matrix');
    })
    或使用语法糖
    app.value('hello','你好')
    app.constant('hi','你好')
    
    // 两者唯一区别是常量与变量的区别
    ```
    
## angular的启动过程
- 启动方式
    - 自动启动
    自动启动：当html中有ng-app 这个指令时，angular会检测到该指令，并进行自动启动
    - 手动启动
    当在html中无ng-app
    ```
    angular.element(document).ready(function(){
        angular.bootstrap(document,["MyModule"])
        // bootstrap 是一个angular实现的启动函数，若需要手动启动，则调用此方法
    })
    ```
    在angular中，一个页面是不可以出现ng-app指令嵌套的。可以平行。
    ```
    <div ng-app="Myapp1"></div>
    <div ng-app="Myapp2"></div>
    这样写，angular是不会报错的。但是第二个ng-app是不会自动启动，需要自己使用bootstrap方法手动启动
    // 但是建议存在多个ng-app
    ```
- 绑定Jquery
angular 会检测是否有导入Jquery  
若没有导入，则使用angular内部实现的简版Jquery——jqLite  
若有导入，则使用外部导入的Jquery
- 全局对象 angular  
angular会将自身实现的内部api全部发布到angular全局对象上
- bootstrap(不是css框架bootstrap，而是angular中自己定义的启动方法)
- 

## 路由
angular配置路由  
使用.config API进行路由配置
- 嵌套路由
```
由于内置路由使用不方便，所以使用第三方路由 angular-ui-router
var app = angualr.module("myAPP",['ui.router'])
// 需要将angular-ui-router注入到angular , 打开angular-ui-router可以看到 module.exports = "ui.router"
app.config(["$stateProvider",function($stateProvider){
    $stateProvider
        // state 参数说明: “路由名”,{路由对象}
        .state("parent",{ 
            url: '/parent',
            template:"<div>parent <div ui-view></div></div>"
        })
        // 定义嵌套路由使用 "." 连接父子路由名; 其使用方式与vue的嵌套路由相同
        .state("parent.child",{
            url:"/child",
            template:"<div>child</div>"
        })
}])
```
- views实现多视图
```
<div ng-app="myApp">
    <a ui-sref="index">点我显示index内容</a>
    <div ui-view="header"></div>
    <div ui-view="nav"></div>
    <div ui-view="body"></div>
    // 此处可以将多个ui-view 同时展出
</div>

var app = angular.module("myApp",['ui.router']);
app.config(["$stateProvider",function($stateProvider){
    $stateProvider
     .state("index",{
         url:'/index',
         views:{
             'header':{template:""},
             'nav':{template:""},
             'body':{template:""}
         }
     })
}])
```
- ui-view 的 定位
@ 的作用是用来绝对定位view，即说明ui-view属于那个模版；eg：‘header@index’表示名为header的view属于index模版
```
<body >    
    <div ng-app="myApp" >
        <a ui-sref="index">show index</a>
        <a ui-sref="index.content1">content111111</a>
        <a ui-sref="index.content2">content222222</a>
        <div ui-view="index"><div>
    </div>  
  </body>

  <script type="text/javascript">
    var app = angular.module('myApp', ['ui.router']);   
    app.config(["$stateProvider",  function ($stateProvider) {      
        $stateProvider     
        .state("index", {
            url: '/index',  
            views:{
            'index':{template:"<div><div ui-view='header'></div>  <div ui-view='nav'></div> <div ui-view='body'></div>  </div>"},
                //这里必须要绝对定位
                'header@index':{template:"<div>头部内容header</div>"},    'nav@index':{template:"<div>菜单内容nav</div>"}, 'body@index':{template:"<div>展示内容contents</div>"}
            }
        })    
        //绝对定位
        // 使用绝对定位会声明该view是属于哪个模版，在其他模版是不能使用的
        .state("index.content1", {
            url: '/content1',  
            views:{
                'body@index':{template:"<div>content11111111111111111</div>"}
                //'body@index'表时名为body的view使用index模板
            }
        })  
        //相对定位：该状态的里的名为body的ui-view为相对路径下的（即没有说明具体是哪个模板下的，所以在任何地方都能使用）
        .state("index.content2", {
            url: '/content2',  
            views:{ 'body':{template:"<div>content2222222222222222222</div>"}//
            }
        })      
    }]);

  </script>
```
- URL 路由传参(通过$stateParams获取参数)
有了$stateParams可以获取参数，这样就能做动态路由匹配==从下面例子可以看出$stateParams是不需要注入就可以使用，而且controller的触发发生在路由跳转前(即相当于beforeRouteEnter)==
```
所支持的传参形式 url:'/index/:id'和url:'/index/{id}'
 <body >    
    <div ng-app="myApp" >
        <a ui-sref="index({id:30})">show index</a>    
        <a ui-sref="test({username:'peter'})">show test</a>
        <div ui-view></div>
    </div>  
  </body>

  <script type="text/javascript">
    var app = angular.module('myApp', ['ui.router']);   
    app.config(["$stateProvider",  function ($stateProvider) {      
        $stateProvider     
        .state("home", {
            url: '/',  
            template:"<div>homePage</div>"

        })
        .state("index", {
            url: '/index/:id',  
            template:"<div>indexcontent</div>",
            controller:function($stateParams){
                alert($stateParams.id)
            }
        })  
        .state("test", {
            url: '/test/:username',  
            template:"<div>testContent</div>",
            controller:function($stateParams){
                alert($stateParams.username)
            }
        })          

    }]);

  </script>
```
- Resolve(预载入)
像vue中的beforeRouteEnter，由于beforeRouteEnter触发时，组件未被渲染，无法取到组件中的数据。Resolve的作用类似处理这种情况，为了能够给controller提供数据调用
```
 <body >    
    <div ng-app="myApp" >
        <a ui-sref="index">show index</a>    
        <div ui-view></div>
    </div>  
  </body>

  <script type="text/javascript">
    var app = angular.module('myApp', ['ui.router']);   
    app.config(["$stateProvider",  function ($stateProvider) {      
        $stateProvider     
        .state("home", {
            url: '/',  
            template:"<div>homePage</div>"

        })
        .state("index", {
            url: '/index/{id}',  
            template:"<div>indexcontent</div>",
            resolve: {
                //这个函数的值会被直接返回，因为它不是数据保证
                user: function() {
                  return {
                    name: "peter",
                    email: "audiogroup@qq.com"
                  }
                },
                //这个函数为数据保证, 因此它将在控制器被实例化之前载入。
                detail: function($http) {
                  return $http({
                    method: 'JSONP',
                    url: '/current_details'
                  });
                },
                //前一个数据保证也可作为依赖注入到其他数据保证中！（这个非常实用）
                myId: function($http, detail) {
                  $http({
                    method: 'GET',
                    url: 'http://facebook.com/api/current_user',
                    params: {
                      email: currentDetails.data.emails[0]
                    }
                  })
                }

            },
            controller:function(user,detail,myId$scope){
                alert(user.name)
                alert(user.email)
                console.log(detail)
            }
        })                  

    }]);

  </script>
```

### 路由设置对象参数

```
$routeProvider.when(url,{
    template:string, //在ng-view中插入简单的html内容
    templateUrl:string, //在ng-view中插入html模版文件
    controller:string,function / array, //在当前模版上执行的controller函数
    controllerAs:string, //为controller指定别名
    redirectTo:string,function, //重定向的地址
    resolve:object<key,function> //指定当前controller所依赖的其他模块
});
```

## 问题
```
var app = angular.module("myApp",[]) // .module("模块名",[依赖项])
app.config
app.run
```

## 自带指令
- ng-class
```
ng-class命令 用于控制视图样式
eg:
<div ng-class="active"></div>
ng-class接受三种形式的参数
- string
- object
- array
<div ng-class="active"></div> // string
<div ng-class="{isactive:'active'}"></div> // object 当isactive为true时active样式生效
<div ng-class="['active','red',{isbig:'big'}]"></div> // array

另外一种obj与arr结合的方式
<div ng-class="{true:'active',false:''}[num == 1]"></div>
当方括号中的表达式num==1为true时，对象中的active生效
```
## 使用开发注意事项
- 多个控制器公用的变量 存在 rootscope上
- 

## 项目使用angularjs经验
- 如何在请求时集中添加 请求加载动画 及 处理请求异常错误？ 
```
可以使用 angular中的拦截器 集中进行处理

eg ： http拦截器

// 拦截器工厂
app.factory("httpInterceptor", [ "$q", "$rootScope", function($q, $rootScope) {
    return {
        request: function(config) {
        // do something on request success
        // return config || $q.when(config);
        console.log('request',config)
        },
        　　 requestError: function(rejection) {
        　　　　 // do something on request error
        // 　　　　 return $q.reject(rejection)
        console.log('requestErr',rejection)
        　　 },
        response: function(response) {
        // do something on response success
        // return response || $q.when(response);
        console.log('response',response)
        },
        responseError : function(rejection) {
            // do something on response error
            // return $q.reject(rejection);
            console.log('rejection',rejection)
        }
    };
}]); 
// 初始化注入
app.config(["$httpProvider", function($httpProvider) {
　　$httpProvider.interceptors.push("httpInterceptor");
}]); 
```
在自己写的过程中遇到的问题
>  - **注入到$httpProvider.interceptors中的服务必须是实例化的**  
> 也就是说可以使用在注入时能够自动被实例化的service定义拦截器服务  
> 也可以使用factory定义，但是需要返回new之后的实例

==拦截器服务解析==  
所需要返回的对象
1. request
2. requestError
3. response
4. responseError

所返回的四个属性值均未处理函数
![](http://ww1.sinaimg.cn/large/9da83df8gy1fmwiem1za5j20w30oxae4.jpg)

- 如何封装基础服务类
```

```
- ==factory与service重理解==
```
在利丰项目中，一个基本的service结构

function loginService($rootScope,$state,...){// 注入项
    function login(){
        // 业务逻辑
    }
    return login;
}

loginService.$inject = ['$rootScope','$state',...] // $inject注入接受项  （要与$injector分开，$injector为方法）
export default angular.module('loginServcie',[]).service('loginService',loginService).name

```
**结构分析**  
```
loginService 为所注册的模块 服务 及 回调
其内层中 login 为 构造函数，供控制器中注入服务后，实例化使用。

```
==angular在注入服务后，就已经实例化，并且生命周期一直存在==  
所以在控制器中的使用是
```
'use strict';
function loginController($rootScope,$scope,$state,loginService){
	$scope.user = new loginService();
}
loginController.$inject =['$rootScope','$scope','$state','loginService'];
export default angular.module('loginController',[]).controller('loginController',loginController).name;

在注入的服务loginService，使用new实例化，其实是实例化service中的构造函数 login
```

- 

## angularjs与vue对比
由于尤雨溪大神原本是参考angularjs  
所以angularjs与vue有可对比处

- 双向绑定
```
angularjs
脏检查

vue2.x
defineProperty
```
- 控制器
```
angular
controller

vue
没有控制器
```
- 服务(factory service provider)
```
angular
服务 生命周期：被首次注入自动实例化后一直存在

vue
没有服务
但感觉服务有点类似于状态管理
生命周期：一直存在
```
- 指令
- 路由