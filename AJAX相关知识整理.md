# AJAX相关知识整理


---
## 全平台兼容的XMLHttpRequest对象
    在标准浏览器中可以使用XMLHttpRequest()创建Ajax请求的对象。

    IE的>=7以上版本支持XMLHttpRequest对象,在之前的版本中需要指明一个类似于"Microsoft.XMLHttp"的progID,且在不同的操作系统下,以下的ProgId都可以创建XMLHttp对象。
```
		Microsoft.XMLHTTP
		Microsoft.XMLHTTP.1.0
		Msxml2.ServerXMLHTTP
		Msxml2.ServerXMLHTTP.3.0
		Msxml2.ServerXMLHTTP.4.0
		Msxml2.ServerXMLHTTP.5.0
	    	Msxml2.ServerXMLHTTP.6.0
		Msxml2.XMLHTTP
		Msxml2.XMLHTTP.3.0
		Msxml2.XMLHTTP.4.0
		Msxml2.XMLHTTP.5.0
		Msxml2.XMLHTTP.6.0
```
		以上的版本中6的功能最强,且更安全,但是只有Vista系统及以上才可使用,4已经被5完全替代,而5是针对Office办公场景的,在没有安装office2003前提下,不一定能使用它。所以3是6的优雅降级版本,适用于Win2k SP4及以上系统。
```
    function getXHR(){
	    var xhr = null;
	    if(window.XMLHttpRequest){
		    xhr = new XMLHttpRequest();
	    }else if(window.ActiveXObject){
	        try{
		    xhr = new ActiveXObject("Msxml2.XMLHttp");
	        }catch(e){
			try{
			    xhr = new ActiveXObject("Microsoft.XMLHttp");
			}catch(e){
			    alert("您的浏览器暂不支持Ajax!")
	            }
		}
	    }
	    return xhr;
    }
```
## Ajax有没有破坏JS单线程机制
一般情况下,浏览器分为以下几个线程:

 1. GUI渲染线程
 2. JS引擎线程
 3. 浏览器事件触发线程
 4. HTTP请求线程
通常情况下,线程间交互以事件形式发生,通过事件回调的方式予以通知,而事件回调又是先入先出的队列形式将时间添加到任务队列的末尾,等JS引擎空闲时,再依次执行排队的任务,事件回调包括setTimeout、setInterval,click,ajax异步请求等回调。

    在浏览器中JS引擎线程会循环从任务队列中读取事件并执行,这样的机制被称为Event Loop(时间循环)。
        
    对于Ajax请求,先创建对象,在执行open调用send方法至此均为同步执行,直达send内部开始,浏览器为将要发生的网络请求创建了新的HTTP请求线程,它与JS引擎线程相对独立,网络异步请求发送时,JS引擎并不会等到Http收到结果,而是直接顺序向下执行。

    当Ajax请求被服务器响应并受到response后,浏览器事件触发机制捕获到了Ajax回调函数onreadystatechange(也可能是onload,onerror等)。该事件并没有立即执行,而是添加到任务队列的末尾,等到JS引擎空闲了再执行。
    
    在onreadystatechange事件内部中,可能会有DOM操作,此时便会挂起JS引擎线程,执行GUI渲染,当JS引擎重新执行时GUI渲染线程才会挂起,GUI更新被保存起来,等JS引擎空闲后再立即执行。
    以上的整个Ajax请求过程中,涉及到了四个线程,除了互斥的GUI渲染和JS引擎线程,其他都是可以并行执行的,通过这样的方式,ajax并没有破坏js的单线程机制。

