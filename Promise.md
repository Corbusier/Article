# Promise

## Promise的使用场景

在实际的运用场景中,ajax请求得到返回值的时间不同,有了callback的回调结果之后才能知道接下来应该做什么。简单的ajax请求过程是：
```
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function(){
        if(xhr.readystate == 4){
            xhr.status = status;
            if(status >= 200 && status < 300 || satus == 304){
                response = xhr.responseText;
            }
        }
    }   
    xhr.open();
    xhr.send();
```
当readystate符合一定条件时才可以触发onreadystatechange事件,继续下一步的动作,处理返回的数据。而如果需要做另一个ajax请求时,这个新的ajax需要以上一个请求结果的某些参数,第三次再重复需要第二次的结果,然后这个过程循环下去...那么结果就会造成回调地狱(callback hell)。

除此之外,我们还需要使代码具有可读性和可维护性,因此必须将**数据请求**和**数据处理**明确的分开。一旦请求变得频繁,那么就无法在复杂的数据中维护代码了。
因为这样的需求,所以使用Promise变得尤为重要。

## Promise的建立实现

Promise是一个对象,它的三种状态分别对应的是：

 - pending: 等待中，或者进行中，表示还没有得到结果
 - resolved(Fulfilled): 已经完成，表示得到了我们想要的结果，可以继续往下执行
 - rejected: 也表示得到结果，但是由于结果并非我们所愿，因此拒绝执行
 
这三种状态不受外界的干扰,他们是各自独立的。而且状态只能从pending => resolved或rejected,并且这个过程不可逆。在Promise构造函数中,以一个函数作为参数,它可以监听Promise的变化。
```
    new Promise(function(resolved,rejected){
        if(true){resolve()};
        if(false){rejected()};
    })
```
### Promise中的then方法
then方法可以接收构造函数中的状态变化,并分别执行对应的操作,它有两个参数,第一个参数对应resolved状态的执行,第二个参数对应reject状态的执行。
```
    var fx = function(num){
        return new Promise(function(resolve,reject){
            typeof num === "number"?resolve():reject()
        }).then(function(){
            console.log("我是一个num值");
        },function(){
            console.log("我不是num值,我们不合适");
        })
    }
    fx('Corlis');
    fx(1019);
```
then会返回一个Promise对象,因此可以采用链式调用分别调用对应resolve和reject的函数,这样的方式就可以解决回调地狱。
    
```
    then(null,function(){}) <==> catch(function(){}) 
```

### Promise中的数据传递
利用Promise的知识封装一个原生的ajax请求

ajax相关函数获取请点击：

Click Me：[Article](/AJAX相关知识整理.md)
```
    var url = 'https://zhuanlan.zhihu.com/p/22890374';
    function getJSON(url){
        return new Promise(function(resolve,reject){
            var xhr = getXHR();//利用上一个文章中所写的函数
            xhr.onreadystatechange = function(){
                if(xhr.readystate == 4){
                    var status = xhr.status;
                    var response;
                    if(status >= 200 && status <300 || status == 304){
                        try{
                            response = JSON.parse(xhr.responseText);
                            resolve(response);
                        }catch(e){
                            reject(e);
                        }
                    }else{
                        reject(new Error(xhr.status))
                    }
                }
            }
        })
    }
    getJSON(url).then(function(){
        console.log("I resolved")
    },function(){
        console.log("I rejected")
    });
```

### Promise.all
如果一个ajax请求需要另外多个ajax请求都有返回结果后才能确定,那么Promise.all可以应用于这种情况。Promise.all接收一个Promise对象组成的数组作为参数,当这个数组所有的Promise对象的状态都变味resolved或rejectd时,才会执行then方法。
```
    var url = 'https://hq.tigerbrokers.com/fundamental/finance_calendar/getType/2017-02-26/2017-06-10';
    var url1 = 'https://hq.tigerbrokers.com/fundamental/finance_calendar/getType/2017-03-26/2017-06-10';
    
    function renderAll() {
        return Promise.all([getJSON(url), getJSON(url1)]);
    }
    renderAll().then(function(value) {
        console.log(value);
    })
```

### Promise.race
和Promise.all相对的是Promise.race,它也是接收一个Promise对象组成的数组为参数,不过只需要其中一个的Promise状态变为resolved或rejected时,就可以调用then方法。而传递给then的值也不同。
```
    function renderRace() {
        return Promise.race([getJSON(url), getJSON(url1)]);
    }
    
    renderRace().then(function(value) {
        console.log(value);
    })
```
### Promise.catch
catch方法返回的是一个对象，所以在它之后还可以继续添加其他的方法
```
    function getJSON(url).then(function(){
        ...
    }).catch(function(error){
        console.log("发生错误!",error);
    });
    //getJSON方法返回一个Promise对象,如果对象状态是Resolved,then方法调用回调函数,如果是Rejected,就会调用catch方法处理处理这个错误,如果then方法的回调函数在运行中抛出错误,也会被catch捕获.
```

