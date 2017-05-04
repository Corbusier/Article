
## 数据类型
数据类型分为两种,基本数据类型和引用数据类型。ES6中的数据类型不做讨论，基本类型包括了：string、null、undefined、number、boolean。引用类型：object。它是由一个或者多个键名键值对的对象。基本数据类型保存在栈内存中,而引用类型保存在堆内存中,栈内存中的数据必须是固定大小的,而引用类型的大小不固定,所以只能白存在堆内存中,将堆内存中的地址直接赋值出来之后就可以访问引用类型数据。他们之间的差别如下：

|  区别   | 基本数据类型   |  引用数据类型  |
| -----   |  --:  | :--:  |
|   保存位置  |  栈内存     |   堆内存     |
|   数据大小  |   固定大小   |  不固定   |
|   访问方式  |   通过变量保存的值   |  通过保存的地址访问   |

## 浅复制与深复制
在浅复制时直接将简单类型值分别赋值给target,而如果复制的是对象就不能通过简单的复值来复制。这样做成的结果就是target引用值改变时也会引起原对象的改变。

### 1.浅复制
```
    var person1 = ['Nicholas','Greg',[{
						name : "linda",
						age : 21					
					},{
						name : "Nancy",
						age : 19
					}]
				]; 
	var person2 = [];
	// //复制
	for(var i  = 0;i < person1.length;i++){
	    person2[i] = person1[i]; 
	}
	person2[2].push('Leo') ;//改变color1的值
	console.log(person2);//'Nicholas','Greg',Array(3)
	console.log(person1);//'Nicholas','Greg',Array(3)
```
只复制第一层属性的方式为浅复制,如果数据类型全部是基本类型是可以成功的,而在复制引用类型时,则需要复制到基本类型为止才可以保证互不影响。

### 2.深复制

原生方法的基本实现方式:
```
    	var objA = {
	    a : "fc",
	    b : "Ig",
	    c : {
		d(){
		    console.log(1)
		}
	    }
	}
	function deepCopy(sub,sup){
	    for(var key in sup){
	        if(typeof sup[key] === 'object'){
		    sub[key] = {};
		    //复制到对象上的属性值为基本类型值为止
		    deepCopy(sub[key],sup[key]);
	        }else{
		    sub[key] = sup[key];
	        } 
	    }
	    return sub;
        }
	var objB = {};
	deepCopy(objB,objA);

	objA.c.d = function(){
		console.log(2)
	}
        //修改源对象上的属性并不会修改目标对象上已经复制的属性
	objB.c.d();//1
```

深复制的原理就是如果复制时如果复制的对象时引用类型,那么就递归运行复制一次,直到为简单数据类型为止。

## $.extend方法
在JQuery的extend方法中,如果传入的是一个或多个对象,那么就会将后面对象的属性复制给target对象,第一个参数可以选择是深复制(true)或浅复制,默认为浅复制,返回的是被扩展的对象。

1.合并但不修改object1。$.extend({}, object1, object2);
```
    var settings = {first:'hello', second: 'world'};
    var options = {first:'hello', second: 'JavaScript',third: 'nodeJs'};
    var results = $.extend({}, settings, options);
```

2.合并并修改第一个对象。$.extend(obj1,obj2)
```
    var obj1 = {first: 1, second: {height: 178, weight: 70,length:100}};
    var obj2 = {second: {height:180, weight:65, width: 90}, third: 90};
    $.extend(obj1, obj2);
    
    //输出结果为：{first: 1, second: {height:180,weight:65, width: 90}, third: 90}
```

3.深复制对象。$.extend(true,obj1,obj2)
```
    var obj1 = {first: 1, second: {height: 178, weight: 70}};
    var obj2 = {second: {height:180, weight: 65, width: 90}, third: 90};
    $.extend(true,obj1,obj2);
    console.log(obj1,obj2);
    //输出结果为:{first: 1, second: Object, third: 90}
```

## 深复制的不同种实现方式
已知的三种方式,$.extend、lodash、Underscore都有可以实现复制的功能,但是也会有一些细微的区别。