## Ajax与setTimeout排队问题
通常情况下,Ajax会与setTimeout同等对待,按照顺序自动的添加到任务队列的末尾,等待JS引擎空闲时执行,但并非xhr的所有回调执行都滞后于setTimeout的回调。例如:
```
    function ajax(url,method){
        var xhr = getXHR();
        xhr.onreadystatechange = function(){
            console.log("xhr.readyState:" + this.readyState);
        }
        xhr.loadstart = function(){
            console.log("loadstart");
        }
        xhr.load = function(){
            console.log("onload");
        }
        xhr.open(method,url,true);
        xhr.setRequestHeader("Cache-Control",3000);
        xhr.send();
    }
    var timer = setTimeout(function(){
        console.log("setTimeout")
    },0);
    ajax("http://www.baidu.com","GET")
    console.warn("这里并不是最先打印出来的")
```
结果为:
```
    xhr.readyState:1
    onloadStart
    这里并不是最先打印出来
    xhr.readyState:4
```
    由此可见并非所有的Ajax请求都是异步的,至少readystatechange回调及loadstart回调都是同步函数。
    
## 原生AJAX封装方法
```
    function Ajax(params){
		params = params || {};
		params.data = params.data || {};
		//判断是ajax请求还是jsonp请求
		var json = params.jsonp?jsonp(params):json(params);
		
		//ajax请求
		function json(params){
			params.type = (params.type || "GET").toUpperCase();
			//格式化传输数据
			params.data = formatParams(params.data);

			var xhr = getXHR();
			//readyState变化就会调用readystatechange 事件
			xhr.onreadystatechange = function(){
				if(xhr.readyState == 4){
					var status = xhr.status;
					var response;
					if(status>=200 && status<300 || status == 304){
						//判断接收数据的内容类型
						var type = xhr.getResponseHeader("Content-Type");
						if(type.indexOf("xml") !== -1 && xhr.responseXML){
							response = xhr.responseXML;
						}else if(type == "application/json"){
							response = JSON.parse(xhr.responseText);//返回JSON响应
						}else{
							response = xhr.responseText;//字符串响应
						}
					}
					//成功时返回数据
					params.success && params.success(response);			
				}else{
					params.error && params.error(status);
					//失败时直接返回失败状态码
				}
			}
			
			if(params.type == "GET"){
				//GET需要将URL末尾的字符串进行编码之后才可以使用
				xhr.open(params.type,params.url + "?" + params.data,true);
				xhr.send(null);
			}else{
				xhr.open(params.type,params.url,true);
				//模拟表单设置请求头,提交时的内容类型
				xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded;");
				xhr.send(params.data);
			}

			function formatParams(data){
				var arr = [];
				for(var key in data){
					arr.push(encodeURIComponent(key) + "=" + encodeURIComponent(data[key]));
				};
				//添加随机数参数,防止缓存
				arr.push("v=" + random());
				return arr.join("&");
			}
			function random(){
				return Math.floor(Math.random() * 1000 + 500);
			}
			//除了设置随机数之外,还可以设置请求头来防止缓存
			xhr.setRequestHeader("Cache-Control","no-cache");
			
			//在JQuery中在setting里面指定或者全局关闭缓存
			cache:true
			$.ajaxSetup({cache:false});
		}
	}
	//使用实例:
	Ajax({
		url : "test.php",
		type : "POST",
		data : {"carry" : "异步请求"},
		success : function(res){
			console.log(JSON.parse(res));
		},
		error : function(error){
			console.log("Not Found");
		}
	})
```
## JSONP
### 同源策略
    
AJAX需要跨域就是因为浏览器的同源策略。一个页面的AJAX只能获取这个页面相同源或相同域的数据。同源或者同域----协议、域名、端口号都必须相同。

可以使用JSONP进行跨域请求,利用script标签可以跨域请求的特性,由其src属性发送请求到服务器,服务器返回JS代码,浏览器接受响应,然后直接执行。这个与加载外部文件原理相同。

JSONP由两部分组成：回调函数和数据,回调函数一般是在浏览器控制,作为参数发往服务器端。当服务器响应时,服务器端就会把该函数和数据拼成字符串返回。

