---
layout: post
title:  "react性能优化与immutable"
date:   2016-12-02 17:30:02 +0800
categories: react
---
### 使用react一定要使用immutable data吗
简单回答，react当然可以不使用immutable的数据结构，react对此并无任何要求。

### 为什么要使用immutable的数据
假如你需要在组件中使用componentWillReceiveProps(nextProps)，你想要通过比较this.props与nextProps中的值是否相等，通过这个，来决定是否触发setState，进而rerender，如果你的props事mutable的，意味着this.props中的值与nextProps中的值是同一个（你直接在原始对象上做了修改），所以此时的结果是，this.props与nextProps是一个东西，你决定不触发setState，尽管事实上props确实改变了，这显然不是你想要的结果。

另外一种情况，当你的组件有性能问题，而你想调优性能时。react提供了shouldComponentUpdate(nextProps, nextState)，你可以在其中通过比较this.props，nextProps，this.state，nextState来决定是否做Reconciliation，如果返回false，就意味着跳过componentWillUpdate、render、componentDidUpdate，同时因为不调用当前组件的render方法，自然不会调用子组件的render，自此节点以下的整个过程都可以不做，这可以显著提高性能。（但是如果子组件的state变化了，子组件当然会rerender，子组件走的是它自己那个shouldComponentUpdate的判断过程）。

### 如何做到immutable

使用deepClone，我们可以修改一个对象之前，做一次深度克隆，这样当然就不会影响到原始对象了，但是这显然存在性能问题。

使用ES6的[object spread properties][object-spread]，或者[Object.assign][object-assign]，这种方式适用于结构比较简单的数据。

使用[Immutable.js][immutable]，这种方式适用于数据结构比较复杂的情况。

### 最佳实践
尽管react并不要求使用immutable的数据结构，但是使用immutable的数据结构却可以避免很多不必要的bug。但是直接引入Immutable.js这种重型的解决方案，又可能需要更多的学习。（通常不建议在一开始就引入Immutable.js， 除非team内部都已经很熟悉其api了）

还好，Immutable.js这种事情，不是all or nothing。

通常我会在一个项目中遵从这样的原则：

- 尽量保证数据结构设计的简单（尽量不嵌套）
- 尽量通过es6的方式，达到immutable。
- 在数据结构足够复杂的组件中，考虑引入Immutable.js

### 参考资料
如果想详细了解，下面的资料也许会帮到你：

- [javascript中的muatble与immutable][js-mutable]
- [why-should-i-care-about-immutable-data-in-reactjs][immutable-data-in-reactjs]
- react官网[optimizing-performance][optimizing-performance]

[object-spread]: https://github.com/sebmarkbage/ecmascript-rest-spread
[object-assign]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
[immutable]: https://github.com/facebook/immutable-js
[optimizing-performance]: https://facebook.github.io/react/docs/optimizing-performance.html
[immutable-data-in-reactjs]: https://www.bennadel.com/blog/2903-why-should-i-care-about-immutable-data-in-reactjs.htm
[js-mutable]: https://anhuiliujun.github.io/javascript/2016/12/02/javascript-mutable.html