1.在Underscore中的_.clone(),如下：
```
    var x = {
        a: 1,
        b: { z: 0 },
        c:[
            2,3,9
        ]
    };

    var y = _.clone(x);
    y.c.push(29);
    console.log(x.c,y.c);//[2, 3, 9, 29][2, 3, 9, 29];
    //_.clone源码部分
    _.clone = function(obj) {
      if (!_.isObject(obj)) return obj;
      return _.isArray(obj) ? obj.slice() : _.extend({}, obj);
    };
    //数组使用slice方法截取所有,对象采用浅复制上的方法复制对象后按键值赋值,由此可见该功能并不能实现深复制
```
2.$.extend的复制方法
该方法下的深复制原理是：通过添加参数来实现递归extend,因此JQuery可以实现深复制。源码(3.2版本)如下：
```
    jQuery.extend = jQuery.fn.extend = function(){
    	var options, name, src, copy, copyIsArray, clone,
    		target = arguments[ 0 ] || {},// 常见用法 jQuery.extend( obj1, obj2 )，此时，target为arguments[0]
    		i = 1,
    		length = arguments.length,
    		deep = false;
            /*
            变量 options：指向某个源对象。
            变量 name：表示某个源对象的某个属性名。
            变量 src：表示目标对象的某个属性的原始值。
            变量 copy：表示某个源对象的某个属性的值。
            变量 copyIsArray：指示变量 copy 是否是数组。
            变量 clone：表示深度复制时原始值的修正值。
            变量 target：指向目标对象,申明时先临时用第一个参数值。
            变量 i：表示源对象的起始下标，申明时先临时用第二个参数值。
            变量 length：表示参数的个数，用于修正变量 target。
            变量 deep：指示是否执行深度复制，默认为 false。
            */
            
    	if( typeof target === "boolean" ){
    	//如果第一个参数为true，即 jQuery.extend( true, obj1, obj2 ); 的情况
    		deep = target;
    		target = arguments[ i ] || {};
    		i++;
    	}
    	
    	//Handle case when target is a string or something (possible in deep copy)
    	//比如$.extend({},{adress:"LosAngles"})
    	if( typeof target !== "object" && !jQuery.isFunction( target ) ){
    		target = {};
    	}
    	// Extend jQuery itself if only one argument is passed
    	// 处理这种情况 jQuery.extend(obj),或jQuery.fn.extend(obj)
    	if( i === length ){
    		target = this;
    		i--;
    	}
    
    	for( ; i < length; i++ ){
    		// Only deal with non-null/undefined values
    		if( ( options = arguments[ i ] ) != null ){
    			//比如 jQuery.extend(obj1,obj2,obj3,ojb4),options则为obj2、obj3...
    			for( name in options ) {
    				src = target[ name ];
    				copy = options[ name ];
    
    				// 防止自引用
    				if( target === copy ) {
    					continue;
    				}
    				// 如果是深拷贝，且被拷贝的属性值本身是个对象或数组
    				if( deep && copy && ( jQuery.isPlainObject( copy ) ||
    					( copyIsArray = Array.isArray( copy ) ) ) ) {
    					if ( copyIsArray ) {
    						copyIsArray = false;
    						clone = src && Array.isArray( src ) ? src : [];
    					} else {
    						clone = src && jQuery.isPlainObject( src ) ? src : {};
    					}
    					// Never move original objects, clone them
    					target[ name ] = jQuery.extend( deep, clone, copy );
                    //普通对象的定义是:通过 "{}"或者"new Object" 创建的	
    				//之前的例子走到了这一步,直接的赋值给对象,所以改变了源对象的属性后,target对象的属性也会发生改变
    				}else if( copy !== undefined ){
    					target[ name ] = copy;
    				}
    			}
    		}
    	}
    	return target;
    };
```

根据extend的源码分析,在关于深复制这部分的核心代码中,判断源对象的属性上是不是"普通对象"这个问题可能会引起深复制结果的错误。

例如在以下的代码中就可以看出,因为判断obj不是"普通对象",所以会影响深复制的结果,在这个例子中,当改变了源对象的属性时,目标对象的属性也被改变,显然这不符合深复制的目的。同样的问题在下面的lodash深复制中并不会出现。
```
    function Obj(){
        this.a = 1;
    }
    var obj = new Obj();
    var tobeCloned = {o:obj};
    var result  = $.extend(true,{},tobeCloned);
    tobeCloned.o.a = 2;
    console.log(result.o.a)//2
    
    console.log($.isPlainObject(obj));//false
```

