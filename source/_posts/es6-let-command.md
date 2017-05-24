---
title: es6之关键字let
tags: javascript
---

前文：在前面闭包一文有提到：

假设页面上有 5 个 div 节点,我们通过循环来给每个 div绑定 onclick 事件,按照索引顺序,点击第 1 个 div 时弹出 0,点击第 2 个 div 时出 1,以此类推。

我们用闭包实现了这个效果，同时我们提到了可以使用es6的let命令实现一样的效果。所以本文将介绍一下es6的let命令。

## 什么是let命令

`let`命令是ES6新增的关键字，它用来声明变量。用法类似于`var`，但是所声明的变量，只在`let`命令所在的代码块内有效。

```
{
  let num1 = 0;
  var num2 = 1;
}

num1 // ReferenceError: num1 is not defined.
num2 // 1
```

上面代码在代码块之中，分别用`let`和`var`声明了两个变量。然后在代码块之外调用这两个变量，结果`let`声明的变量报错，`var`声明的变量返回了正确的值。这表明，`let`声明的变量只在它所在的代码块有效。

`for`循环的计数器，就很合适使用`let`命令。

```
for (let i = 0; i < 10; i++) {}

console.log(i);
//ReferenceError: i is not defined
```

上面代码中，计数器`i`只在`for`循环体内有效，在循环体外引用就会报错。

下面的代码如果使用`var`，最后输出的是`10`。

```
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

```

上面代码中，变量`i`是`var`声明的，在全局范围内都有效，所以全局只有一个变量`i`。每一次循环，变量`i`的值都会发生改变，而循环内被赋给数组`a`的`function`在运行时，会通过闭包读到这同一个变量`i`，导致最后输出的是最后一轮的`i`的值，也就是10。

而如果使用`let`，声明的变量仅在块级作用域内有效，最后输出的是6。

```
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6

```

上面代码中，变量`i`是`let`声明的，当前的`i`只在本轮循环有效，所以每一次循环的`i`其实都是一个新的变量，所以最后输出的是`6`。你可能会问，如果每一轮循环的变量`i`都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量`i`时，就在上一轮循环的基础上进行计算。

另外，`for`循环还有一个特别之处，就是循环语句部分是一个父作用域，而循环体内部是一个单独的子作用域。

```
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc

```

上面代码输出了3次`abc`，这表明函数内部的变量`i`和外部的变量`i`是分离的。

## 使用let命令带来的影响

### 1.不存在变量提升

### 2.TDZ(暂时性死区)

### 3.无法重复声明变量


### 无法变量提升

我们知道在javascript（ES5）中只有全局作用域和函数作用域，并不存在块级作用域，使用`var`命令会发生所谓的”变量提升“现象，即变量可以在声明之前使用，值为`undefined`。

**变量提升：函数声明和变量声明总是被JavaScript解释器隐式地提升(hoist)到包含他们的作用域的最顶端。**

这种变量提升会带来一些意想不到的结果，比如函数内层的变量可能会覆盖函数外层的变量，在一个循环中用来计数的循环变量泄露为全局变量。

```
// var 的情况
console.log(msg); // undefined
var msg = 'hello';

//发生了变量提升,上面的代码等价于下面的代码
var msg ;
console.log(msg); // msg没有赋值，输出undefined
msg = 'hello';

// let 的情况，暂时性死区，无法变量提升
console.log(tip); // 变量tip不存在，报错，Uncaught ReferenceError: tip is not defined
let tip = 'ok';
```



```
var tip = 'success';

function updateTip() {
  console.log(tip);
  if (false) {
    var tip = 'error';
  }
}

updateTip(); // undefined
```

上面代码的原意是，`if`代码块的外部使用外层的`tip`变量，内部使用内层的`tip`变量。但是，函数`updateTip`执行后，输出结果为`undefined`，原因在于变量提升，导致内层的`tip`变量覆盖了外层的`tip`变量。上面的代码等价于下面的代码：

```
var tip = 'success';

function updateTip() {
  var tip;
  console.log(tip);
  if (false) {
    tip = 'error';
  }
}

updateTip(); // undefined
```