如果Promise状态已经变为了resolved,再抛出错误是无效的
```
    var promise = new Promise(function(resolve,reject){
        resolve("ok")
        throw new Error("test");//无效！此时Promise状态已经变化
    })
    
    promise.
    then(function(value){
        console.log(value)}).
    catch(function(error){
        console.log(error);
    })
```

promise直接抛出一个错误，就被catch方法指定的回调函数捕获
```
    var promise = new Promise(function(resolve,reject){
        throw new Error("test");
    });
    promise.catch(function(error){
        console.log(error);
    })
    
```
这样的写法等价于

```
//way1:
    var promise = new Promise(function(resolve,reject){
        try{
            throw new Error("test");
        }catch(e){
            reject(e);
        }
    });
    promise.catch(function(error){
        console.log(error);
    })
//way2:
    var promise = new Promise(function(resolve,reject){
        reject(new Error("test"));
    });
    promise.catch(function(error){
        console.log(error);
    });
```

一般来说，不要在then方法里定义reject状态的回调函数，尽量使用catch方法，这样更接近于同步的(try/catch)方法。

**Promise对象抛出的错误不会传递到外层代码，catch无法捕获到这个错误。**

注意以下的代码,在Promise对象中再有一个Promise对象处理resolve/reject,而里面这个Promise对象抛出的错误无法传递到外层的Promise对象中,所以后面也无法catch这个错误。
```
    var someAsyncThing = new Promise(function(){
        return new Promise(function(resolve,reject) {
            //下面一行会报错，因为x没有声明
            resolve(x + 2);
        });
    });

    someAsyncThing.catch(function() {
        console.log('everything is great');
    });
```
总结就是,抛出错误要在状态改变之前,并且是由Promise对象直接抛出的错误才可以被之后的catch捕捉到。

