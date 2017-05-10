#CSS中的居中方式



---

刚学习CSS的时候尝试过几种居中的方法,这些方法不需要借助JS手段,所写的方法有一个原则,就是在不需要直接人为的设定好宽高计算后再实现居中,还有诸如table布局啊、行高设定、margin：auto之类的我就不写了。

以下几种方法针对不同的浏览器,经过测试,从IE8及以下到IE9+、Safari都可以支持。

##1.标准浏览器全兼容,内容水平居中

```
    .wraper{
		float:left;
		position:relative;
		left:50%;
		clear:both;
	}
	.wraper div{
		border:1px solid palevioletred;
		position:relative;
		left:-50%;
	}

    <div class="wraper">
		<div>瓜迪奥拉</div>
	</div>
	<div class="wraper">
		<div>克洛普克洛普克洛普克洛普克洛普克洛普克洛普</div>
	</div>
	<div class="wraper">
		<div>孔蒂</div>
	</div>
	<div class="wraper">
		<div>穆里尼奥</div>
	</div>
	<div class="wraper">
		<div>温格</div>
	</div>
```
这种方法的原理是让容器层和内层都相对自己移动,容器层移动自身宽度的50%,内层相对于移动自身宽度的-50%。这样一来外部容器正好可以将内容垂直的包裹住,并且由于外部容器是浮动的,所以容器的宽度和内层的宽度一致,这样可以做到完全自适应的实现居中。该方法适用于竖向排版的情况。

##2.IE9+以上的浏览器,垂直居中
```
    div{
        position: absolute;
        width:100px;
        height:50px;
        top:0;
        right:0;
        bottom:0;
        left:0;
        margin:auto;
        background:#f60;
    }

    <div></div>
```
这个方法原理不太清楚,我是这样理解的,当这个div哪里也去不了,然后margin还是auto的时候,他就只能相对于外部容器垂直居中了..

##3.IE8及以下的垂直居中
```
    .parent{
		height:500px;
		width:500px;
		font-size:438.6px;/*(font-size:width/114)*/
		background-color:paleturquoise;
	}
	.child{
		background-color:deepskyblue;
		vertical-align:middle;
		zoom:1;
		display:inline;
		width:50px;
		height:50px;
		font:18px/18px "微软雅黑";
	}
	/*.child一定要写上font属性,这个方法在ie中才会实现.*/
	
    <div class="parent">
		<div class="child"></div>
	</div>

```
这个方法只有IE8及以下才可以实现,至今也没明白父级的font-size属性为什么一定要等于宽度/114,IE真是一个奇葩的存在...在虚拟机下测试这个方法确实有效,子级一定要写font-size属性,即使没有内容也要写,否则无效。针对IE8以下的兼容性时,这个方法可以作为一种hack使用。

##4.除IE和Safari之外的标准浏览器
IE是指9及以下,我只测试了这些版本的IE,另外Safari也是不支持的。
```
    .box{
        width:600px;
        height:600px;
        border:1px solid palegreen;
        position:relative;
    }
    .item{
        width:200px;
        height:200px;
        border:1px solid palegreen;
        position:absolute;
        top:50%;
        left:50%;
        transform:translate(-50%,-50%);         
    }
    
    
    <div class="box">
        <div class="item"></div>
    </div>
```
容器和内层都已经脱离了文档流,内层先移动自身的50%,再通过transform属性移动自身的-50%,修正后的top、left就正好处于容器内的垂直居中位置了。

##5.flex方法
IE依然不支持CSS3是必须的,遗憾的是Safari依然是不支持的,至少在我测试阶段(5.1.7)window下还没有支持。
```
    #box{
		width:800px;
		height:800px;
		display:flex;
		border:1px solid palevioletred;
	}
	#box div{
		width:200px;
		height:200px;
		border:1px solid #2189BF;
		flex-direction:row;
		justify-content:center;
		align-self: center;
	}
	
    <div id="box">
		<div></div>
		<div></div>
		<div></div>
		<div></div>
		<div></div>
	</div>
```
flex针对垂直居中的方法比较直接,justify-content:center; align-self: center;这两个属性直接设置为center就可以了,在不支持CSS3的浏览器中是无法实现的,这也是flex的暂时的缺点之一。
