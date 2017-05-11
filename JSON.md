# JSON

## 1.语法
JSON可以表示以下的三种类型值：

 1. 简单值：Boolean、字符串、null、数组。但JSON不支持JS中的特殊值undefined。
 2. 对象：每个键值对的值可以是简单值,也可以是复杂数据类型的值。
 3. 数组：数组的值也可以是任意类型---简单值、对象、数组。

        JSON不支持变量,函数或对象实例。

### ①.简单值
JS字符串与JSON字符串的最大区别在于JSON字符串必须使用""(双引号),使用''(单引号)会因其语法错误。布尔值和null也是有效的JSON格式。
### ②.对象
与JS对象不同的是,JSON对象不能使用声明变量,只能使用双引号,没有末尾的分号。例子：
```
    {
        "name": "Nicholas",
        "age" :28,
        "school": {
            "name":"Peking University"
        }
    }
```
### ③.数组
JSON也可以是一个对象数组,对象和数组可以作为JSON的数据结构的最外层形式。例如：
```
var books =  [
    {
        "title":"Professional JavaScript",
        "authors":[
            "Nicholas C.zacks"
        ]
    },
    {
        "edition": 20,
        "year" : 2019 
    },
    {
        "edition": 20,
        "year" : 2017 
    }
]
```
## 2.解析与序列化
假设把以上的数据解析后得到的对象存在变量books中,和JS对象类似,可以通过这样的方式获取数据:
```
    books[0].title;//"Professional JavaScript"
```
### 1.JSON对象
JSON对象有两种方法,stringify()和parse(),分别为将原生的JS对象转换为JSON字符串,将JSON字符串解析为原生JS值。

在序列化JS对象时,所有的函数和原型成员都会被忽略,此外值为undefined的任何属性也会被跳过,有一点需要注意的是,如果undefined作为数组值时,转换后会返回null。NaN、Infinity和-Infinity，不论在数组还是非数组的对象中,都被转化为null。而parse()时,如果传进去的不是有效的JSON,该方法会抛出错误。

### 2.序列化选项
JSON.stringify()除了必选的参数,即序列化对象外,还有两个可选的参数,分别是过滤器和缩进符参数。先考虑其中的第一个：
#### ①.过滤结果
1)过滤器是一个数组。会按照数组中的属性进行筛选,如果JSON对象中不包含这些属性,则返回(过滤器.length)个空对象。
```
    var json = JSON.stringify(books,["title","name"]);
    console.log(json);//[{"title":"Professional JavaScript"},{},{}]
```
2)过滤器是函数。传入的函数接收两个参数key、value,函数的参数只能是字符串,当值不是键值对结构的值时,键名可以是空字符串例子。如下：
```
    var JsonText = JSON.stringify(books,function(key,value){
        switch(key){
            case "authors":
                return value.join(",")
                
            case "year":
                return 4000;
                
            case "edition":
                return undefined;//序列化的过程中会将此项忽略
            
            default:
                return value;
        }
    })
```
在最后一定提供default项,此时返回的是传入的值,这样可以保证其它的值能够出现在结果中。
#### ②.字符串缩进
这是stringify()方法中的第三个参数,控制结果中的缩进或空白符。缩进参数如果是字符串则按照其作为缩进字符使用,如果是数值则按照对应的数值缩进。最大缩进空格数为10。建议不要使用,否则违背了JSON设计以轻便为目的的初衷。

#### ③.toJSON()方法
给对象定义toJSON方法可以返回其自身的JSON数据格式,原生Date对象有toJSON()方法,返回的是ISO  8601日期字符串,可以为任何对象添加toJSON方法,例如：

```
    var book = {
        "title":"Professional JavaScript",
        "authors":[
            "Nicholas C.zacks"
        ],
        toJSON:function(){
            return this.title;
        }
    }
    var json = JSON.stringify(book);
    console.log(json);//"Professional JavaScript"
```
toJSON在这里的用处就是把title转换为JSON格式,然后再用stringify方法转换后(相当于没转)返回。如果加上一个函数作为参数,那么返回的就是undefined。toJSON可以返回任何值,他并不会受到stringify转换规则的限制,如果toJSON是顶级对象可以直接返回undefined,如果是嵌套的结构可以返回null。

嵌套时的转换结果,如下所示：
```
    var book = [
            {"title":"Professional JavaScript"}
            ,{"authors":[
                "Nicholas C.zacks"
            ]}
            ,{toJSON:function(){
                return undefined;
            }
        }
    ]
    
    var json = JSON.stringify(book,"a");
    console.log(json)//[{"title":"Professional JavaScript"},{"authors":["Nicholas C.zacks"]},null]
```
作为序列化方法的补充,可以让toJSON()返回任何值,比如可以让该方法返回undefined,如果它被包含在一个对象中,那么返回的是null,假设把一个对象传入JSON.stringify()中,序列化的顺序为：

 1. 如果存在toJSON()方法而且能取得返回值,则调用该方法。否则返回对象本身。
 2. 如果有第二个参数,应用这个过滤器,传入的值是第(1)步返回的值。
 3. 对第(2)步返回的值进行相应的序列化。
 4. 如果有第三个参数,执行相应的格式化。
 
### 3.解析JSON
该方法将JSON对象转换为JS对象,同样也可以接受第二个参数,参数被称为还原函数。如果还原函数返回undefined,则表示要从相应的键中删除
```
var book = {
            "title":"Professional JavaScript",
            "authors":[
                "Nicholas C.zacks"
            ],
            release : new Date(2017)
        }
        var Jsontext = JSON.stringify(book)
        var bookCopy = JSON.parse(Jsontext,function(key,value){
            if(key == "release"){//对象包含时改为key == "toJSON"
                return undefined;
            }else{
                return value;
            }

        })
        console.log(bookCopy);//title:...author:...
```

新发现Tips：
        
    而如果在JSON对象中有toJSON方法,在第一种情况下,即toJSON是顶级对象时,这个解析是无法进行的,即使是在解析函数中将键名(toJSON)返回值设为undefined也同样无效。在toJSON被包在对象里面时转换结果该项返回null。
    
```
    var book = {
        "title":"Professional JavaScript"
        ,"authors":[
            "Nicholas C.zacks"
        ]
        ,toJSON:function(){
            return undefined;
        }
    }
        
    var jsonText = JSON.stringify(book);
    var bookCopy = JSON.parse(jsonText);
    console.log(bookCopy);//Uncaught SyntaxError: Unexpected token u in JSON at position 0
    at JSON.parse (<anonymous>)
        
    //或者是数组对象中的嵌套toJSON   
    var book = [
            {"title":"Professional JavaScript"}
            ,{"authors":[
                "Nicholas C.zacks"
            ]}
            ,{toJSON:function(){
                return undefined;
            }
        }
    ]
    
    var jsonText = JSON.stringify(book);
    var bookCopy = JSON.parse(jsonText);
    console.log(bookCopy);//array:3  object,object,null
```
    
    JSON的stringify和parse,toJSON在<=IE7的IE下是不支持的。