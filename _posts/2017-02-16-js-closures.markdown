---
layout: post
title:  "js中的闭包"
date:   2017-02-16 18:30:02 +0800
categories: javascript
---

### 词法作用域（Lexical scoping）

```
function init() {
  var name = 'Mozilla'; // name is a local variable created by init
  function displayName() { // displayName() is the inner function, a closure
    alert(name); // use variable declared in the parent function
  }
  displayName();
}
init();
```

以上是嵌套函数的例子，在js中，也存在词法作用域，它的含义是，当js查找一个变量的声明位置时，会根据它在源码中的使用位置去查找。
此处，内部函数displayName里面使用变量name，它会首先就近查找声明，如果没有，就去查找上层函数，结果在init中找到了此函数。

### 作用域链

在js中，每当我们定义一个函数的时候，其实就形成了一个作用域，当函数执行时，会从内向外逐层去寻找变量的声明，这就形成了一个作用域链。

### 闭包

```
function makeAdder(a) {
  return function(b) {
    return a + b;
  };
}
var x = makeAdder(5);
var y = makeAdder(20);
x(6); // return 11
y(7); // return 27
```

此处仍然是嵌套函数，然而与上面不同的是，此处并没有立即执行，而是返回了一个function，待随后执行。你也许会奇怪：函数的作用域中的局部变量，
不应该是在函数执行完之后就销毁了吗？为什么还能访问到呢？

到底发生了什么呢：

* 每当 JavaScript 执行一个函数时，都会创建一个作用域对象（scope object），用来保存在这个函数中创建的局部变量。
* 它和被传入函数的变量一起被初始化，你可以认为传入函数的变量也是保存在作用域对象上。
* 这与那些保存的所有全局变量和函数的全局对象（global object）类似，但仍有一些很重要的区别
    * 每次函数被执行的时候，就会创建一个新的，特定的作用域对象
    * 与全局对象（在浏览器里面是当做 window 对象来访问的）不同的是，你不能从 JavaScript 代码中直接访问作用域对象，
    也没有可以遍历当前的作用域对象里面属性的方法

> 所以当调用 makeAdder 时，解释器创建了一个作用域对象，它带有一个属性：a，这个属性被当作参数传入 makeAdder 函数。
然后 makeAdder 返回一个新创建的函数。通常 JavaScript 的垃圾回收器会在这时回收 makeAdder 创建的作用域对象，但是返回的函数却保留一个
指向那个作用域对象的引用。结果是这个作用域对象不会被垃圾回收器回收，直到指向 makeAdder 返回的那个函数对象的引用计数为零。

> 作用域对象组成了一个名为作用域链（scope chain）的链。它类似于原形（prototype）链一样，被 JavaScript 的对象系统使用。

> 一个闭包就是一个函数和被创建的函数中的作用域对象的组合。闭包是一种特殊的对象。它由两部分构成：函数，以及创建该函数的环境。
环境由闭包创建时在作用域中的任何局部变量组成

> 闭包允许你保存状态——所以它们通常可以代替对象来使用

> 闭包是指那些能够访问独立(自由)变量的函数

> 理论就是这些了 — 可是闭包确实有用吗？让我们看看闭包的实践意义。闭包允许将函数与其所操作的某些数据（环境）关连起来。
这显然类似于面向对象编程。在面对象编程中，对象允许我们将某些数据（对象的属性）与一个或者多个方法相关联。



