# DOM

## 1.节点层次

DOM可以将任何HTML或XML文档描绘成一个或多层节点构成的结构,节点分为几种不同的类型,每种类型分别代表了不同的信息和标记,每个节点都有各自的特点、数据和方法。文档节点是每个文档的根节点。文档节点只有一个子节点,即 html 元素,我们称为文档元素,每个文档只能有一个文档元素,在HTML中文档元素始终是 html 元素。在XML中没有预定义的元素,因此任何元素都可能成为文档元素。每一段标记都可以通过树中的一个节点来表示,总共有12种节点类型,这些类型都继承自一个基类型。

特别注意：

IE中的所有DOM对象都是以COM对象的形式实现的,所以IE中的DOM对象与原生的DOM对象的活动特点和行为并不一致。
    
### 1. Node类型

JavaScript中的所有节点类型都继承自Node类型,因此所有节点类型都共享着相同的基本属性和方法。每个节点都有一个nodeType属性,用于表现节点的类型,然而IE并没有公开Node类型的构造函数,最好将nodeType属性与数字值进行比较。
```
    if(someNode.nodeType == 1){
        alert("Node is an element");
    }
```
#### ①.nodeName和nodeValue属性

这个两个属性的值完全取决于节点的类型,在使用之前最好先检测节点的类型。例如:
```
    if(someNode.nodeType == 1){
        value = someNode.nodeName;
    };
```
元素节点的nodeName一直是元素的标签名,而nodeValue则始终为null。

#### ②.节点关系

节点与节点之间有传统的家族关系,每个节点都有一个childNodes属性,其中保存着一个NodeList对象,这是一个类数组对象,用于保存一组有序列的节点,可以使用方括号或item()方法访问,并且也有length属性,但并不是Array实例。它的独特之处在于DOM结构的查询结果是动态的,DOM的变化能实时得反映到NodeList对象中。
        
如果需要把DOM节点的NodeList转换为数组,非IE浏览器可以使用Array原型上的方法将其转换,而IE浏览器需要手动的循环它然后返回。如下:
```
    function convertArray(nodes){
        var array = null;
        try{
            array = Array.prototype.slice.call(nodes,0);
        }catch(ex){
            for(var i = 0;i<nodes.length;i++){
                array = new Array();
                array.push(nodes[i]);
            }
        }
        return array;
    }
```

每个节点都有一个parentNode属性,而父节点以下的子节点可以成为兄弟节点,他们之间可以使用previousSibling和nextSibling属性访问兄弟节点之间的其他节点,如果没有则为null。最后一个所有节点都有的属性是ownerDocument,该属性指向表示整个文档的文档节点,这种关系表示的是任何节点都属于它所在的文档,通过这个属性我们可以直接访问到文档节点Document。

#### ③.操作节点

replaceChild()、removeChild()、insertBefore()、appendChild()方法,这四种方法都必须要取得父节点,操作的都是某个节点的子节点,但是并不是所有类型的节点都有子节点,如果在不支持子节点的节点上调用了这些方法将会导致错误。

1)appendChild()
用于向childNodes列表的末尾添加一个节点,如果传递给该方法的节点已经是文档的一部分了,那么它将会从原位转移到新位置上。
```
    var return = someNode.appendChild(someNode.firstChild);
    alert(someNode.lastChild == return);//true
```
2)insertBefore()

如果节点需要插入某个位置而不是直接添加到某位,那么可以使用该方法,接受两个参数,分别是需要插入的节点,参照节点。如果参照节点为null,那么结果与appendChild()相同。

3)replaceChild()
以上的两种方法都只是插入节点,并不会移除节点,该方法会插入并移除节点,接受两个参数为,要插入的节点,要替换的节点。尽管从根本上说被移除的节点还在文档中,但是已经没有它的位置了。

4)removeChild()
如果只做移除并不替换,可以采用该法。接收一个参数为被移除的项。

#### ④.其他方法
cloneNode()接收一个布尔值参数,分为深复制或浅复制。深复制包括了节点以及子节点树。浅复制只包含该节点。复制后返回的节点副本归文档所有,并没有为它指定父节点,除非使用appendChild()、replaceChild()、inserBefore()方法添加到文档中。
        
