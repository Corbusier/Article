# ES6基础

## 变量声明方式let/const

主要区别在于块级作用域和不再具备变量提升的能力。
```
    {
        let a = 10;
    }
    console.log(a);//a is not defined
    //经过编译后===>
    
    "use strict";
    {
        var _a = 20;
    }
    
    console.log(a);
```
### 变量提升：
```
    // ES5
    console.log(a);   // undefined
    var a = 20;
    
    // ES6
    console.log(a); // a is not defined
    let a = 20; 
```
ES5中的是a声明但暂未定义,所以是undefined,而ES6中时没有声明,找不到a所以是not defined。

### const与let的区别
在ES6的语法中let和const的区别在于,let声明的是可以改变的变量,而const声明的是不可改变的变量,因此成为常量。当这个值为:

①.基础数据类型时,常量本身就不可以改变,也就是说这个基础类型值不可以被修改。
②.引用类型值时,只要这个引用不变,即整个应用类型没有被重写,引用内部的任何数据类型的值都可以修改。
```
    //基础类型比较：
    //Assignment to constant variable.
    const a = 1;
    a = 12;
    console.log(a)
    
    let a = null;
    a = 20;
    console.log(a)//20
    
    //引用类型比较
    const obDev = {
        a: 20,
        b:[1,9,29]
    }
    //修改的是常量内部的引用对象,常量本身的引用并没有变化
    obDev.b.push(2);
    console.log(obDev);
    //Object {a:20,Array(4)}
    
    //如果修改常量的引用对象
    //Assignment to constant variable.
    obDev = {c:2};
```
## 箭头函数(Arrow function)

### 定义的书写不同
```
    //ES5写法
    var f1 = function(a, b) {
      return a + b;
    };
    
    //ES6,当函数直接被return时，可以省略函数体的括号
    const f1 = (a,b) => a + b;
    console.log(f1(1,2))
    
    //ES5
    var foo = function() {
        var a = 20；
        var b = 30;
        return a + b;
    }
    
    //ES6
    const foo = () => {
       const a = 20;
       const b = 30;
       return a + b;
    }
```
### 没有this值
箭头函数中的this值继承外层的this,所以也无法用call/apply/bind来修改this指向。
```
    var person = {
        a(){
            console.log(this);
        }
    }
    person.a();//object:person
    //ES6的写法来重构上面的对象
    const person = {
        a: () =>{
            console.log(this);
        }
    }
    person.a();//window
```

## 模板字符串
解决使用 + 号拼接字符串的不便,使结果和过程清晰
```
    //ES6
    const a = 20;
    const b = 30;
    const string = `${a}+${b}=${a+b}`
    
    //ES5
    var a = 20;
    var b = 30;
    var string = a + "+" + b + "=" + (a + b);
    //使用${}包裹一个变量或表达式,``将整个字符串包裹起来,这个功能在需要拼接字符串生成html结构时非常直观,并且支持换行
    fake.style.cssText = `width: 10px;
    					  height: 10px;
    					  background: red;
    					  position: absolute;
    					  left:0;
    					  top:0;
    					  opacity:0;
    			  		 `;
```
## 解构赋值
ES6允许按照一定模式，从数组和对象(函数)中提取值，对变量进行赋值，这被称为解构（Destructuring）。
```
    let arr = [1,2,3,4];
    let [a,b,c] = arr;
    console.log(a,b,c);
    //1,2,3
    
    var arr = [1,4,9,13,17];
    let arr = [,a,b,c];
    console.log(a,b,c);
    /*4,9,13
    ,空缺的这一位会自动忽略，继续由后面的几位赋值
    假如超过了length,这一位为undefined*/
    
    const arr = [1, 2, 3];
    const [a, b, c] = arr;
    console.log(a)//1,2,3
    
    //函数参数
    function fn([age,name,sex="女"]){
        console.log(age,name,sex);
    }
    fn([27,"duoduo","男"]);
    fn([30,"momo"]);
    
    //给sex一个默认值，当没有设置sex时
```
实际应用中,比如阶乘函数的尾递归优化：

