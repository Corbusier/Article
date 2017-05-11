#DOM2 和 DOM3


DOM1级定义的是HTML和XML文档的底层结构,DOM2和DOM3级则在这个结构的基础上引入更多的交互能力,也支持更高级的XML特性。

## 1.样式
### 1. 访问元素的样式
任何支持style特性的HTML元素在JS中都有对应的style属性。这个style对象是CSSStyleDeclaration的实例,包含行间样式的信息,但不包括链入和嵌入样式表的样式信息。在JS中定义样式时,对于短划线属性必须使用驼峰大小写形式。同样通过JS代码读取的也是行间样式信息。
#### ①.DOM样式属性和方法
通过CSSText属性可以访问行间样式,在写入模式下,可以为元素应用多项样式,一次性的应用所用样式。
```
    content.style.cssText = "width:200px;height:100px;background-color:green";
```
length属性可以迭代在元素中定义的CSS属性。
getPropertyValue( )方法,可以返回指定属性的字符串值。
示例:
```
    var prop,
        value,
    for(var i = 0;i<content.style.length;i++){
        prop = content.style[i];
        value = content.style.getPropertyValue(prop);
        alert(prop + ":" + value);
    }
```
#### ②.计算的样式
虽然style对象能获取到样式,但是都是行间样式,如果想获取其他方式叠加而来的样式信息则需要这个方法,"DOM2级样式"增强了document.defaultView,提供了getComputedStyle()方法。这个方法接收两个参数,要取得计算样式的元素,一个伪元素字符串(例如":after"),如果不需要伪元素信息,第二个参数可以是null。
```
    var div = document.getElementsByTagName("div")[0];
    var computedStyle = document.defaultView.getComputedStyle(div,null);
    console.log(computedStyle.width);
    console.log(computedStyle.backgroundColor);
```
在获取这个对象时,如果CSS属性是一个综合属性(border等),不一定会返回值,例如Firefox就不能返回boredr属性值。

### (2).元素大小
#### ①.元素偏移量
元素的可见大小由高度、宽度决定,包括所有内边距、滚动条和边框大小。通过以下四个属性可以获得元素的偏移量：

 - offsetHeight:包括元素的高、滚动条(可见的)宽度[17px],边框、内边距；
 - offsetWidth:同上。宽度；
 - offsetLeft:子级元素左外边框与父级元素左内边框的距离,如果没有定位父级则为body；
 - offsetTop:子级元素上外边框和父级元素的上内边框的距离,如果没有定位父级则为body；

    
定位父级偏移量的属性与父节点的值不一定相同,td元素的定位父级元素是table元素,因为table是在DOM层次中距td最近的一个具有大小的元素。

如果想知道某元素在页面上的偏移量,可以将这个元素的offsetLeft与定位父级的offsetLeft相加,循环至根元素,就可以得到准确的值。
```
    function getOffsetLeft(element){
        var actualLeft = element.offsetLeft;
        var current = element.offsetParent;
        
        while(curren != null){
            actualLeft += current.offsetLeft;
            current = current.offsetParent;
        }
        return actualLeft;
    }
```
对于一般的元素这个函数够用,但如果是表格元素或内嵌架构布局,由于不同的浏览器实现这些元素的方式不同,因此得到的值就不太准确。还有一点要注意就是偏移量属性是只读的,每次访问都需要重计算,如果需要重复使用某些属性值,可以将他们保存在变量中以提高性能。

#### ②.客户区大小
指的是元素内容及内边距所占据空间的大小。clientWidth、clientHeight属性,包含的是内容尺寸、内边距。不包含滚动条、边框尺寸。当内容出现滚动条时,还需要减去滚动条的宽度(17px)。如果要确定视口的尺寸,可以使用document.documentElement或document.body(IE7之前的版本中)中的clientWidth和clientHeight。
```
    function getViewPort(){
        //混杂模式渲染
        if(document.compatMode == "BackCompat"){
            return {
                width : document.body.clientWidth,
                height : document.body.clientHeight
            };
        }else{
            return {
                width : document.documentElement.clientWidth,
                height : document.documentElement.clientHeight
            };
        }
    }
```

#### ③.滚动大小
指的是包含滚动内容的元素的大小,即被子元素撑开的尺寸。以下是4个与滚动大小相关的属性:

 - scrollWidth。子级元素撑开的宽度。子级元素的宽度+(2*父级级padding)；
 - scrollHeight。子级元素撑开的宽度。子级元素的高度+(2*父级padding)；
 - scrollLeft。隐藏在元素内容区左侧的像素值,可读可写；
 - scrollTop。隐藏在元素内容区顶部的像素值,可读可写,当页面拉到底部时,scrollHeight == scrollTop  + clientHeight；

