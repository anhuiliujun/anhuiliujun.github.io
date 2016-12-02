---
layout: post
title:  "javascript中的muatble与immutable"
date:   2016-12-02 10:30:02 +0800
categories: javascript
---
mutable（可改变的）与immutable（不可改变的）是一对反义词。

在编程语言中，当我说一个对象是immutable的时候，意味着，我一旦创建了一个对象，我就不能改变它的属性。例如：

```javascript
let a = 'abc';
a[0] = '1';
console.log(a) => 'abc'
a = 1;
console.log(a) => 1
```

此时`'abc'`这个字符串对象就是一个immutable（不可改变）的数据，我们不能改变`'abc'`这个对象。

单这并不是说，我们不能给变量`a`重新赋值，我们只是不能直接改变变量`a`所引用的对象而已。

### javascript中的数据类型
javascript中的数据类型分为如下两类：

- 原始（primitive）数据类型
    - Booleans
    - Numbers
    - Strings
- 对象（object）数据类型
    - Dates
    - Regular expressions
    - Arrays
    - Functions
    - Objects

在`javascript`中，原始数据类型是immutable（不可改变的），而对象数据类型是mutable（可变的）。
对象数据类型是可变的，是因为它们是通过引用（reference）寻址而不是通过值（value）寻址。

`By the way`在ruby中，字符串是mutable的，这与javascript中不同。

```ruby
a = '123'
a[0] = 'a'
p a => 'a23'
```
