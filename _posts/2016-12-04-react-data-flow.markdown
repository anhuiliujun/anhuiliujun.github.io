---
layout: post
title:  "react data flow与controlled/uncontrolled component"
date:   2016-12-03 20:30:02 +0800
categories: react
---
### react data flow
在react的组件中有两类数据，props与state。

props一定是父组件传给子组件的，即一个组件里面的props一定是其父组件传递给它的，props是只读的，即子组件无法直接更改props，
因为props数据本不属于它。

state才是组件自己内部的数据，state是可读写的，而且你只应该在组件内部更改它。父组件可以通过将其state以props的方式传递给子组件，
进而控制子组件的ui。state也只能控制自它而下的组件。

这样一来，我们可以把组件树看成是以props为载体的自上而下（top-down）的单向（unidirectional）流动的一个瀑布，
而每个组件内部的state则看成是在这个瀑布的任意一点加进来的水源，一旦加进来，它同样是自上而下的流动。

以上就是react的单向数据流动，这与很多前端框架的数据流动是不同的。比如angular中推崇的双向绑定（two-way binding），
在其中数据是双向流动的，数据会在ui与model之间形成环路，这往往会非常容易带来极难调试的bug。而在react中数据通常是单向流动的，
state永远只属于特定的组件，假如某个ui出现了bug，你可以逆行数据流动，直到找到state真正属于的组件，精确定位bug。

### controlled component
尽管双向数据流动很容易出bug，但是有些时候，我们又确实需要它。比如我们要实现一个form表单，我们想要在数据提交之前做一个验证，
此时我们就需要获取当前各个输入框的数据。而react给我们提供的input组件默认是内部维护其状态的，即DOM自己维护其数据。

input组件支持我们给其传递参数value，这样的话我们就可以控制input的值了，我们可以让当前组件维护这个state，
然后将其以props的方式传递给input，这样当state变化的时候，ui就跟着变，这是react的单向数据流动。

可是我们希望当用户输入的时候，state也跟着变，这样才能达到数据的回流，让ui与state保持同步。input组件为我们提供了onChange方法，
我们可以通过在onChange中setState来保持数据的回流，如下：

```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

### uncontrolled component
如果你觉得为每一个form元素都写一个onChange太繁琐了，你可以使用uncontrolled的方式，让Dom自己去维护其数据的变化。
但是此时需要使用react的ref。如下所示：

```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

### 最佳实践
什么时候用controlled component，什么时候用uncontrolled component？

逻辑复杂时用controlled，简单时用uncontrolled。

如果你发现你需要实时field验证、条件控制disabled提交按钮等等复杂情况，通常意味着你需要使用controlled component。