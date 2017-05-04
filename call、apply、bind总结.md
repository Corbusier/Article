# call、apply、bind总结

call、apply、bind都可以动态的改变执行上下文,应用的场景也基本一样,在细节上有一些差别。在求最大最小值,排序、平均数、类数组转换数组、继承上都有体现。具体的用法如下：

## call( )
指定this值,第二个参数为参数列表,必须是注意传递进去使用,这也是和apply最大的区别。
在判断类型时,可以使用这样的方法:
```
    Object.prototype.toString.call([]);
```
借用Object上的toString方法来判断类型,说明call可以在一个对象上借用另一个对象上的方法。所以在类数组转换数组时也可以使用类似的方式
```
    [].slice.call(arguments);
```

### this值

 - 不传递,或为null、undefined,默认指向window
 - 传递另一个函数名,则this指向这个函数的引用,但是根据执行上下文的定义,也不一定就是真正的结果。
 - 传递原始值,this指向该原始值的包装对象。
 - 传递对象。this指向这个对象。

### argments1,arguments2...

逐一传递参数列表。

### 示例

以下示例来自其他网站：
```
    var animals = [
      {species: 'Lion', name: 'King'},
      {species: 'Whale', name: 'Fail'}
    ];

    for (var i = 0; i < animals.length; i++) {
      (function (i) { 
        this.print = function () { 
          console.log('#' + i  + ' ' + this.species + ': ' + this.name); 
        } 
        this.print();
      }).call(animals[i], i);
    }
```
分析以上的代码,在for循环的内部,有一个匿名函数,然后call传递this指向的是animals,改变了匿名函数本身的指向(window),并且call还会执行该匿名函数。在匿名函数的内部保留着对animals的引用,所以最后每个数组元素对象都可以得到一个print方法,返回它的索引值和内容。

## apply( )
与call方法基本相同,只是第二个参数必须是一个包含一个或多个参数的数组。apply和call都可以将类数组转化为数组,在IE9之前的版本中不接受类数组。可以采取这样的方式,兼容IE浏览器：
```
    function likeToArr(likeAry){
        let ary = [];
        try{
            ary = [].call(likeAry);
        }catch(e){
            for(let i = 0;i<likeAry.length;i++){
                ary[ary.length] = likeAry[i];
            }
        }
        return ary;
    }
```
### this值
同call方法

### argments
关于第二个参数,其实只要有length属性就可以,并非一定是数组。因为函数参数列表本身就是类数组,这也就导致不能直接使用Array.max/min求最大最小值,而借助apply可以达到目的。

### 比较apply和内置函数进行max、min求值
```
    var numbers = [5, 6, 2, 3, 7];

    /* 使用 Math.min/Math.max 在 apply 中应用 */
    var max = Math.max.apply(null, numbers);
    // 一般情况是用 Math.max(5, 6, ..) 或者 Math.max(numbers[0], ...) 来找最大值
    var min = Math.min.apply(null, numbers);
    
    //通常情况我们会这样来找到数字的最大或者最小值
    max = -Infinity, min = +Infinity;
    for (var i = 0; i < numbers.length; i++) {
      if (numbers[i] > max)
        max = numbers[i];
      if (numbers[i] < min) 
        min = numbers[i];
    }
```
### 获取body的所有Tag节点
```
    function bodyTree(){
        let root = document.body;
        function findChidren(el) 
            //el => body
            let children = el.children;
            let node = {
                curNode : el,
                children: []
            }
            if(children.length != 0){
                [].forEach.call(children,function(nodeItem,index){
                    node.children.push(findChidren(nodeItem));
                })
            } 
            return node;
        }
        return findChidren(root);
    }
    console.log(bodyTree());
```
除了用到了call之外,还有递归、闭包、作用域的知识点在里面。在核心的代码中是利用forEach函数遍历children的元素,递归调用该函数到children没有子元素为止,将取到的节点依次放到弄得对象中再返回。
 
## bind( )

 - bind是ES5中新增的方法,所以不兼容低版本IE。
 - 传参和call、apply类似,第二个参数可以使用类数组。
 - 不会直接执行函数,这是与前两种方法最大的区别。

## call和apply的模拟实现
### 1）.
在函数构造函数原型上添加自定义的方法,并且指定this对象,然后在原型上将指定的函数设为对象的属性,执行该函数后再删除。
```
    //第一步
    foo.fn = bar;
    //第二步
    foo.fn();
    //第三步
    delete foo.fn;
```
由此可以得到：
```
    Function.prototype.call2 = function(context){
        //此时this对象还是bar,执行bar时,
        context.fn = this;
        context.fn();
        delete context.fn;
    }
    var obj = {
        value : 1
    }
    function bar(){
        console.log(this.value);
    }
    bar.call2(obj);//1
```
在call2的内部将bar函数赋值给obj构造出来的一个属性,这时的再执行的过程相当于:
```
    obj.fn = bar = function(){console.log(this.value)};
    obj.fn = function(){console.log(this.value)};
    obj.fn();
    //console.log 1
    delete obj.fn;
```
## 2）.
如果bar传递的参数是不定长度的？

从arguments中取第一个到最后一个参数,因为第0个是arguments的活动对象的前端,此时即obj。
```
    Function.prototype.call2 = function(context){
        context.fn = this;
        var args = [];
        for(var i = 1, len = arguments.length; i < len; i++) {
            args.push('arguments[' + i + ']');
        }
        eval('context.fn(' + args +')');
        delete context.fn;
    }
```
在eval的过程中会自动的调用toString方法。
## 3）.
还有一个问题是如果第一个参数是null、undefined时,this指向的是window。
```
    Function.prototype.call2 = function (context) {
        var context = context || window;
        context.fn = this;

        var args = [];
        for(var i = 1, len = arguments.length; i < len; i++) {
            args.push('arguments[' + i + ']');
        }

        var result = eval('context.fn(' + args +')');

        delete context.fn
        return result;
    }
    var obj = {
        value : 1
    }
    var value = 2;
    function bar(name, age) {
        console.log(this.value);
        return {
            value: this.value,
            name: name,
            age: age
        }
    }
    console.log(bar.call2(null,'kevin',18));
```