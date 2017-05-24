---
title: javascript实现继承的几种思路
tags: javascript
---

本文主要总结一下js实现继承的几种思路。我们知道，在ES6之前，`javascript`是没有类的概念的，想要实现继承，主要是依靠原型链来模拟实现的。

进入正文前先了解下以下知识：

> 每个构造函数(constructor)都有一个原型对象(prototype),原型对象都包含一个指向构造函数的指针,而实例(instance)都包含一个指向原型对象的内部指针.

> 如果试图引用对象(实例instance)的某个属性,会首先在对象内部寻找该属性,直至找不到,然后才在该对象的原型(instance.prototype)里去找这个属性.

> 搜索轨迹: instance --> constructor.prototype…-->Object.prototype
>
> 1).首先会在instance内部属性中找一遍;
>
> 2).接着会在instance.__proto__(constructor.prototype)中寻找直至Object的原型对象
>

这种搜索轨迹,形似一条长链, 又因prototype在这个搜索过程中充当链接的作用,于是我们把这种实例与原型的链条称作 **原型链** .

## 类式继承

```javascript
function Animal (color) {
	this.name = 'animal';
	this.type = ['dog','cat'];
	this.color = color;
}

Animal.prototype.greet = function (sound) {
	console.log(sound);
}

function Cat () {
	this.name = 'cat';
}
//在实例化一个类时，新创建的对象复制了父类的构造函数内的属性与方法并且将原型__proto__指向了父类的原型对象，这样就拥有了父类的原型对象上的属性与方法。
Cat.prototype = new Animal('白色');

var cat = new Cat();
cat.greet('喵喵');
console.log(cat.type);
//类式继承方式缺点
//引用缺陷
cat.type.push('bird');
var cat2 = new Cat();
console.log(cat2.type);
//无法为不同的实例初始化继承来的属性
console.log(cat.color); // "白色"
console.log(cat2.color); // "白色"
```

## 构造函数继承

> 基本思路:即在子类型构造函数的内部调用超类型构造函数.

```javascript
/**
 * 构造函数继承方式可以避免类式继承的缺陷
 * 其一, 保证了原型链中引用类型值的独立,不再被所有实例共享;
 * 其二, 子类型创建时也能够向父类型传递参数.
 */

// 声明父类
function Animal(color) {
	this.name = 'animal';
	this.type = ['dog','cat'];
	this.color = color;
}

// 添加共有方法
Animal.prototype.greet = function(sound) {
	console.log(sound);
}

// 声明子类
function Cat(color) {
	Animal.apply(this, arguments);
}

var cat = new Cat('白色');
var cat2 = new Cat('黑色');

cat.type.push('bird');
console.log(cat.color);  // "白色"
console.log(cat.type);  // ["dog", "cat", "bird"]

console.log(cat2.type);  // ["dog", "cat"]
console.log(cat2.color);  // "黑色"

//构造函数继承也是有缺陷的，那就是方法都在构造函数中定义, 因此函数复用也就不可用了,
//对于子类而言，我们也无法获取到父类的共有方法，也就是通过原型prototype绑定的方法：
//cat.greet();  // Uncaught TypeError: cat.greet is not a function
```

## 组合继承

> 基本思路: 使用原型链实现对原型属性和方法的继承,通过借用构造函数来实现对实例属性的继承.
>
> 这样,既通过在原型上定义方法实现了函数复用,又能保证每个实例都有它自己的属性.

```javascript
// 声明父类   
function Animal(color) {    
	this.name = 'animal';    
	this.type = ['dog','cat'];    
	this.color = color;   
}     

// 添加共有方法  
Animal.prototype.greet = function(sound) {    
	 console.log(sound);   
}     

// 声明子类   
function Cat(color) { 
	// 构造函数继承    
	Animal.apply(this, arguments);   
}   

// 类式继承
cat.prototype = new Animal();   

var cat = new Cat('白色');   
var cat2 = new Cat('黑色');     

cat.type.push('bird');   
console.log(cat.color); // "白色"
console.log(cat.type);  // ["dog", "cat", "bird"]

console.log(cat2.type); // ["dog", "cat"]
console.log(cat2.color);  // "黑色"
Cat.greet('喵喵');  // "喵喵"

//组合继承最大的问题就是无论什么情况下,都会调用两次父类构造函数: 一次是在创建子类型原型的时候, 另一次是在子类型构造函数内部.
```
## 寄生组合继承

```javascript
//寄生组合式继承强化的部分就是在组合继承的基础上减少一次多余的调用父类的构造函数：	
function Animal(color) {
	this.color = color;
	this.name = 'animal';
	this.type = ['dog', 'cat'];
}

Animal.prototype.greet = function(sound) {
	console.log(sound);
}

function Cat(color) {
	Animal.apply(this, arguments);
	this.name = 'cat';
}

//如果不支持Object.create方法则模拟实现Object.create方法
/*
if (!Object.create) {
	Object.prototype.create = function (proto) {
		function F() {}
 		F.prototype = proto;
		return new F();
 	}
}
*/

/* 注意下面两行 */
//使用Object.create()进行一次浅拷贝，将父类原型上的方法拷贝后赋给Cat.prototype，这样子类上就能拥有了父类的共有方法，而且少了一次调用父类的构造函数
Cat.prototype = Object.create(Animal.prototype);
//由于对Animal的原型进行了拷贝后赋给Cat.prototype，因此Cat.prototype上的constructor属性也被重写了，所以我们要修复这一个问题
Cat.prototype.constructor = Cat;

Cat.prototype.getName = function() {
	  console.log(this.name);
}

var cat = new Cat('白色');   
var cat2 = new Cat('黑色');     

cat.type.push('bird');   
console.log(cat.color);   // "白色"
console.log(cat.type);   // ["dog", "cat", "bird"]

console.log(cat2.type);  // ["dog", "cat"]
console.log(cat2.color);  // "黑色"
cat.greet('喵喵');  //  "喵喵"
```
## es6实现继承

```javascript
class Animals {   
	constructor(color) {   
		this.color = color;   
	}   
	greet(sound) {   
	    console.log(sound);   
	}  
}   

class Cats extends Animals {   
	  constructor(color) {   
	  	/**
	  	 * 子类必须在constructor方法中调用super方法，否则新建实例时会报错。
	  	 * 这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。
	  	 * 如果不调用super方法，子类就得不到this对象
	  	 */
	    super(color);   
	    this.color = color;   
	  }  
}   

let cats = new Cats('黑色');  
cats.greet('喵喵');  // "喵喵"
console.log(cats.color); // "黑色"
```
以上就是javascript实现继承的几种思路，其中最常用的就是寄生组合继承了，而es6实现的继承最为简洁方便，不得不说es6大法好啊。本文如果描述有误，欢迎到 [issues](https://github.com/chenxiaohuan/chenxiaohuan.github.io/issues) 交流，本人会及时修正。