# 反应设置状态用法和陷阱

> 原文：<https://itnext.io/react-setstate-usage-and-gotchas-ac10b4e03d60?source=collection_archive---------0----------------------->

![](img/651693188ebf89b808998ee4a6b5c3d6.png)

> [*点击这里在 LinkedIn*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Freact-setstate-usage-and-gotchas-ac10b4e03d60%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer) 上分享这篇文章

React 类组件有一个内部状态，像 props 一样影响组件的渲染和行为。与 props 不同，状态是组件的本地状态，只能在组件内部初始化和更新。

# **初始化**

在使用 state 之前，我们需要为初始状态声明一组缺省值。这可以通过在构造函数中创建一个状态对象或者直接在类中创建来实现。

```
class Counter extends React.Component {
  constructor(props, context) {
    super(props, context)
    this.state = {
      quantity: 1,
      counter: 0
    }
  }
}class Counter extends React.Component {
  state = {
    quantity: 1,
    counter: 0
  }
}
```

# **更新**

状态可以响应于事件处理程序、服务器响应或属性改变而被更新。React 为此提供了一个名为`setState`的方法。

`setState()`将组件状态的更改排入队列，并告诉 React 该组件及其子组件需要用更新后的状态重新呈现。

```
this.setState({quantity: 2})
```

这里，我们传递了一个包含我们想要更新的状态的对象`setState()`。传递的对象会有对应于组件状态中的键的键，然后`setState()`通过将对象合并到状态来更新或设置状态。

# **设置状态并重新渲染**

`setState()`将总是导致重新渲染，除非`shouldComponentUpdate()`返回`false`。为了避免不必要的渲染，只在新状态不同于前一状态时调用`setState()`是有意义的，并且可以避免在某些生命周期方法(如`componentDidUpdate`)中无限循环地调用`setState()`。

> 从 16 开始，用`null`调用`setState`不再触发更新。这意味着我们可以决定是否在我们的`setState`方法本身中更新状态！

# **签名**

```
setState(updater[, callback])
```

第一个参数是一个带有签名的`updater`函数:

```
(prevState, props) => stateChange
```

`prevState`是对之前状态的引用。应该不会直接变异。相反，应该通过基于来自`prevState`和`props`的输入构建一个新对象来表示变化。例如，通过`props.step`增加状态值:

```
this.setState((prevState, props) => {
  return {counter: prevState.counter + props.step};
})
```

> 由于`setState`的异步特性，不建议使用`this.state`在`setState`内获得前一个状态。而是一直依靠上面的方式。更新函数接收到的`prevState`和`props`都保证是最新的。更新器的输出与`prevState`浅合并。

`setState()`的第二个参数是一个可选的回调函数，一旦`setState`完成并且组件被重新渲染，这个函数就会被执行。`componentDidUpdate`应改为在大多数情况下应用此类逻辑。

你可以直接传递一个对象作为第一个参数给`setState`，而不是一个函数。这将执行状态变化到新状态的浅层合并。

```
this.setState({quantity: 2})
```

# 批处理状态更新

如果进行了多次`setState()`调用，React 可能会在考虑更新顺序的同时批量更新状态。目前(React 16 及更早版本)，**默认情况下只有 React 事件处理程序内部的更新被批处理**。事件结束时，更改总是一起刷新，您看不到中间状态。

无论您在 React 事件处理程序中调用多少个组件，它们都只会在事件结束时产生一次重新渲染。例如，如果子节点和父节点在处理 click 事件时都调用了`setState()`，那么子节点只会重新渲染一次。

直到 React 16，**在 React 事件处理程序**之外默认没有批处理。因此，如果每个`setState()`位于任何事件处理程序之外，它们将被立即处理。例如:

```
promise.then(() => {
  // We're not in an event handler, so these are flushed separately.
  this.setState({a: true}); // Re-renders with {a: true, b: false }
  this.setState({b: true}); // Re-renders with {a: true, b: true }
})
```

然而，`ReactDOM`提供了一个 api，可以用来强制批量更新。

```
promise.then(() => {
  // Forces batching
  ReactDOM.unstable_batchedUpdates(() => {
    this.setState({a: true}); // Doesn't re-render yet
    this.setState({b: true}); // Doesn't re-render yet
  });
  // When we exit unstable_batchedUpdates, re-renders once
})
```