请求过程：

 - 请求阶段：浏览器创建一个script标签,并给其src赋值(类似于http://example.com/api/?callback=jsonpCallback)
 - 发送请求：当给script标签的src赋值时,浏览器就会发起一个请求。
 - 数据响应：服务端将要返回的数据作为参数和函数名称拼接在一起返回。当浏览器接收到了响应数据,由于发起请求的是script,所以相当于直接调用jsonpCallback方法,并且传入了一个参数。

JSONP的配置参数多了一个jsonp参数,即callback函数名。
```
    function Ajax(params){
		function jsonp(params) { 
		      //创建script标签并加入到页面中 
		      var callbackName = params.jsonp; 
		      var head = document.getElementsByTagName('head')[0]; 
		      // 设置传递给后台的回调参数名 
		      params.data['callback'] = callbackName; 
		      var data = formatParams(params.data); 
		      var script = document.createElement('script'); 
		      head.appendChild(script);  
		      
		      //创建jsonp回调函数 
		      window[callbackName] = function(json) { 
		       head.removeChild(script); 
		       clearTimeout(script.timer); 
		       window[callbackName] = null; 
		       params.success && params.success(json); 
		      };  
		    

		      //发送请求 
		      script.src = params.url + '?' + data;  
		    

		      //为了得知此次请求是否成功，设置超时处理 
		      if(params.time) { 
		      script.timer = setTimeout(function() { 
		       window[callbackName] = null; 
		       head.removeChild(script); 
		       params.error && params.error({ 
		        message: '超时' 
		       }); 
		      }, time); 
		      } 
		     };  
		    

		     //格式化参数 
		     function formatParams(data) { 
		      var arr = []; 
		      for(var name in data) { 
		       arr.push(encodeURIComponent(name) + '=' + encodeURIComponent(data[name])); 
		      };
		      // 添加一个随机数，防止缓存 
		      arr.push('v=' + random()); 
		      return arr.join("&"); 
		     } 
		    

		     // 获取随机数 
		     function random() { 
		      return Math.floor(Math.random() * 10000 + 500); 
		     }
		}
	}
```
注意：script标签只在第一次设置时起作用,导致script标签没法重用,所以每次操作完之后都要移除;
使用实例：
```
    ajax({
        url : "test.php",
        jsonp : "jsonpCallback",
        data : {"carry" : "异步请求"},
        success : function(res){
            console.log(res);
        },
        error : function(error){
            console.log("Not Found");
        }
    })
```
## XMLHttpRequest实例的属性

 - readyState属性
    从0-4代表了HTTP请求的状态。
    0，UNSENT。实例已经生成,但是open方法还没有被调用。
    1，Opened。open已调用,但send还没有被调用,仍然可以使用setRequestHeader,设定请求头。
    2，HEADERS_RECEIVED,send方法已经执行,并且头信息和状态码已经收到。
    3，LOADING,正在接收服务器传来的主体部分的数据。
    4，DONE,数据接收完成,或者接收失败。
 
 - status
    只读属性,表示本次请求得到的HTTP状态码。
```
    200，OK，访问正常
    301，永久移动
    302，暂时移动
    304，未修改
    307，暂时重定向
    401，未授权
    403，禁止访问
    404，未发现指定网址
    500，服务器发生错误
```
 - statusText
 只读属性,访问一个字符串,表示服务器发送的状态提示。不属于status属性的是,该属性包含整个状态信息,比如""200 OK"。
 - timeout
 设定超限时间,超过了时间没有得到结果就会自动终止,关于该属性的监听函数:
```
    xhr.ontimeout = function(){
        //时间超时
    }
```
 - response
 只读属性,返回接收到的数据体。可以是ArrayBuffer(缓冲数组)、Blob(二进制对象)、Document、JSON对象或者一个字符串。如果请求成功或者数据不完整，该属性就会等于null。
 - responseType
 指定返回数据(xhr.response)的类型
```
    "":字符串(默认值)
    "arraybuffer":ArrayBuffer对象
    "blob":Blob对象
    "document":Document对象
    "json":JSON对象
    "text":字符串
```
blob对象适用于图片文件,Document类型适合XML文档,大多数情况下使用text类型
 - responseText
 只读属性,请求不成功或数据不完整,该属性为null,如果返回的数据格式是JSON,就可以使用responseText属性
```
    var data = xhr.responseText;
    data = JSOn.parse(data);
```
 - 事件监听接口
 在XHR 1.0版本中只能对onreadystatechange指定回调函数,而xhr 2.0中允许更多的事件指定回调函数
```
    onloadstart 请求发出  
    onprogress 正在发送和加载数据  
    onabort 请求被中止，比如用户调用了abort()方法  
    onerror 请求失败  
    onload 请求成功完成  
    ontimeout 用户指定的时限到期，请求还未完成  
    onloadend 请求完成，不管成果或失败
```
## XHR一级与XHR二级
XHR一级的缺点：
    
 - 仅支持文本数据传输,无法传输二进制数据。
 - 传输数据时,没有进度消息提示
 - 受浏览器同源策略限制,只能请求同域资源
 - 没有超时机制,不方便掌控ajax请求节奏

XHR二级的改进：

 - 支持二进制数据,可以上传文件,可以使用FormData对象管理表单
 - 提供进度支持,通过xhr.upload.onprogress事件回调函数获取传输进度
 - 仍然受同源策略限制,XHR二级新提供Access-Control-Allow-Origin等headers,设置为   从而实现跨域访问
 - 可以设置timeout和ontimeout,方便设置超时时长和超时后续处理

IE10以上都支持XHR2,对于IE8,9用户使用只对IE有效的的过渡方案。

## JQuery中的AJAX
直接封装在JQuery类上的静态方法,JQ所有的ajax请求都是通过该方法实现的。在1.5版本之后的XHR对象实现了promise。

| 新方法      | 被替代的方法   | 
| ---------   | --------  | 
| done(function(data,textStatus,jqXHR){})  | success |
| fail(function(jqXHR,textStatus,errorThrown){}) |   error   | 
| always(function(data or jqXHR,textStatus,jqXHR or errorThrown){}) |    complete   |
 
以上的三种方法按照队列函数原则可以分配多个回调。

## cookie、localstorage、sessionstorage

### 共同点
都可以在浏览器端上储存数据。
### 区别
 - 大小限制
 - 储存路径
 - 接口数量
 - 生存周期
 - 参与通信,发送给服务器

具体如下：

cookie主要用于用户的识别身份,且每次的通信过程中都会在HTTP的头部中携带,为了节省带宽优化性能,一般只有4K左右大小。默认是关闭浏览器后失效,可以设置失效时间。

webstorage的两种方式不会参与到通信中,虽然有大小的限制,但可以>=5M,接口也更为丰富,可以将数据更新的通知发送给监听者。

其中localstorage可以永久保存,除非手动移除,否则手动的移除。
sessionstorage只在会话阶段有效,之后就失效了,作用域也不相同,在不同的浏览器窗口中,即使是同一个页面,sessionstorage也也不可以共享。而localstorage在所有同源窗口中共享。

### webstorage的用法
保存数据：localStorage.setItem(key,value);
读取数据：localStorage.getItem(key);
删除单个数据：localStorage.removeItem(key);
删除所有数据：localStorage.clear();
得到某个索引的key：localStorage.key(index);示例：

#### 监听 storage 事件
触发事件的时间对象（e 参数值）有几个属性：

 1. key : 键值。
 2. oldValue : 被修改前的值。
 3. newValue : 被修改后的值。
 4. url : 页面url。
 5. storageArea : 被修改的 storage 对象。

对应的处理事件函数：
```
    window.addEventListener('storage',function(e){
        console.log('key='+e.key+',oldValue='+e.oldValue+',newValue='+e.newValue);
    })
```

## AJAX跨域请求
### 什么是CORS(cross-origin-resource-sharing)
允许AJAX向跨域服务器发出异步http请求,克服同源策略的限制,实际上浏览器并不会阻止不合法的跨域请求,服务器收到了请求,只是浏览器拦截了传回来的响应,因此不合法(除了Chrome和FF下的https网站不允许发送http异步请求)。

### 跨域的实现原理
通过自定义的HTTP头部让浏览器与服务器进行沟通,从而决定请求或相应是成功还是失败。在请求时附加一个Orgin头部,其中包含请求页面的源信息(协议、域名和端口),以便服务器根据这个头部信息确定是否给予响应。示例：
```
    Origin:http://www.nczonline.net
```
如果服务器认为可以接受,就在Access-Control-Allow-Origin头部返回相同的源信息,出于安全策略的考虑,跨域时请求和响应都不能包含cookie信息。


### IE8、9的XDomainRequest

XDomainRequest仅可用于发送GET和POST请求,如下即创建过程：
```
    var xdr = new XDomainRequest();
```
xdr具有以下属性：

 - timeout
 - responseText
如下方法：
 - open：只能接收Method,url两个参数。只有post和get方式,只能发送异步请求
 - send
 - abort
如下事件回调：
 - onprogress
 - ontimeout
 - onerror
 - onload

还有两点需要注意的是：

 1. XDomainRequest不支持跨域传输cookie
 2. 只能设置请求头的Content-Type字段,不能访问响应头信息(getResponseHeader/getAllResponseHeaders),不能获取到status属性和statusText,只能获取到原始的响应文本。

### 其他浏览器的跨域实现
针对XHR对象的原生支持,无需额外添加代码,只要在open时加入绝对的URL即可。与IE中的XDR不同的是,跨域的XHR可以访问status、statusText属性。并且支持同步请求。出于安全策略考虑也有一些限制：

 - 不能使用setRequestHeader设置自定义头部；
 - 不能发送、接收cookie；
 - 调用getAllResponseHeaders方法总会返回空字符串；

### 其他跨域技术

#### JSONP
JSON with padding的简写,应用JSON的一种新方法。包含在函数调用中的JSON,如下所示：
```
    callback({"name":"Nicholas"});
```
JSONP由两部分组成,回调函数和数据。回调函数的名字一般是在请求中指定的。而数据就是传入回调函数中的JSON数据。下面是一个典型的JSONP请求:
```
    http://freegeoip.net/json/?callback=handleResposne
```
JSONP是通过动态script元素来使用的,使用时可以为src属性指定一个跨域URL,这里的script元素和img元素类似,都能不受限制的从其他域加载资源。作为一段有效的代码,在加入body之后,就可以立即执行返回JSON数据的callback。
```
    function handleResponse(response){
        alert("You're at IP address" + response.ip + ",which is in " + response.city + ", " + response.region_name);
    }
    var script = document.createElement("script");
    script.src = "http://freegeoip.net/json/?callback=handleResponse";
    document.body.insertBefore(script,document.body.firstChild);
```
JSONP的优点在于可以实现浏览器与服务器之间的双向通信,能够直接访问响应文本。但是也存在着两点缺点：
```
    1.跨域使用url加载src属性,如果来源域不安全,那么除了完全放弃调用之外别无他法。
    2.确认请求是否失败并不容易。
```
#### commet
这是一种从服务器向页面推送数据的技术,可以让信息实时的推送到页面上,非常适合体育比赛的分数和股票报价。

一个简单的例子,如果把一次请求看作是去隔壁寝室找同学的过程。短轮询需要过去看一次得到回应(在/不在)之后马上回自己的寝室,再循环这个过程。而长轮询是看到如果不在,那么会接着等,直到等到了再回去,停止这一次的过程,然后又开启新的之前的过程。

    所有的浏览器都支持长轮询。

----------------------------------------  待续补充  --------------------------------------- 

## csrf、xss概念及如何防范
