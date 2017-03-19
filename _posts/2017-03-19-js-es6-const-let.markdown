---
layout: post
title:  "es6-const-let"
date:   2017-03-19 12:00:02 +0800
categories: javascript es6
---
### 背景
在es5中我们定义变量的唯一方式是使用`var`关键字，这通常伴随着声明提升（hoisting），如下：

```js
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other code

        return value;
    } else {

        // value exists here with a value of undefined

        return null;
    }

    // value exists here with a value of undefined
}
```

这段代码实际上等价于：

```js
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

这通常会造成猝不及防的bug，使我们的代码更难预测，这是`const`与`let`出现的初衷。

### Block-level Declarations

使用块级声明所声明的变量只能在指定的块级作用域中访问。块级作用域（又叫词法作用域-lexical scopes），会在两种情况下创建：

1. 函数内部
2. `{`与`}`之间

```js
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // other code

        return value;
    } else {

        // value doesn't exist here

        return null;
    }

    // value doesn't exist here
}
```

`let`与`const`都会创建块级作用域，所不同的是，`const`声明的变量不能重新`赋值`（也意味着`const`必须在声明的时候就初始化赋值），
但是如果`const`指向一个`object`，你却能改变这个`object`的属性

> `let`与`const`声明的时候，都会产生一个`binding`，这个`binding`你可以认为它是存在`栈`里面的一个值，
对于`原始类型`数据，这个`binding`的值存的可能就是这个`原始类型`的值，但是对于`引用类型`，`binding`中存储的就是`引用类型`的`引用`。
`const`只是不允许你改变这个`binding`的值，所以对于`引用类型`来说，意味着你不能将`binding`指向一个新的对像，
但是你却可以修改它所指定对象的属性。

### TDZ（块级作用域产生的原理）

```js
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";
}
```

当js`引擎`进入到一个block的时候，它会将所有`const`与`let`所声明的变量放入到一个叫做TDZ（Temporal Dead Zone）的东东里。
然后代码一行一行解释执行的时候，直到`引擎`遇到某个`let`或者`const`声明时，再将对应的变量从TDZ中拿出来。

所以这种机制就会出现以下结果：
1. 在声明之前访问变量会报错（因为此时变量在TDZ中）
2. 在声明之后可以正常访问（此时变量已经从TDZ中移除）

### loops
在loops中使用`var`常常会给人造成困惑。

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // outputs the number "10" ten times
});
```

为了解决这个问题，我们常常需要使用IIFE:

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        }
    }(i)));
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
});
```

这样我们就在作用域链的底层做了一层shadow，解决了我们的问题。

使用`let`就简洁多了：

```js
var funcs = [];

for (let i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // outputs 0, then 1, then 2, up to 9
})
```

`let`其实是模拟了使用IIFE的方式，每次进入block的时候，都新定义了一个变量`i`，并且每次为其初始化不同的值，对于`for-in`与`for-of`也是一样。

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

for (let key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

在loops使用`const`：

```js
var funcs = [];

// throws an error after one iteration
for (const i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

因为将`i`声明为了`const`，所以在给其做++的时候，就会报错。

```js
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

// doesn't cause an error
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // outputs "a", then "b", then "c"
});
```

但是对于`for-in`与`for-of`，只要你在loops内部不修改变量，确是可以使用的。

> 需要注意的一点是，`let`与`const`在loops中的表现是规范中明确定义的，这与规范中针对`hoist`的定义是没有关系的。
事实上一开始，规范中只定义了关于`hoist`，并没有处理loops这种情况。

### 全局块绑定（Global Block Bindings）

`var`与`const`、`let`还有一点区别是他们在全局作用域中的表现。

```js
// in a browser
var RegExp = "Hello!";
console.log(window.RegExp);     // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);        // "Hi!"
```

```js
// in a browser
let RegExp = "Hello!";
console.log(RegExp);                    // "Hello!"
console.log(window.RegExp === RegExp);  // false

const ncz = "Hi!";
console.log(ncz);                       // "Hi!"
console.log("ncz" in window);           // false
```

`var`在定义全局变量的时候，默认会将其添加到`window`的属性上，意味着它会覆盖原来的属性。

`let`与`const`只是定义了全局变量，但是并不会将其添加到`window`的属性上。

> 当你需要在多个frame访问共同的对象时，也许你希望你的全局变量添加到window属性上，此时你可能会用到`var`，否则尽量不要使用它。

### best practice

1. 尽量不要再使用`var`了
2. 优先使用`const`
3. 当你明确变量会在接下来改变的时候，使用`let`
