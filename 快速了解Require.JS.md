# 快速了解Require.JS

R.J的主要功能是按不同的先后依赖关系,异步加载JS文件,适合在浏览器中使用,可以确保依赖的JS文件加载完成之后再加载当前的文件,核心的思想是AMD。只做一件事：模块化。而模块化的作用就是避免全局变量污染,命名冲突的问题,它可以作为一种命名空间使用。

RequireJS主要的功能：
1)引入需要使用的命名空间(顺便加个别名也行)；
2)将自己的代码放到命名空间中，避免全局污染。

RequireJS 定义了(define)一个命名空间，在定义的时候，顺便引用了需要使用其他命名空间。按照 RequireJS 的术语，它把命名空间叫做“模块”。注意，在这里，RequireJS 定义的模块(命名空间)是匿名的，没有取名，这是和c#不同的地方。

## 定义模块的方式
假如新建另外一个awesome.test2.js文件来使用awesome.test1.js：
```
    define(['awesome.test1'],function(t1){
        return{
            NewTest : function(){
                t1.MyTest();
            }
        }
    })
```
通过不加后缀的文件名,引入了awesome.test1的命名空间,并给它取了别名 t1,然后在代码中使用 t1.MyTest() 方法。按照AMD规范,所有的模块定义都必须放在都必须放在 return {} 对象中。你的代码都要写在 return 返回的 {} 对象里面。这样做显得臃肿难看？你可以重构一下，比如这样做：
```
    define(['awesome.test1'],function(t1){
        function someFunc1() {
            // Your Code...
        }
        function someFunc2() {
            // Your Code...
        }
        //通过 return 方式，将需要公开的函数暴露出来，供其他 js 调用
        //这一点正是契合了在这个模块中使用单例的公共接口,这样就可以访问私有变量和函数
        return{
            NewTest : function(){
                t1.MyTest();
            },
            fun1 : someFunc1,
            fun2 : someFunc2
        }
    })
```
define函数的三个参数分别为：

 1. 第一个参数是依赖的模块,可以是在path定义的公用模块,也可以是自定义的代码,不过自定义的代码需要注明引用路径,例如../js/util.js。
 2. 第2个参数是自定义模块，传入的是一个匿名函数，函数参数顺序和模块顺序一一对应，最后return的对象或函数即为对外暴露的接口。
 3. 第3个是模块加载报错的处理函数（一般可以省略）。

这里的模块加载都是异步的,这一点和CMD(sea.js)不同。

当然,如果你想使用这些功能,必须要在页面中引入 require.js 文件：

## 配置文件
使用require.js的方式是：
```
    <script data-main="main.js" src="js/require.js"></script>
```

main方法作为入口文件，上面这种写法，做了几件事情：

 1. 加载了 require-jquery.js 文件。注意,官方提供了 RequireJS 和 jquery 的打包版本,推荐。
 2. 在加载之后，加载入口文件 js/main.js,注意，main.js 写进去的时候,不需要后缀名。

所有其他 js 模块文件，都可以写在 main.js 里面，通过 main.js 加载。

require.js的入口文件main.js的配置是这样的
```
    requirejs.config({
        baseUrl: "frame",//模块查找的根路径
        paths: {
            "zepto": "js/zepto",
            "frozen": "js/frozen",
            "underscore": "js/underscore",
            "text": "js/text",
            "css": "js/css",
            "route": "../js/route"
        },
        shim: {
            "zepto": {
                exports: "$"
            },
            "underscore": {
                exports: "_"
            },
            "frozen": {
                deps: ["zepto"]
            }
        }
    });
``` 
paths是预定义的一些模块,比如js/zepto定义为zepto模块。

shim处理了导出名称和依赖关系。第1个zepto模块的全局变量名称为$（这个就好比 jquery.js我们叫做jquery，实际用的时候我们用$作为变量），第3个frozen模块表示当它被加载的时候，需要先加载zepto模块。

## 加载依赖的js方式
目前为止遇到了两个关键字,一个是define,它可以用来定义模块(命名空间),还有一个是require,可以直接加载其他的js。
```
    <script>
        require( ["some" ] ); 
    </script>
```
还有一种类似define的复杂用法：
```
    <script>
        require(["aModule","bModule"],function(){ 
            myFunctionA(); //使用 aModule.js中的函数myFunctionA
            myFunctionB(); //使用 bModule.js中的函数myFunctionB
        }
    </script>
```
总结一下,define是定义自己的模块的时候使用,可以顺便加载其他js；require 直截了当,供你加载用的,它就是一个加载方法,加载的时候,可以定义别名。

## 实际例子

这个例子的设计要求是 workjs01.js文件依赖 jQuery,workjs02.js 依赖 workjs01.js,只有等依赖文件加载完了,最后在页面打出提示信息。

Click Me：

[Tool-Instructions](/require.js/userExample01/index.html)

## 使用r.js打包
如果模块数很多,那么请求数就会大大增加,而require.js针对这个情况提出了解决方案,就是r.js。首先要下载r.js,然后把它放在项目的根目录下使用。还是以上一个实际例子为例
<pre>
.userExample01
├── index.html      
├── main.js(主文件)  
├── main-built.js
├── r.js          
├── built.js             
├── static
   ├── scripts
       ├── src
            ├── lib
                ├── jquery
                  ├── jquery
                ├── require
                  ├── require
            ├── work
                ├── workjs01
                ├── workjs02
.
</pre>
可以看到在根目录下多了r.js、built.js、main-built.js和built.js。r.js是用来压缩代码的,而main-built.js是压缩后的模块。具体使用方式：
创建built.js,然后配置built文件,和main.js差别不大,只是多了name属性和out属性,out是在该目录下压缩后的输出文件,name指向的是require.js的入口文件即main.js,可以这样理解,从模块文件lib、work这一级需要返回三级才能到main.js所在的目录,所以：
```
    ,name: "../../../main"
    ,out:"main-built.js"
```
然后cmd进入该目录执行：
```
    node r.js -o built.js
```
就可以得到main-built了,在index.html中修改原本指向的main.js为man-built.js。打开控制台的network可以看到请求数比之前减少了,这在大型项目中使用可以大大减少请求。

## 技巧
### CDN
如果要使用cdn上的代码,可以在配置文件上的paths属性中：
```
    "zepto": ['http://apps.bdimg.com/libs/zepto/1.1.4/zepto.min',"js/zepto"]
```
第一个是CDN,加载失败时从本地获取
### package
模块再次组合封装
### 插件
配上text插件即可把html文件当做模块引入。用处就是可以当做碎片页来处理,类似jquery中的load函数,不过好处就是这个html文件可以被缓存,而不是像jquery每load一次就发起一次ajax请求。