递归非常耗费内存，因为需要同时保存成千上百个调用记录，很容易发生"栈溢出"错误。但对于尾递归来说，由于只存在一个调用记录，所以永远不会发生"栈溢出"错误。
```
    function fac(n,total=1){
        if(n<=1) return total;
        return fac(n-1,n*total);
    }
```

## 扩展运算符
```
example：找出n个数组中的最大值
1）先用concat合并
2）Math.max方法
    普通的方法：
    var arr1 = [2,3,6,9];
    var arr2 = [5,4,8,10,29];
    var newArr = arr1.concat(arr2);
    console.log(Math.max.apply(null,newArr));
    
扩展运算符方法：
1)不使用concat,使用...arr1,...arr2将两个数组合并
2）Math.max方法
    //注意：这个方法不需要apply转换参数序列，...将一个数组转为用逗号分隔的参数序列
    var arr1 = [2,3,6,9];
    var arr2 = [5,4,8,10,29];
    var newArr = [...arr1,...arr2];
    console.log(Math.max(...newArr));
   
    //所有参数的求和
    const add = (a, b, ...more) => {
        return more.reduce((m, n) => m + n) + a + b
    }
    console.log(add(1,12,1,8,6,4,19));//51
```

## 对象字面量及class
比如属性与值同名时。
```
    const name = "Messi",
          age  = 28;
    //ES6
    const person = {
        name,
        age
    }
    //ES5
    var person = {
        name : name,
        age : age
    }
```
对象字面量的方法可以运用到模块化的提供对外接口时使用：
```
    const getName = () => person.name;
    const getAge = () => person.age;
    //commonJS的语法
    module.exports = {getName,getAge};
    //ES6 modules语法
    exports default{getName,getAge};
```

class语法糖
```
    //ES5面向对象的写法
    function Person(age,name){
        this.name = name;
        this.age = age;
    }
    //原型方法
    Person.prototype.getName = function(){
        return this.name;
    }
    
    //ES6 class语法糖
    class Person{
        constructor(name,age){
            this.name = name;
            this.age = age;
        }
        getName(){
            return this.name;
        }
    }
```
试试Babel转换为ES5写法？
http://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-2&targets=&browsers=&builtIns=false&debug=false&code=

此外,还应注意在语法糖中关于其他属性、方法的异同。
```
    class Person{
        constructor(name,age){
            this.name = name;
            this.age = age;
        }
        getName(){
            return this.name;
        }
        static a = 20;//Person.a 父类型的静态属性
        c = 20;//父类型的属性 this.c = 20;
    }
```
## 继承
```
    class Person{
        constructor(name,age){
            this.name = name;
            this.age = age;
        }
        getName(){
            return this.name
        }   
    }
    //Student类继承Person类
    class Student extends Person{
        constructor(name,age,gender,classes){
            super(name,age);
            this.gender = gender;
            this.classes = classes;
        }
        getGender(){
            return this.gender;
        }
    }
```
直接通过extends关键字实现类继承,不用像ES5那样担心构造函数继承和原型继承,除此之外还应该注意在Student构造函数上的super方法,这个和ES5中的call方法一样,使超类型在父类型上调用一次。

## for of循环
数组可以for of循环,而对象不可以
```
    var arr = [123,2,3,33,1];
    var obj = {
        a : "Messi",
        age : 25,
        arr : [
            "R","C"
        ]
    }
    for(var key of arr/obj){
        console.log(key)
    }
    //undefined is not a function(chrome)
```
for of 方法会调用Object原型上的Symbol iterator方法返回一个对象，再依次使用next方法返回对象 
当done = true时，for of才会停止，否则这个过程会一直进行。更改原型上的这个迭代接口就可以用for-of遍历出对象的键值
```
    Object.prototype[Symbol.iterator] = function(){
        let _this = this;
        let len = Object.keys(obj);
        let index = 0;
        return {
            next:function(){
                return index<len.length?{
                    value : _this[len[index++]],
                    done : false
                } : {
                    value : undefined,
                    done : true
                }
            }
        }
    }
```
