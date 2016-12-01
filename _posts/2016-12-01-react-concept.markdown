---
layout: post
title:  "react 基本概念"
date:   2016-12-01 08:30:02 +0800
categories: react
---
### 缘起

学习react也有3个多月了，一直都是在使用，从未想过它是如何实现的，今天就思考一下react的实现细节，便于对其本质的理解。

### 一些概念

下面这些概念对于理解react的本质非常重要

#### Element

元素（Element）就是一个对象（object），该对象描述了一个组件实例（Component instance）或者文档节点（Dom node）以及它拥有的属性。

通常我们定义一个元素是使用JSX的语法，然后再由babel使用React.createElement转译成javascript的。他们的对应关系如下：

```
<button class='button button-blue'>
  <b>
    OK!
  </b>
</button>
```

如果button是一个Dom node的话，会转译如下

```
React.createElement(
    'button',
    {className: 'button button-blue'},
    React.createElement('b', null, 'OK!')
)
```

最后返回一个object表示此Dom node：

```
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```



而如果Button是一个Component的话，会转译如下：

```
<Button color="blue">
    OK!
</Button>
```

```
React.createElement(
    Button,
    {color: 'blue'},
    'OK!'
)
```

最后返回一个object表示此Component instance

```
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

`createElement`的定义如下：

```
React.createElement(component, props : object, ...children)
````

它的返回值如下：

```
{type: (string | ReactClass), props: Object}
```

React.createElement会将参数中的children数组放到最后的props中（只有一个的时候children的值为object）。

#### Component

组件（Component）封装了元素树（Element Tree）。

当React在渲染的时候，如果他发现元素（Element）的type为function或class，它就知道向组件（Component）去询问该组件会被渲染成什么元素，
同时将props传给该组件。例如当他看到如下元素：

```
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

它就会找到Button Component，最后这个Button可能会返回如下的元素：

```
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

React会重复这个过程，知道它为页面上的所有Component找到底层的Dom标签。

**对于一个Component来说，它的输入时props，输出是一个元素树（element
 tree），返回的元素树可以包括描述Dom nodes的元素，也可以包括描述其他components的元素。
 这使得我们可以组合各个独立的UI，而无需依赖于他们内部的Dom结构。**

#### Instance

 在写react程序的时候，我们几乎很少或者说从不自己主动去创建一个Instance（这与其它基于面向对象的MVC框架可能很不一样），
 而是将创建、更新、删除instance的工作都交给react来做。我们用Component返回的元素（element）来描述instance，
 react内部帮我们管理这些instance。

 当我们在`component class`中使用`this`的时候，这个`this`就是instance，它经常被用于存储本地状态（local state）
 和响应生命周期事件（lefecycle events）。