---
layout: post
title:  "react性能调谐与diff算法"
date:   2016-12-01 08:30:02 +0800
categories: react
---
### 前提
在任意一个时刻，react中的render创建一颗元素树（element tree），当一个状态（state）变迁或者属性（props）更新，
render将会返回一颗不同的元素树。react接下来将会计算出如何高效的根据最新的元素树更新UI。

react的更新算法，基于以下假设：

1. 两个不同类型（type）的元素产出不同树。
2. 开发者可以通过为子元素添加key属性，标示该元素在不同render之间是保持稳定不变的。


### 算法
比较两棵树的时候，react首先比较两棵树的根节点，会有如下几种情况：

#### 不同类型的元素
销毁原来的树，创建新树。

销毁时，原来的Dom节点会被删除，component instance触发componentWillUnmount事件，创建新树时，新的Dom节点会插入到Dom树中，
component instance会先后触发componentWillMount、componentDidMount。

根节点下的其他组件也会unmounted。例如：

```
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```
将会删除Counter，同时remount一个新的Counter。

#### 相同类型的`Dom`元素
此时react检查两个Dom元素的属性，仅仅更新改变的属性，但是底层Dom节点是不变的。例如：

```
<div className="before" title="stuff" />
<div className="after" title="stuff" />
```
此时仅仅会更改底层Dom节点的className属性。

#### 相同类型的`Component`元素
当一个组件更新时，组件实例保持不变，所以它的状态得以在不同render之间保持。react用新的元素（Element）更新底层组件实例的属性（props），
并且调用实例的componentWillReceiveProps和componentWillUpdate事件方法。

然后调用render。然后这个diff算法会根据之前的结果（prevRenderedElement）与现在的结果（nextRenderedElement）递归调用。

### 关于性能
当在Dom节点的children上递归调用的时候，默认的情况React仅仅同时迭代两棵树，当预见不同的时候就产生一个`改变`。
这对于如下情况，没有问题：

```
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```
react只会插入一个节点。

但是对于如下情况：

```
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```
它会修改1、2两个li的text，然后插入一个li，这显然不是我们想要的，我们当然希望仅仅插入一个节点就可以了。
react需要找到一种方式，让我们在两次render之间，可以通过保持一些不变的东西，让他通过这个值去对比。

react通过`key`属性来表示。当children拥有keys的时候，react使用这些key来比较原始元素树与最新元素树。如下：

```
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```
这样的话react就知道拿key值相同的进行比较，如此以来，发现仅仅添加了一个元素，就解决了我们的性能问题。

#### key的取值
我们需要保证key值在兄弟节点中是唯一的。通常按照如下步骤去选择key：

1. model的id
2. 添加一个唯一的id到model上
3. 使用index（如果不reorder的话）

#### 性能原则
因为react的diff算法基于的那两个假设，如果你的程序不遵从它或者偏离它，就可能出现性能问题。

1. 此算法针对不同类型的component，会remount。所以当你发现两个component拥有极其类似的输出时，公用他们（只有一个类型）。
2. key应该稳定（stable）、可预测（predicable）、唯一（unique）。不稳定的key会导致许多组件实例与Dom节点重复创建。