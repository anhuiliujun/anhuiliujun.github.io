---
layout: post
title:  "js中的event loop"
date:   2017-02-17 14:00:02 +0800
categories: javascript
---
### Stack

函数调用形成了一个堆栈帧。里面存放正在执行的函数调用。

### Event Table

每次当tack执行到异步任务时，Stack会将异步任务的回调函数注册到Event Table里。Call Stack告诉Event Table去注册一个函数，只有当指定的
event发生时，才执行此函数，同时，当此事件发生时，Event Table会将此函数移动到Event Queue里。

### Event Queue

里面存放所有将要被执行的函数。

### Event Loop

Event Loop就像个永动机，一直在那轮询Event Queue，如果发现Call Stack是空的，就立即将函数调用推入Call Stack中执行，直到Event Queue
变空，Event Loop就一直在哪里轮询。

更详尽的介绍，请查看MDN的[并发模型与Event Loop]，[what-is-the-javascript-event-loop],
[what-the-heck-is-the-event-loop-anyway]

[what-the-heck-is-the-event-loop-anyway]: https://www.youtube.com/watch?v=8aGhZQkoFbQ
[what-is-the-javascript-event-loop]: http://altitudelabs.com/blog/what-is-the-javascript-event-loop/
[并发模型与Event Loop]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop
