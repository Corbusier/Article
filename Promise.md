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
总结起来就是,抛出错误要在状态改变之前,并且是由Promise对象直接抛出的错误才可以被之后的catch捕捉到。

## 模块系统中使用Promise
关于Require.js的快速上手参考之前撰写的：
    [Tool-Instructions](/require.js/userExample01/index.html)
    