## 模块系统中使用Promise
关于Require.js的快速上手参考之前撰写的：
    [Tool-Instructions](https://github.com/Corbusier/Tool-Instructions/blob/master/require.js/%E5%BF%AB%E9%80%9F%E4%BA%86%E8%A7%A3Require.JS.md "Require.JS")
    
### 使用Promise的ajax请求
文件目录如下：

<pre>
.ajax
├── index.html      
│   
├── libs(常用的库)                              
│   ├── API.js(请求的数据)               
│   ├── Jquery.js          
│   ├── request.js(请求的方法)         
│   ├── require.js 
│   
├── componnets(定义的模块)
│   ├── calendar.js  
│
├── pages
│   ├── index.js(入口文件)  
.
</pre>
请求的原理：
    通过加载不同的模块,在libs上的request发出请求获取API上的数据,请求的方法是Jquery中get方法的Promise,返回的也是一个Promise对象。将数据请求和数据处理(calender.js)放在不同的模块中,然后用统一的模块(request.js)管理所有的数据请求。
    
在html中要引入require.js
```
    // index.js为入口文件
    <script data-main="./pages/index.js" src="./libs/require.js"></script>
```
#### ①.将所有的url放在API模块下统一处理
```
    // libs/API.js
    define(function() {
        return {
            dayInfo: 'https://hq.tigerbrokers.com/fundamental/finance_calendar/get_day/2017-04-03',
            typeInfo: 'https://hq.tigerbrokers.com/fundamental/finance_calendar/getType/2017-03-26/2017-04-15'
        }
    })
```
#### ②.将所有的数据放在request模块中统一管理
```
    define(function(require){
    
        var API = require('API');
        var $ = require('jquery');
        /*因为jQuery中的get方法也是通过Promise进行了封装,最终返回的是一个Promise对象,
        因此可以将数据请求与数据处理放在不同的模块,使用一个统一的模块管理所有的数据请求
        需要的是API中提供的url下的数据,以及ajax的get方法。
        所以要引入API和jquery,即require('API')和require('jquery');
        */
        //获取当天的信息
        getDayInfo = function() {
            return $.get(API.dayInfo);
        }
        // 获取type信息
        getTypeInfo = function() {
            return $.get(API.typeInfo);
        };
        return {
            getDayInfo: getDayInfo,
            getTypeInfo: getTypeInfo
        }
    });
```
#### ③.将请求的结果获取并处理
```
    //components/calendar.js
    define(function(require) {
        var request = require('request');
    
        //拿到数据之后,需要处理的组件,可以根据数据render
        //为了简化,本例只是输出数据,在实际中,拿到数据之后还要进行相应的处理
    
        //request.js设置的请求函数,其中包含可以直接使用的Promise对象
        request.getTypeInfo()
            .then(function(resp) {
                //拿到数据,并执行处理操作
                console.log(resp);
            })
        //将数据请求与数据处理分开,有利于可维护性
    })
```
关于以上文件的获取请点击：
Click Me : [Tool-Instructions](https://github.com/Corbusier/Tool-Instructions/tree/master/require.js/ajax(promise))

### 图片加载
在实际的运用中,经常需要将一些图片放在块元素中,而如果把宽高限定好,这样会影像图片的美观,有些图片可能会严重的"比例失调"。
利用require可以写一个image组件来解决这个问题,使图片能够根据自己的宽高逼,合理的缩放。

如果保持这样的修正,那么图片就不会出现失调：
    
    高>>宽时,令宽=100%,高自动补位然后overflow:hidden
    宽>>高时,令高=100%,宽自动补位然后overflow:hidden
    
在样式中我们可以针对这两种情况,定义两个class选择器：
```
    .img-center img.aspectFill-x {
        width: 100%;
        top: 50%;
        transform: translateY(-50%);
    }
    
    .img-center img.aspectFill-y {
        height: 100%;
        left: 50%;
        transform: translateX(-50%);
    }
```
获取图片的原始宽高,需要等到图片加载完毕之后才能获取。而当图片已经存在缓存时,则有一个complete属性变成true。那么我们就可以根据这些基础知识,定义一个模块来处理这件事情。

(complete应该少用,当src无效时,对于complete值变化,不同的浏览器表现不一样,并且以complete为true来判断是否触发onload事件并不科学,complete为true的情况有4种,completely available只是其中一种)
    
    不考虑兼容性和特殊情况的前提下,本例只作示例使用。

```
    //components/imageCenter.js
    define(function(require) {
    //利用Promise封装一个加载函数,也可以单独放在一个功能模块中进一步优化
    //complete属性在各个浏览器下表现不同,不建议使用
    var imageLoad = function(img) {
        return new Promise(function(resolve, reject) {         
            if(img.complete) {
                resolve();
            }else{
                img.onload = function(event) {
                    resolve(event);
                }
                img.onerror = function(err) {
                    reject(err);
                }
            }
        })
    }
    //模块name(外部容器,图片的class模式)
    var imageCenter = function(domList,mode) {
        domList.forEach(function(item) {
            /*
                获取容器下的img容器尺寸200*150,图片实际尺寸比例itemR(设为X) = 4/3
                图片原始尺寸比例imgR(设为N)
                假设实际尺寸两项都小于容器,
                  1)如果高>>宽,应该让宽=100%,高自动补位然后hidden
                  2)如果宽>>高,应该让高=100%,宽自动补位然后hidden
                然后给img分别加上对应的class
              */
            var img = item.children[0];
            var itemW = item.offsetWidth;
            var itemH = item.offsetHeight;
            var itemR = itemW / itemH;
            imageLoad(img).then(function() {
                var imgW = img.naturalWidth;
                var imgH = img.naturalHeight;
                var imgR = imgW / imgH;
                var resultMode = null;
                switch(mode){
                    case 'aspectFill':
                        resultMode = imgR > 1 ? 'aspectFill-x' : 'aspectFill-y';
                        break;
                    case 'wspectFill':
                        resultMode = itemR > imgR ? 'aspectFill-x' : 'aspectFill-y'
                        break;
                    default:
                }
                $(img).addClass(resultMode);
            })
        })
    }
    return imageCenter;
})
```
在使用时index.js直接引入并调用即可
```
    define(function(require) {
        /*
            其他的模块中可以引入jquery
            需要在index.js下调用imageCenter函数
            没有进行配置的button模块组件,也可以这样以的形式引入
            包裹图片的容器
            传入image的warp标签list，将其中的iamge标签设置为居中
        */
        var $ = require('jquery');
        var imageCenter = require('imageCenter');
        var imageWrapList = document.querySelectorAll('.img-center');
        imageCenter(imageWrapList, 'wspectFill');
    })    
```
本例的代码：

Click Me ： [Tool-Instructions](https://github.com/Corbusier/Tool-Instructions/tree/master/require.js/imgTest)

