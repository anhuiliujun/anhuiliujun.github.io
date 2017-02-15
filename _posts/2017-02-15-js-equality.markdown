---
layout: post
title:  "js中的相等性判断与数组去重"
date:   2017-02-15 20:30:02 +0800
categories: javascript
---
### 规范中的相等

- [Abstract Equality Comparison][abstract-equal] (==)
- [Strict Equality Comparison][strict-equal] (===)，`Array.prototype.indexOf`、`Array.prototype.lastIndexOf`与`case-matching`都使用这种策略
- [SameValueZero][same-value-zero] `TypedArray`、`ArrayBuffer`、`Map`、`Set`、`Array.prototype.includes`、`String.prototype.includes`都使用
这种策略
- [SameValue][same-value] `Object.is`和其他情况均使用这种策略

### 区别

- Abstract Equality Comparison 会做类型转换
- Strict Equality Comparison 对于不同类型的值，直接返回false
- SameValueZero 与 Strict Equality Comparison的唯一不同的地方是对`NaN`的处理，SameValueZero对于两个`NaN`返回true，
而Strict Equality Comparison返回false
- SameValue 与 SameValueZero的唯一不同是对于+0与-0的处理，SameValueZero返回true，而SameValue返回false

需要注意的是，所有这些策略都是针对原始类型的，对于对象，仅仅是判断他们是否是同一个实例。

有些面试题目会问到数组去重的问题，其实这个题目的考察点的核心就是*相等性*判断，关于数组去重，详见兔八哥的博文[也谈JavaScript数组去重][uniq]

[uniq]: https://www.toobug.net/article/array_unique_in_javascript.html
[abstract-equal]: http://ecma-international.org/ecma-262/7.0/#sec-abstract-equality-comparison
[strict-equal]: http://ecma-international.org/ecma-262/7.0/#sec-strict-equality-comparison
[same-value-zero]: http://ecma-international.org/ecma-262/7.0/#sec-samevaluezero
[same-value]: http://ecma-international.org/ecma-262/7.0/#sec-samevalue