```
    <div id="test" style="width: 100px;height: 100px;padding: 10px;margin: 10px;border: 1px solid black;overflow:scroll;font-size:20px;line-height:200px;box-sizing:conetnt-box;">
    	    内容
    </div>
	<button id='btn1'>点击</button>
	<div id="result"></div>
	
	<script>
	btn1.onclick = function(){
		    result.innerHTML = 'scrollTop:' + test.scrollTop+';clientHeight:' + test.clientHeight + ';scrollHeight:' + test.scrollHeight
		}
	</script>
```
在浏览器中基于document.documentElement使用clientWidth和scrollWidth时,如果不足以产生滚动条的情况下,这两个数值是相等的,都等于视口的大小,该电脑下为1366px;当有滚动条出现时,clientWidth代表的仍然是视口的大小(1366-17=1349px),而scrollWidth代表的是文档内容区域的大小;
        
在确定文档的总高度时,必须取得scrollWidth/clientWidth和scrollWidth/clientWidth中的最大值,才能保证跨浏览器环境下得到精确结果。
```
    var docWidth = Math.max(document.documentElement.clientWidth,document.documentElement.scrollWidth);
    var docHeight = Math.max(document.documentElement.clientHeight,document.documentElement.clientHeight);
```

#### ④.确定元素大小
getBoundingClientRect()方法,该方法会返回一个矩形对象,在IE下包含四个属性：top、left、right、bottom。这些属性给出了元素在页面中相对于视口的位置。在IE8以前的浏览器中以文档的(2,2)作为原点,IE9之后的其他标准浏览器都为(0,0)。要修正这个小bug可以这样做：
```
    function GetBoundingClient(element){
        if(typeof aruguments.callee.offset != "number"){
            var scrollTop = document.documentElement.scrollTop;
            var temp = document.createElement("div");
            temp.style.cssText = "position:absolut;top:0;left:0";
            document.body.appendChild(temp);
            arguments.callee.offset = -temp.getBoudningClientRect().top - scrollTop;
            document.body.removeChild(temp);
            temp = null;
        }
        var rect = element.getBoundingClientRect();
        var offset = arguments.callee.offset;
        
        return{
            left : rect.left + offset;
            right : rect.right + offset;
            top : rect.top + offset;
            bottom : rect.bottom + offset;
        };
    }

```
首先验证这个属性有没有被创建,如果没有就定义这个属性,在文档区先创建一个元素,这个元素是不可见的,这是为了定位该浏览器下的原点,然后给这个值设为负数,为了结构考虑,在这之后将这个元素移除出DOM树。如果是其他浏览器下offset值应该为0,而IE8之前的版本为-2,所以只需要减去这个值即可得到正确的值。


#### 图片懒加载Example：
```
    //原理：图片元素盒模型的顶部小于可视区的高度,说明图片此时已经进入了文档可视区,此时可以获取图片中的自定义属性,此法可以有效地节省加载时间,有利于页面性能优化。
    <img src="" data-src="img/img02.jpg" alt="">
    
    function lazyLoad(){
        for(var i = 0;i<imgs.length;i++){
            var disH = imgs[i].getBoundingClientRect().top;
            var clientH = document.documentElement.clientHeight;
            if(disH < clientH){
                imgs[i].src = imgs[i].dataset.src;
            }
        }
    }
    lazyload();
    window.onscroll = lazyload;
    
```
以上的写法会导致每当屏幕滚动时就会调用该函数,该函数高频触发显然不是最优解,所以可以通过节流函数进行性能优化。节流函数：只允许函数在N秒内执行一次。
```
    function throttle(fun, delay, time) {
    var timeout,
        startTime = new Date();

    return function() {
        var context = this,
            args = arguments,
            curTime = new Date();

        clearTimeout(timeout);
        // 如果达到了规定的触发时间间隔，触发 handler
        if (curTime - startTime >= time) {
            fun.apply(context, args);
            startTime = curTime;
            // 没达到触发间隔，重新设定定时器
        } else {
            timeout = setTimeout(fun, delay);
        }
    };
};

```
更多关于页面滚动的优化：[页面滚动优化](https://github.com/Corbusier/Article/blob/master/%E9%A1%B5%E9%9D%A2%E6%BB%9A%E5%8A%A8scroll%E5%8F%8A%E6%B8%B2%E6%9F%93%E4%BC%98%E5%8C%96.md)