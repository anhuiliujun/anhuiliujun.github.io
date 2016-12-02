---
layout: post
title:  "react组件生命周期"
date:   2016-12-02 08:30:02 +0800
categories: react
---
### 概览
每个组件都有很多生命周期方法（lifecycle methods）可以使用，我们可以在我们自己的组件中覆盖这些方法。react的生命周期方法遵从如下规则：

- 以`will`开头的，都在某件事发生之前调用
- 以`did`开头的，都在某件事情发生之后调用

我们可以将react组件的生命周期分为三个阶段：

- mounting
    - constructor
    - componentWillMount
    - render
    - componentDidMount
- Updating
    - componentWillReceiveProps
    - shouldComponentUpdate
    - componentWillUpdate
    - render
    - componentDidUpdate
- Unmounting
    - componentWillUnmount

### Mounting
如下方法会在组件实例被创建或者插入到Dom的过程中 被调用：

- constructor
- componentWillMount
- render
- componentDidMount

#### constructor(props)
constructor是component的构造函数，当然会在组件被mount之前被调用。当你实现constructor方法的时候，
唯一需要注意的是：应该在其他语句之前调用super(props)，否则在consturctor中this.props就是undefined了。

##### 最佳实践

**constructor中适合初始化state。** 如果你不需要bind方法也不需要初始化state，那就不需要constructor。

##### 注意
如果你在constructor中基于props初始化state，如下：

```javascript
constructor(props) {
  super(props);
  this.state = {
    color: props.initialColor
  };
}
```
既然你需要基于props去初始化state，通常意味着你需要将state上移到父组件中，这样可以减少bug，也更易于调试。

如果你一定要使用这种方式，由于constructor只用于初始化组件，当父组件的传过来的props变化的时候，如果你想子组件
state跟着变化，通常意味着，你需要实现`componentWillReceiveProps`，在其中触发`setState`。

#### componentWillMount
componentWillMount会在mount之前被调用，即在`render`之前被调用。

##### 最佳实践
由于componentWillMount在render之前被调用，所以在其中直接`setState`并不会触发rerender，只会基于当前setState中的值做一次render，
而我们之所以使用setState肯定是想rerender的，所以绝大多数情况，我们并不需要componentWillMount，因为如果初始化state的话，
我们最好使用`constructor`。虽然我们可以在componentWillMount中处理异步，待数据回来之后使用`setState`，这样尽管也能工作，
但不建议这样做，与componentWillMount的语义不符，在其中只应该做一些组件mount之前的一些准备工作，但是这些又可以在`constructor`中做，
所以componentWillMount就像个鸡肋。

#### componentDidMount
componentDidMount在组件mount之后执行。

##### 最佳实践
如果你需要直接操作Dom，componentDidMount是一个好地方。

如果你需要操作远程数据，然后触发rerender，这里同样是一个好地方。

### Updating
props或者state的变化都可能触发组件的更新。在组件rerender的过程中，如下方法会被调用：

- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate
- render
- componentDidUpdate

#### componentWillReceiveProps(nextProps)
componentWillReceiveProps在mounted组件接收到新的props的时候被调用。

##### 最佳实践
当你需要使state与props保持同步时。通常是因为父组件的变化导致子组件的state需要根据父组件的props同步变化。
此时你可以根据比较this.props与nextProps的不同来触发setState

#### shouldComponentUpdate(nextProps, nextState)
shouldComponentUpdate用于告诉react组件的输出是否应该受state与props变化影响。默认是返回true的，也即是默认情况下，
每一个state的变化都会触发rerender。

当你需要使用shouldComponentUpdate的时候，意味着你想要优化组件的性能。

##### 最佳实践
当你的组件出现性能问题的时候，可以使用shouldComponentUpdate，通过比较this.props，nextProps，this.state，nextState，
来控制是否需要rerender。

如果返回false，componentWillUpdate、render、componentDidUpdate都会被忽略。
但是并不会影响它的子组件的表现。

#### componentWillUpdate(nextProps, nextState)
componentWillUpdate在render之前，当组件接收到新props或者state的时候调用。

##### 最佳实践
如果你想在rerender之前，做一些准备工作，这里是一个好地方，但请不要再其中使用setState或者处理异步等操作。
当你发现你想根据props的变化，触发rerender的时候，请使用componentWillReceiveProps。

##### 注意
在componentWillUpdate中使用setState会触发循环调用。切记勿用！

#### componentDidUpdate(prevProps, prevState)
componentDidUpdate在rerender之后执行。

##### 最佳实践
如果你想在Dom更新之后，操作它，这里是一个好地方。

当你想根据this.props，prevProps，this.state，prevState的比较结果，来判定是否需要一个网络请求时，这里是一个好定。

#### componentWillUnmount
componentWillUnmount在组件被unmount并且删除之前调用。

##### 最佳实践

- 取消网络请求
- 清除一些dom节点

#### render
render方法是component组件的唯一必须的方法。

它必须返回**一个**React元素（element）。该元素可以是Dom组件的一个表示（例如`<div />`），或者用户自定义的组件。

##### 注意
当你不想渲染任何东西的时候，render方法也可以返回null或者false。此时ReactDOM.findDOMNode(this)将会返回null。

render方法应保持`纯净`，你不应该在其中更改任何state，当然如果你在render中使用this.setState，react会给你警告信息。

你同样不应该在render中与browser交互（操作dom），你可以在componentDidMount中做这些操作。