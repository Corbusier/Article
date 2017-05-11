#DOM 扩展
    
## 1. 选择符API

    核心的两个方法是querySelector()及querySelectorAll()。
    
### ①.querySelector( )
接受一个CSS选择符,返回该模式匹配的第一个元素,如果没有匹配到则为null。

### ②.querySelectorAll( )
返回的是匹配的所有元素,而不是一个,这个方法返回的是NodeList的实例。其底层实现并不是不断对文档进行动态查询,而仅仅是类似于一组元素的快照。所以这两种方法其实都是静态方法,而非动态方法。

## 2. 元素遍历
对于元素的空格,除IE9之前版本以外,大部分浏览器都会解析空白符作为一个节点,为了弥补差异添加了以下5个属性:

1.childElementCount:返回子元素(不包括文本节点和注释)的个数。
2.firstElementChild:指向第一个子元素;firstChild的元素版;
3.lastElementChild:指向最后一个子元素;lastChild的元素版;
4.previousElementSibling:指向前一个兄弟元素;
5.nextElementSibling:指向后一个兄弟元素;
 
## 3. HTML 5
### ①.与类相关的扩充
#### (1).getElementsByClassName()
该方法接收一个参数,包含了一个或多个类名的字符串,返回带有指定类的所有元素的NodeList。传入多个类名时,次序并不重要。因为返回的是NodeList对象,所以也会对文档进行不断地查询,这样会带来定能的问题。

#### (2).classList属性
操作类名时,需要对类名进行添加删除或替换,因为类名是一个字符串,所以即使修改一小部分,也需要每次都设置整个字符串值,例子:
```
    <div class="bd user disabled"></div>
    //元素获取
    var classNames = div.className.split(/\s+/);
    var len = classNames.length;
    for(var i = 0;i<len;i++){
        if(classNames[i] == "user"){
            break;
        }
    }
    classNames.splice(i,1);
    div.className = classNames.join(" ");
```
HTML 5新增了一种操作类名的方式,就是为所有元素添加classList属性,该属性是新集合类型DOMTokenList的实例,同样是一个类数组。除了item()和方括号索引值外,还有以下的方法:

 - add(value)。将给定的字符串添加进列表,存在则不添加。
 - contains(value)。判断列表中是否存在给定的值,存在返回true,否则为false。
 - remove(value)。从列表中删除给定字符串。
 - toggle(value)。如果已经存在给定字符串则删除,如果没有则添加。

之前的例子可以改写为:
```
    div.classList.remove("user");
```

### ②.焦点管理
 document.activeElement属性,会始终引用DOM中获得焦点的元素。元素获得焦点的方式有页面重载、用户输入和在代码中调用focus()方法。

```
    var button = document.getElementById("myButton");
    button.focus();
    alert(document.activeElement === button);//true
    
```
在默认情况下,文档刚加载完成时,该属性保存的是document.body元素的引用,文档加载期间,该属性保存的值是null。
还有就是新增了检测是否获得焦点的方法。
```
    button.focus();
    alert(button.hasFocus());
```

### ③.HTMLDocument的变化
#### (1).readyState属性
Document的readyState属性有两个可能的值:

 - loading,正在加载文档;
 - complete,已经加载完文档;

#### (2).兼容模式
如果需要检测浏览器的渲染模式,可以使用document的compatMode属性,标准模式下,该值为CSS1Compat,而混杂模式下为BackCompat。
```
    if(document.compatMode == "CSS1Compat"){
        alert(1);
    }else{
        alert(2);
    }
```
### ④.自定义数据属性
可以为元素添加的非标准属性,但要添加前缀data-,目的是为元素提供与渲染无关的信息,或者提供语义信息。可以通过元素的dataset来访问自定义属性值。
```
    <div id="myDiv" data-Age="25" data-jim-ol="eq"></div>
	console.log(myDiv.dataset.age);
	console.log(myDiv.dataset.jimOl);	
```