内部 React 事件处理程序都包装在`unstable_batchedUpdates`中，因此默认情况下它们是批处理的。在`unstable_batchedUpdates`中包装一个更新两次没有效果。当最外层的`unstable_batchedUpdates`调用退出时，更新被刷新。

> 该 API 是“不稳定的”,因为当批处理在 React 核心中已经默认启用时，它将被删除。

# 在 React 组件外部声明状态更改

这一部分的灵感来自下面丹·阿布拉莫夫的推文。

由于`setState`期望参数中有一个函数，该函数可以在 React 类之外的某个地方实现，然后作为`setState`的参数导入并在组件内部使用。如果需要额外的参数，高阶函数可以帮忙。

```
const multiplyBy = multiplier => state => ({
  value: state.value * multiplier
})
```

由于状态更新现在是普通的 JavaScript，测试复杂的状态转换不会涉及 React 组件的浅层呈现。

# setState 和生命周期方法

在生命周期方法中调用`setState`需要一定程度的谨慎。有一些方法调用 setState 是没有意义的，也有一些方法应该有条件地调用。让我们具体情况具体分析。

## 组件将安装

`setState`可以在这里调用。更新后的状态将在即时呈现中使用，只要它不是基于承诺解析计算的。建议使用`componentDidMount`对承诺解析等执行任何状态更新。不使用该方法的另一个原因是，在 React 的未来版本(17 以后)中，该方法将被弃用。

[](https://github.com/reactjs/rfcs/blob/master/text/0006-static-lifecycle-methods.md) [## reactj/RFC

### RFC-对变更做出反应的 RFC

github.com](https://github.com/reactjs/rfcs/blob/master/text/0006-static-lifecycle-methods.md) 

## 组件安装

这是异步渲染的首选方法。

## componentWillReceiveProps

此方法的主要目的是计算一些从 props 派生的值，以便在渲染过程中使用。虽然，如果计算足够快，它可以在`render`完成。在这里使用绝对安全。

## shouldComponentUpdate

这是更新组件状态没有意义的方法之一。它在更新阶段(props 或状态更改)由 React 在内部调用。在这里调用`setState`会导致无限循环，因为这是它在更新状态时调用的下一个方法。如果需要在道具更新阶段设置状态，使用`componentWillReceiveProps`。

## 组件将更新

这里不要用`setState`。和`shouldComponentUpdate`类似的原因。

## 提供；给予

在这里调用`setState`会使您的组件成为产生无限循环的竞争者。`render`应保持纯净，并用于根据状态或道具有条件地在 JSX 片段/子组件之间切换。render 中的回调可用于更新状态，然后根据更改重新呈现。

> 如果你发现自己不得不在`render`中写`setState`，你可能需要重新考虑设计。事实上，这可能是实现状态机模式的完美用例。

[](https://medium.freecodecamp.org/boost-your-react-with-state-machines-1e9641b0aa43) [## 使用状态机增强您的反应

### 混合 React 和状态机对于开发人员来说是极大的生产力提升。它还提高了通常…

medium.freecodecamp.org](https://medium.freecodecamp.org/boost-your-react-with-state-machines-1e9641b0aa43) 

## componentDidUpdate

这里调用`setState`最常见的用例是更新 DOM 以响应属性或状态的变化。这里，在再次更新状态之前，您等待组件被呈现。这使得它成为设置依赖于呈现的 DOM 值的状态值的候选对象。请记住检查您是否再次更新了相同的状态，因为在调用`setState()`之后会再次调用该方法。例如:

```
componentDidUpdate = (prevProps, prevState) => {
  let width = ReactDOM.findDOMNode(this).parentNode.offsetWidth
  if (prevState && prevState.width !== width) {
    this.setState({ width })
  }
}
```

## 组件将卸载

这里不要用`setState`。你的组件正在消失。

参考

*   [https://reactjs.org/docs/react-component.html#setstate](https://reactjs.org/docs/react-component.html#setstate)
*   [https://stack overflow . com/questions/48563650/does-react-keep-the-order-for-state-updates/48610973 # 48610973](https://stackoverflow.com/questions/48563650/does-react-keep-the-order-for-state-updates/48610973#48610973)

***附:如果这篇文章对你有帮助，一定要鼓掌👏，关注我的*** [***推特***](https://twitter.com/nitishk88) ***，并分享给你的朋友！***