clone只会复制特性、子节点,其他都不会复制,但是IE有一个Bug,该方法会复制事件处理程序,所以建议在复制之前移除事件处理程序。还有一点要注意的是,childNodes()包含的有空白处理符,而children[],只含有元素节点。

### 2. Document类型

JavaScript通过Document类型表示文档,在浏览器中document对象是HTMLdocument(继承自Document类型)的实例。并且document对象是window对象的一个属性,因此可以使用全局对象访问。该节点就有以下的特征:

 - nodeType的值为9;
 - nodeName的值为"#document";
 - nodeValue的值为null;
 - parentNode的值为null;
 - ownerDocument的值为null;
 - 其子节点可能是一个DocumentType(最多一个),Element(html元素,最多一个),ProcessingInstruction或Comment。


#### 1)文档的子节点

DOM标准规定的Document节点的子节点可以是DocumentType(文档类型节点)、Element、ProcessingInstruction(处理命令节点)或Comment(主世界店)。还有两个内置的访问其子节点的快捷方式,分别是documentElemen属性,它始终指向 html元素 以及childNodes列表访问方法。通过第一种方法更快捷、迅速。作为HTMLDocument对象的实例,document还有一个body属性,直接指向body元素。

    所有浏览器都支持document.documentElement和document.body属性。

#### 2)文档信息
第一个是title属性,它是可读可写的。然后是三个与网页请求有关的属性,分别是URL、domain、referrer。URL包含完整的URL、domain只包含页面的域名、referrer保存的是连接到当前页面的那个页面的URL。这三个属性中只有domain是可写的,由于安全方面的限制,设置的值只能是URL中包含的域,否则会报错。如果两个来自不同URL,但有相同域名部分的互相引用的页面,出于安全限制,可能无法通信,将document.domain属性更改为这个字符串,就可以达到目标。

####3)查找元素
    Document类型提供了两种方法,分别为getElementById()和getElementsByTagName()。如果页面中有多个ID相同的元素,那么只会得到第一个,IE7以下中Id获取元素会有这样的bug,name特性的表单元素(input、textarea、button、select)如果与ID名相同,那么也会被返回。最好的办法就是让name属性与ID不同。
    第三种方法,也是只有HTMLDocument类型才有的方法,getElementsByName(),为了确保给浏览器发送的值准确无误,name属性都要相同,该方法可以获得所有name属性相同的单选按钮。
    
### 3. Element类型
Element类型提供了对元素标签名、子节点及特性的访问。Element节点具有以下的特征:

 - nodeType的值为1;
 - nodeName的值为元素的标签名;
 - nodeValue的值为null;
 - parentNode可能是Element、Text、Comment、ProcessingInstruction、CDATASection或EntityReference。

访问元素的标签名可以是tagName或nodeName,在HTML中实际输出的是全大写而不是小写,最好将其转换为全小写,这样更适用于HTML文档。可以再任何浏览器中通过脚本访问Element类型的构造函数及原型,包括IE8以前的版本。

#### ①.HTML元素
HTMLElement类型直接继承自Element并添加了一些属性,添加的属性分别对应于每个HTML元素中都存在的下列标准特性。

 - id,元素在文档中唯一标识符。
 - title,有关元素的附加说明信息。
 - lang,元素内容的语言代码。
 - dir,元素的方向,值为"ltr"(从左至右),或"rtl"(从右至左)。
 - className,与元素的class特性对应,即为元素指定的CSS类。
 
#### ②.取得特性
每个元素有一或多个特性,作用是给出相应元素或其内容的附加信息。操作特性的DOM方法有三个,分别是getAttribute()、setAttribute()、removeAttribute()。这些方法可以对任何特性使用,包括HTMLElement定义属性的特性。

传递给getAttribute()方法的特性名与实际的特性名一致,因此想得到class特性值应该传入"class"而不是"className"，后者只有通过对象访问特性时才会使用。
    
getAttribute()方法还可以获取行间自定义特性的值,任何元素得所有特性,也都可以用过DOM元素本身的属性访问,但是自定义属性不会添加到DOM对象中。
```
    <div id="MyDiv" align="left" data-special="hello"></div>
    alert(div.id);//"MyDiv"
    alert(div.align);//"left"
    alert(div.data-special);//undefined
```
有两个特性虽然有对应的属性名,但属性的值与getAttribute()返回的值并不相同。第一类就是style,用get返回的是CSS文本,通过属性访问则会返回对象。第二类是onclick这样的事件处理程序,在元素上使用时,该特性返回的是JS代码,而通过get方法返回的是相应代码的字符串。由于这些差别导致一般只是用对象属性操作DOM,只有在获取自定义特性值的情况下才会使用getAttribute()方法。