以上的例子看出,不管是大写还是小写,浏览器都会转换为全小写,所以获取时需要写成小写,因此建议自定义属性全小写,而如果属性名很长,在获取时则采用驼峰的方式获取。

### ⑤.插入标记
#### (1).innerHTML属性
在读模式下,各浏览器返回的innerHTML不尽相同,在IE8以下的IE中返回的是全大写,其余的都是DOM格式的代码。

在写模式下,属性的值会被解析为DOM子树,使用该属性也会有一些限制,在多数浏览器中通过innerHTML插入script元素,并不会执行其中的脚本。IE8以前的浏览器是唯一能突破这种限制的浏览器,但必须满足一些条件。

 - 必须为script元素指定defer属性。
 - 在script前添加一个元素,可以是文本节点,也可以是一个没有结束标签的元素如input。

```
    div.innerHTML = "_<script defer>alert("hi")<\/script>";
    div.innerHTML = "<div>&nbsp;</div>_<script defer>alert("hi")<\/script>";
    div.innerHTML = "<input><script defer>alert("hi")<\/script>";
```
第一行代码会在script元素前插入一个文本节点,在之后需要将其移除。
第二行代码则加入了一个非换行空格的div元素,如果是空div也是不行的。
第三行代码加入了隐藏的input标签。

大多数浏览器都支持以直观的方式通过HTML插入style元素,如:
```
    div.innerHTML = "<style>body{background-color:red;}</style>";
```
在IE8以前的版本中依然要使用script标签插入的方法。
并不是所有的元素都支持innerHTML属性,不支持该属性的有col、colgroup、frameset、head、html、style、表格的所有元素。

#### (2).outerHTML属性
在读模式下,它会返回包括元素本身在内的所以DOM格式的代码。而在写模式下会根据字符串建立DOM树,并用DOM树完全替换调用元素。
```
    div.outerHTML = "<p>This is a paragraph.</p>";
    //等价于
    var p = document.createElement("p");
    p.appendChild(document.createTextNode("This is a paragraph."));
    div.parentNode.replaceChild(p,div);
```

#### (3).内存与性能问题
在使用之前所述的某些属性将元素移除出DOM,事件处理程序与元素的绑定关系并没有在内存中一并移除。如果这种情况频繁出现,就会造成内存的明显增加。设置innerHTML或outerHTML时使用的是浏览器级别的代码,使用的是HTML解析器,创建和销毁解析器也会带来性能损失。尽量控制少用这个机制,可以使用如下的办法创建:
```
    var str = "";
    for(var i = 0;i<value.length;i++){
        str += "<li> + values[i] + </li>"
    }   
    ul.innerHTML = str;
```
这样做就可以少访问innerHTML。

## 4.专有扩展
### ①.children属性
这个属性是HTMLcollection的实例,只包含元素中的子节点,不包含空白符。
### ②.contains()方法
不通过DOM文档树查找获取节点信息,判断节点是不是另一个节点的后代。调用该方法的应该是祖先节点,该方法接收一个参数,即要检测的后代节点。
```
    alert(document.documentElement.contains(document.body));//true
```

DOM Level3中还有一种方法也能确定节点间的关系,compareDocumentPosition( ),返回的是该关系的位掩码。该方法接收的参数也是子节点,调用方法的是祖先节点。
|   掩码    |   节点关系  |
| --------  | ------------|
| 1         |     无关        |
| 2         |     居前        |
| 4         |     居后        |
| 8         |     包含        |
| 16        |     被包含      |

如果检测节点属于被包含关系,那么会返回20(掩码4+掩码16),再执行按位操作会返回非零数值,再转换为布尔值true、false即可判断包含或不包含关系。例子:
```
    var result = document.documentElement.compareDocumentPosition(document.body);
    alert(!!(result & 16));//true
```
如果掩码的结果小于16,那么再执行按位操作就会返回负数,转换布尔值的结果为false,说明节点与另一个节点并非包含关系。