```
var msg = 'hello';
var msg2 ='hello world';
for (var i = 0; i < msg.length; i++) {
  console.log(msg[i]);
}
console.log(i); // 5
/*
 *中间这里省略一大段代码
 **/
for (; i < msg2.length; i++) {
  console.log(msg2[i]);
}
console.log(i); // 11
```

上面代码中，变量`i`只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。后面还有另一个循环，循环变量同样是i，这时原本要输出的'hello world'就只输出了'world'，因为没有重新给变量赋值导致了与期待结果不一致。

### 暂时性死区

只要块级作用域内存在`let`命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。

ES6明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

**暂时性死区（temporal dead zone，简称 TDZ）：只要一进入当前作用域，所要使用的变量就已经存在，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。**

```
var input = 1;

if (true) {
  input = 'oh'; // Uncaught ReferenceError: input is not defined
  let input;
}
```

上面代码中，存在全局变量`input`，但是块级作用域内`let`又声明了一个局部变量`input`，导致后者绑定这个块级作用域，所以在`let`声明变量前，对`input`赋值会报错。

```
if (true) {
  // TDZ开始
  tmp = '0'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // tmp变量使用let命令声明，当前作用域下该声明变量到此前的所有代码块无法赋值和使用，形成TDZ
  //TDZ结束
  console.log(tmp); // undefined

  tmp = 1;
  console.log(tmp); // 1
}
```

上面代码中，在`let`命令声明变量`tmp`之前，都属于变量`tmp`的“死区”。

“暂时性死区”也意味着`typeof`不再是一个百分之百安全的操作。

```
typeof x; // ReferenceError
let x;

```

上面代码中，变量`x`使用`let`命令声明，所以在声明之前，都属于`x`的“死区”，只要用到该变量就会报错。因此，`typeof`运行时就会抛出一个`ReferenceError`。

作为比较，如果一个变量根本没有被声明，使用`typeof`反而不会报错。

```
typeof undeclared_variable // "undefined"

```

上面代码中，`undeclared_variable`是一个不存在的变量名，结果返回“undefined”。所以，在没有`let`之前，`typeof`运算符是百分之百安全的，永远不会报错。现在这一点不成立了。这样的设计是为了让大家养成良好的编程习惯，变量一定要在声明之后使用，否则就报错。

有些“死区”比较隐蔽，不太容易发现。

```
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 报错

```

上面代码中，调用`bar`函数之所以报错（某些实现可能不报错），是因为参数`x`默认值等于另一个参数`y`，而此时`y`还没有声明，属于”死区“。如果`y`的默认值是`x`，就不会报错，因为此时`x`已经声明了。

```
function bar(x = 2, y = x) {
  return [x, y];
}
bar(); // [2, 2]

```

另外，下面的代码也会报错，与`var`的行为不同。

```
// 不报错
var x = x;

// 报错
let x = x;
// ReferenceError: x is not defined

```

上面代码报错，也是因为暂时性死区。使用`let`声明变量时，只要变量在还没有声明完成前使用，就会报错。上面这行就属于这个情况，在变量`x`的声明语句还没有执行完成前，就去取`x`的值，导致报错”x 未定义“。

ES6 规定暂时性死区和`let`、`const`语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量，从而导致意料之外的行为。这样的错误在 ES5 是很常见的，现在有了这种规定，避免此类错误就很容易了。

### 不允许重复声明

`let`不允许在相同作用域内，重复声明同一个变量。

```
// 报错
function a() {
  let flag = true;
  var flag = false;
}

// 报错
function b() {
  let status = 'ok';
  let status = 'no';
}
```

因此，不能在函数内部重新声明参数。

```
function func(arg) {
  let arg; // 报错
}

function func(arg) {
  {
    let arg; // 不报错
  }
}
```

参考资料：

[(阮一峰)ECMAScript 6 入门 --let 和 const 命令](http://es6.ruanyifeng.com/#docs/let)

更多关于ES6的知识可以去看阮一峰老师的[ECMAScript 6 入门](http://es6.ruanyifeng.com/)

