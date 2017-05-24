---
title: javascript老生常谈之闭包
tags: javascript
---



# 前文：假设页面上有 5 个 div 节点,我们通过循环来给每个 div绑定 onclick 事件,按照索引顺序,点击第 1 个 div 时弹出 0,点击第 2 个 div 时出 1,以此类推。

```js
var nodes = document.getElementsByTagName( 'div' );

for(var i=0,len=nodes.length;i<len;i++){ 

    nodes[ i ].onclick = function(){
        alert(i); 
    }

};
```
测试这段代码会发现,无论点击哪个 div,最后弹出的结果都是 5。
这是因为 div 节点的 onclick 事件是被异步触发的,当事件被触发的时候,for循环早已结束,此时 i 的值已经是 5,
所以在 div 的 onclick 事件函数中顺着作用域链从内到外查找变量 i 时,查找到的值总是 5。
解决方法是在闭包的帮助下,每次循环的 i 值都封闭起来。当在事件函数中顺着作用域链从内到外查找变量 i 时,会先找到被封闭在闭包环境中的 i,
如果有 5 个 div,这里的 i 分别 是 0,1,2,3,4
```js
for(var i=0,len=nodes.length;i<len;i++){ 

    (function( i ){
        nodes[ i ].onclick = function(){ 
            console.log(i);
        } 
    })(i)

};
```
当然，你也可以把循环内的变量声明关键字var改为let，不过这是es6的解决方法不在本文探讨范围内O(∩_∩)O~。

下面进入正文。

## 一、了解闭包须知：

闭包的形成跟变量的作用域以及变量的生存周期密切相关。

变量的作用域,是指变量的有效范围。
当在函数中声明一个变量的时候,如果该变量前面没有带上关键字 var,这个变量就会成为全局变量 ,这是一种很容易造成命名冲突的做法。
另外一种情况是用 var 关键字在函数中声明变量,这时候的变量即是局部变量,只有在该函数内部才能访问到这个变量,在函数外面是访问不到的。

变量的搜索是从内到外而非从外到内的。
当函数被调用时：

先创建一个执行环境(execution context),及相应的作用域链；

将arguments和其他命名参数的值添加到函数的活动对象(activation object)

作用域链：当前函数的活动对象优先级最高，外部函数的活动对象次之，外部函数的外部函数的活动对象依次递减，直至作用域链的末端--全局作用域。优先级就是变量查找的先后顺序。

你可以在Chrome浏览器Console控制台运行下面的代码看看运行结果。

```js
var func=function(){

    var msg = "hello";
    console.log(msg); //输出:hello 

};

func();

console.log ( msg ); // Uncaught ReferenceError: msg is not defined
```

```js
var num1 = 1;

var func1 = function(){ 

    var num2 = 2;
    var func2 = function(){ 
        var num3 = 3;
        console.log ( num1 ); // 输出:1 
        console.log ( num2 );// 输出:2
    }
    func2();
    console.log( num3 );//输出:Uncaught ReferenceError: num3 is not defined

}; 

func1();
```
对于全局变量来说,全局变量的生存周期当然是的永久,除非我们主动销毁这个全局变量。
而对于在函数内用 var 关键字声明的局部变量来说,当退出函数时,这些局部变量即失去了 它们的价值,它们都会随着函数的调用的结束而销毁。

## 二.什么是闭包

闭包是指有权访问另一个函数作用域中变量的函数 --《JS高级程序设计第三版》
函数对象可以通过作用域链相关联起来，函数体内部的变量都可以保存在函数作用域内，这种特性称为 ‘闭包’ 。 --《JS权威指南》
内部函数可以访问定义它们的外部函数的参数和变量(除了this和arguments)。 --《JS语言精粹》

根据上面的描述我们来创建个简单的闭包：
```js
var sayName = function() {

    var name = 'shana';
    return function() {
        alert(name);
    }

};

var say = sayName(); 

say();
```
上面的例子简单说明一下：
var say = sayName() ：返回了一个匿名的内部函数保存在变量say中，并且引用了外部函数的变量name，由于垃圾回收机制，sayName函数执行完毕后，变量name并没有被销毁。

say() ：执行返回的内部函数，依然能访问变量name,输出 'shana' .

## 三.为什么使用闭包

1.封装变量—-闭包可以帮助一些不需要暴露在全局的变量封装成“私有变量”。
2.延长变量的生存周期，可以让一个变量常驻内存。
3.避免全局变量的污染。

```js
var add = (function() {

    var  num = 0;
    return function() {
        num++;
        console.log(num);
    }

})();

console.log(num); //undefined

add();

add();
```

## 四.使用闭包需要注意的一些问题

1.因为闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。
2.使用闭包要小心循环引用，不要造成死循环，引起内存泄露。
举个例子：A对象包含一个指向B的指针，对象B也包含一个指向A的引用。 这就可能造成大量内存得不到回收（内存泄露），因为它们的引用次数永远不可能是 0 。

## 五.补充：

内存泄露及解决方案

说到内存管理，自然离不开JS中的垃圾回收机制，有两种策略来实现垃圾回收：标记清除 和 引用计数；

标记清除：垃圾收集器在运行的时候会给存储在内存中的所有变量都加上标记，然后，它会去掉环境中的变量的标记和被环境中的变量引用的变量的标记，此后，如果变量再被标记则表示此变量准备被删除。 2008年为止，IE，Firefox，opera，chrome，Safari的javascript都用使用了该方式；

引用计数：跟踪记录每个值被引用的次数，当声明一个变量并将一个引用类型的值赋给该变量时，这个值的引用次数就是1，如果这个值再被赋值给另一个变量，则引用次数加1。相反，如果一个变量脱离了该值的引用，则该值引用次数减1，当次数为0时，就会等待垃圾收集器的回收。

这个方式存在一个比较大的问题就是循环引用，就是说A对象包含一个指向B的指针，对象B也包含一个指向A的引用。 这就可能造成大量内存得不到回收（内存泄露），因为它们的引用次数永远不可能是 0 。早期的IE版本里（ie4-ie6）采用是计数的垃圾回收机制，闭包导致内存泄露的一个原因就是这个算法的一个缺陷。

我们知道，IE中有一部分对象并不是原生额javascript对象，例如，BOM和DOM中的对象就是以COM对象的形式实现的，而COM对象的垃圾回收机制采用的就是引用计数。因此，虽然IE的javascript引擎采用的是标记清除策略，但是访问COM对象依然是基于引用计数的，因此只要在IE中设计COM对象就会存在循环引用的问题！

举个栗子：
```js
window.onload = function(){

    var el = document.getElementById("id");
    el.onclick = function(){
        alert(el.id);
    }

}
```
这段代码为什么会造成内存泄露？
```js
el.onclick= function () {

    alert(el.id);

};
```
执行这段代码的时候，将匿名函数对象赋值给el的onclick属性；然后匿名函数内部又引用了el对象，存在循环引用，所以不能被回收；

解决方法：
```js
window.onload = function(){

    var el = document.getElementById("id");
    var id = el.id; //解除循环引用
    el.onclick = function(){
        alert(id); 
    }
    el = null; // 将闭包引用的外部函数中活动对象清除

}
```



到此本文结束，如果还有什么疑问或者建议，可以多多交流。