3.lodash中的复制方法
复制的方法分别是_.clone()和_.cloneDeep()。其中_.clone(obj, true)等价于_.cloneDeep(obj)。
```
    var arr = new Int16Array(5),
    obj = { a: arr },
    obj2;
    arr[0] = 5;
    arr[1] = 6;
    
    //Int16Array是类型化数组。16位二补码有符号整数。
    // 1. jQuery
    obj2 = $.extend(true,{},obj);
    console.log(obj2.a);                            // [5, 6, 0, 0, 0]
    Object.prototype.toString.call(obj2);           // [object Int16Array]
    obj2.a[0] = 100;
    console.log(obj);                               // [100, 6, 0, 0, 0]
    
    //此处jQuery不能正确处理Int16Array的深复制！！！
    // 2. lodash
    obj2 = _.cloneDeep(obj);                       
    console.log(obj2.a);                            // [5, 6, 0, 0, 0]
    Object.prototype.toString.call(arr2);           // [object Int16Array]
    obj2.a[0] = 100;
    console.log(obj);                               // [5, 6, 0, 0, 0]
```

综合三种方法来看,JQuery不能复制JSON对象以外的对象。而在lodash中用了大量的代码来实现ES6引入的新标准对象,并且还可以对Date、RegExp进行深复制。单就深复制的实现上来说,lodash的效率和适用范围要优于JQuery。因此可以说lodash是一种更拥抱未来的类库。


## JQuery插件
JQuery插件主要分为两类：1，类级别 2，对象级别

 - 类方法。直接使用$类引用,不需要实例化就可使用。类方法在项目中被设置为工具类使用。
 - 对象级别。必须先创建实例，然后才能通过实例调用该实例方法。
 
### 1.$.extend扩展

直接扩展JQuery类,相当于静态方法,典型的方法有$.ajax,扩展方法:
```
    $.extend({
        add:function(a,b){
            return a + b;
        },
        divide:function(a,b){
            return a/b;
        }
    })
    //调用方式
    $.add(3,0);
    $.divide(9,3);
```
### 2.$.fn.extend()扩展插件
这种扩展方式是基于原型对象的,扩展插件时一般使用这种方式,扩展之后只有JQuery的实例才可以调用该方法,比如希望使页面上所有的连接转为红色。
```
    $.fn.myLink = function(){
		this.css({
			"color":"red"
		})
	}
	$("a").myLink();
```
如果需要对每个具体的元素进行操作,可以对该方法再次进行扩展。
```
    $.fn.myLink = function(){
		this.css({
			"color":"red"
		});
		this.each(function(){
		    $(this).append($(this).attr("href"));
		})
	}
	$("a").myLink();
```
注意在each的内部遍历出来的是DOM元素,所以需要在包装一次才可以使用JQuery的方法。而如果我们希望可以自己定制,根据自身需求来设置，所以可以利用$.extend方法合并对象之后来作为参数使用,在没有参数时使用默认值。
在extend时使用空对象作为第一个参数,避免修改defaults默认的属性值,保护好defaults的默认参数。
```
    $.fn.myLink = function(options){
    	var defaults = {
    	    "color" : "red",
    	    "fontSize" : "18px",
    	    "lineHeight" : "18px"
    	}
    	//此处使用空对象是为了保护默认参数,避免被修改之后复用出错,注意默认还是浅复制,如果options有引用类型参数时,还是会对defaults造成印象
    	var setting = $.extend({},defaults,options);
    	return this.css({
    	    "color" : setting.color,
    	    "fontSize" : setting.fontSize,
    	    "lineHeight" : setting.lineHeight
    	})
    }
    $("a").myLink({
    	"color":"#333"
    });
```

### 3.面向对象的插件开发
为什么需要面向对象的插件方法？方便管理直接使用，第二不会影响外部命名空间。
```
    ;(function($,window,document,undefined){
        var Beautify = function(ele,opt){
            this.$elements = ele,
            this.defaults = {
                "color" : "red",
    			"fontSize" : "18px",
    			"textShadow" : "none"
            },
            this.options = $.extend({},this.defaults,opt);
        }
        Beautify.prototype = {
            constructor : Beautify,
            beautiful(){
                return this.$elements.css({
                    "color" : this.options.color,
        			"fontSize" : this.options.fontSize,
        			"textShadow" : this.options.textShadow
                })
            }
        }
        $.fn.myPlug = function(options){
            //this指向新的实例beautify,其中有一个函数beautify(),可以返回指定样式;
            var beautify = new Beautify(this,options);
            return beautify.beautiful();
        }
    })(jQuery,window,document,undefined);
    $("").myPlug({
        "fontSize":"30px",
		"textShadow": "3px 2px 3px #ff0000"
    })
```

