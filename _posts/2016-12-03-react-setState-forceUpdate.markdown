---
layout: post
title:  "react Api之setState与forceUpdate"
date:   2016-12-03 10:30:02 +0800
categories: react
---
### setState(nextState, callback)
如果我们没有使用像redux之类的状态管理库的话，我们肯定在组件中会大量的使用setState方法来更新组件状态。

setState会将nextState与currentState做一个浅合并(shallow merge)。它通常用于远程请求回调或者事件处理中。

默认情况下，你每次调用setState，都会触发ui的rerender，除非shouldComponentUpdate返回false。

nextState参数可以是一个object，例如：

```javascript
this.setState({mykey: 'my new value'});
```
也可以是一个`function(state, props) => newState`，例如：

```javascript
this.setState((prevState, props) => {
  return {myInteger: prevState.myInteger + props.step};
});
```

需要注意的问题:

setState并不会立即触发this.state的变化，它只会将需要做的操作发送到队列中，所以如果你直接通过this.state取值，
可能访问的不是你本次setState时的值，这意味着，你不能直接根据this.state值来计算setState中的nextState， 例如：

```javascript
// wrong
this.setState({counter: this.state.counter + 1})
```
遇到这种情况，我们只能通过nextState的function的方式，如下：

```javascript
this.setState((prevState, props) => {
  return {counter: prevState.counter};
});
```
使用这种方式，会将一个原子操作放到队列里，在其中我们可以得到之前的state和props。

setState的第二个参数是一个可选参数，当setState完成并且组件rerender之后，会执行这个回调函数。
通常不建议在setState中使用回调函数，推荐使用生命周期函数componentDidUpdate。

### forceUpdate(callback)
默认情况下，一旦组件的state或者props发生变化，组件就会rerender，如果你的组件依赖于外部数据，或者你想将状态管理放到组件之外，此时，你应该使用component.forceUpdate。

调用forceUpdate会触发render方法，并且绕过shouldComponentUpdate。但不会改变其子组件的行为（子组件仍然会执行shouldComponentUpdate）。

也许你听说过redux，专门用于状态管理的，使用它可以将应用的ui部分与state（数据）部分隔离开来管理。
react-redux中间的实现就使用了forceUpdate。下面是其connect方法的一个近似实现：

```javascript
export const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent) => {
    class ConnectInnerComponent extends Component {
        static contextTypes = {
            store: PropTypes.object.isRequired
        };

        componentDidMount() {
            const {store} = this.context;
            this.listener = store.subscribe(() => {
                this.forceUpdate()
            })
        }

        componentWillUnmount() {
            this.listener()
        }

        render() {
            const {store} = this.context;
            return (
                <WrappedComponent
                    {...this.props}
                    {...mapStateToProps(store.getState())}
                    {...mapDispatchToProps(store.dispatch)}
                />
            )
        }
    }
    return ConnectInnerComponent;
};
```