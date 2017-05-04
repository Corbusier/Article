# 页面滚动scroll及渲染优化


## 滚动优化
页面scroll事件上如果绑定了某些函数,那么就会频繁的触发他们,加入涉及到很多的运算、DOM操作、元素重绘,那么可能无法在下一次scroll触发前完成,引起浏览器掉帧,影响用户体验。

## 页面滚动与渲染的关系
在chrome下一个Web页面会经历几个步骤:

 1. JvaScript
 2. Style：
    计算样式,确定了每个 DOM 元素上该应用什么 CSS 样式规则。
 3. Layout
    具体计算每个 DOM 元素最终在屏幕上显示的大小和位置。
 4. Paint
    绘制，本质上就是填充像素的过程。包括绘制文字、颜色、图像、边框和阴影等，也就是一个 DOM 元素所有的可视效果。
 5. Composite
    渲染层合并，由上一步可知，对页面中 DOM 元素的绘制是在多个层上进行的。在每个层上完成绘制过程之后，浏览器会将所有层按照合理的顺序合并成一个图层，然后显示在屏幕上。


简单来说，网页生成的时候，至少会渲染（Layout+Paint）一次。用户访问的过程中，还会不断重新的重排（reflow）和重绘（repaint）。
其中，用户 scroll 和 resize 行为（即是滑动页面和改变窗口大小）会导致页面不断的重新渲染。

## 防抖与节流函数

### 防抖(Debouncing)

```
    function Debouncing(fn,delay){
        let timer = null;
        return function(){
            let context = this;
            let args = arguments;
            clearTimeout(timer);
            timer = setTimeout(function(){
                fn.apply(context,args);
            },delay)
        }
    }
```
当下一次再执行函数时,先把之前的清除之后,再开启新的定时器继而执行fn,这样在一定的时间内,规定事件函数被触发的次数。

### 节流(Throttling)
防抖函数并不一定适应所有的场景,比如当图片懒加载时,期望下滑过程中不断加载,而不是停止时才加载。

在这样的前提下,希望即使页面不断滚动,但是滚动函数也可以按照一定的频率触发,这就需要节流函数(Throttling)。允许函数在X毫秒内执行一次。和上一种方法相比最大的不同在于保证在X毫秒内至少执行一次希望出发的事件handler。
```
    function Throttling(func,delay,mustRun){
		let timer,
			startTime = new Date();
		return function(){
			let context = this,
				args = arguments,
				curTime = new Date();

			clearTimeout(timer);
			if(curTime - startTime >= mustRun){
				func.apply(context,args);
				startTime = curTime;
			}else{
				timer = setTimeout(func,delay);
			}
		}
	}
	function realFunc(){
		console.log("Success");
	}
	window.addEventListener('scroll',Throttling(realFunc,500,1000));
```
这样做之后如果在一段时间内scroll触发的间隔小于500毫秒,那么能保证至少在1000毫秒内触发一次调用的函数。

### 使用rAF(requestAnimationFrame)触发滚动事件
如果页面只需要兼容高版本浏览器或应用在移动端，又或者页面需要追求高精度的效果，那么可以使用浏览器的原生方法 rAF(requestAnimationFrame)。

#### requestAnimationFrame
window.requestAnimationFrame() 这个方法是用来在页面重绘之前，通知浏览器调用一个指定的函数。这个方法接受一个函数为参，该函数会在重绘前调用。

通常来说，rAF被调用的频率是每秒60次,也就是 1000/60 ，触发频率大概是16.7ms 。（当执行复杂操作时，当它发现无法维持 60fps 的频率时，它会把频率降低到 30fps 来保持帧数的稳定。)
使用 requestAnimationFrame 来触发滚动事件，相当于上面的：
```
    throttle(func, xx, 1000/60)//xx代表 xx ms内不会重复触发事件 handler
```
示例：
```
    var ticking = false; //rAF触发锁
    function onScroll(){
      if(!ticking) {
        requestAnimationFrame(realFunc);
        ticking = true;
      }
    }
    function realFunc(){
    	// do something...
    	console.log("Success");
    	ticking = false;
    }
    // 滚动事件监听
    window.addEventListener('scroll', onScroll, false);
```
在不考虑兼容性时,因为他只能实现16.7ms的频率触发,代表可调节性很差,但是有利于精确度还原,常用于页面的帧刷新渲染,动画效果更流畅。

以上三种方式都可以避免scroll事件过度消耗资源,但是还是建议在scroll事件中涉及大量计算和样式操作的环节去除。