#### ③.设置特性
setAttribute()方法,第一个参数为需要设置的特性,特性的值。而不管是通过set还是DOM元素方法都无法使用get获取到,get只能获取行间的自定义属性。
#### ④.创建元素
使用document.createElement()方法创建新元素,只接受一个参数,那就是创建元素的标签名,标签名不区分大小写。同时还可以操作元素的特性,为他添加更多的子节点。然后再将创建的元素添加到文档树中,对该元素的修改都会反映到浏览器中。
在IE中可以用另一种方式使用该方法,传入完整的元素标签,创建一个完整的新元素。注意该方法只能在IE中使用:
```
    var div1 = document.createElement("<div class=\"lio\" id=\"myNewId\"></div>");
```
并且该方法还可以解决在IE7之前版本中的一些问题,

 - 不能动态创建一个带name特性的ifame元素;
 - input元素;
 - button元素;
 - 单选按钮;

#### ⑤.元素的子节点
元素可以有任意多个子节点和后代节点,元素的childNodes属性中包含了所有的子节点,包括了元素、文本节点、注释或处理指令。而且会把空白符解析为一个子节点,所以使用children代表子节点。

## DOM操作技术
### 1. 动态脚本

    指的是页面加载时不存在,但将来的某一时刻通过修改DOM动态添加的脚本。
```
    var script = document.createElement("script");
    script.type = "text/javascript";
    script.src = "client.js";
    document.body.appendChild(script);
```
这个过程即为外链JS文件加载的过程,在最后一步之前,是不会下载外链文件的。
动态加载外链的JS文件能够立即执行,另一种创建方式是行内方式。
###  2.动态样式

    和动态脚本类似,页面刚加载时不存在的样式,在加载完成后动态添加到页面中的。
###  3.操作表格
如果要使用DOM方法来创建以下的表格:
```
    <table border="1" width="100%">
		<tbody>
			<tr>
				<td>Cell 1,1</td>
				<td>Cell 2,1</td>
			</tr>
			<tr>
				<td>Cell 1,2</td>
				<td>Cell 2,2</td>
			</tr>
		</tbody>
	</table>
```
DOM方法：
```
    var table = document.createElement("table");
    table.border = 1;
    table.width = "100%";
    
    //创建tbody
    var tbody = document.createElement("tbody");
    table.appendChild(tbody);
    
    //创建第一行
    tbody.insertRow(0);
    tbody.rows[0].insertCell(0);
    tbody.rows[0].cells[0].appendChild(document.createTextNode("Cell 1,1"));
    tbody.rows[0].insertCell(1);
    tbody.rows[0].cells[1].appendChild(document.createTextNode("Cell 2,1"));
    
    //创建第二行
    tbody.insertRow(1);
    tbody.rows[1].insertCell(0);
    tbody.rows[1].cells[0].appendChild(document.createTextNode("Cell 1,2"));
    tbody.rows[1].insertCell(1);
    tbody.rows[1].cells[1].appendChild(document.createTextNode("Cell 2,2"));
	
	//将表格添加到文档中
	document.body.appendChild(table);
```
在创建第一行时,通过tbody元素调用了insertRow(),传入了参数表示应该将插入的行放在什么位置,执行完这一行代码之后就可以通过tbody.rows[0]来引入新插入的行。创建单元格的方法与此类似。

## 小结
DOM由各种节点构成,简要的总结:

 - 最基本的节点类型是Node,用于抽象的表示文档中的一个独立部分;所有其他类型都继承自Node。
 - Document类型表示整个文档,是一组分层节点的根节点。document对象是Document的实例,使用document对象,有很多方式可以查询和取得节点。
 - Element节点表示文档中的所有HTML或XML元素,可以用来操作这些元素的属性或特性。

DOM操作是程序中开销最大的一部分,因而访问NodeList导致的问题最多,每一次访问都要运行一次基于文档的查询,所以尽量少使用DOM操作。