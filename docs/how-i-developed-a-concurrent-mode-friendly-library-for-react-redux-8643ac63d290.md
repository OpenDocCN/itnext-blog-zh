# 我如何为 React Redux 开发一个并发模式友好库

> 原文：<https://itnext.io/how-i-developed-a-concurrent-mode-friendly-library-for-react-redux-8643ac63d290?source=collection_archive---------3----------------------->

## 为并发模式做好准备

![](img/a3e2493dcb3f441a369e3d21b467aeaa.png)

# 介绍

几个月来，我一直在开发几个 React hooks 库。在这篇文章中，我将解释为什么以及如何用 React 钩子开发 React Redux 绑定库。该库实现为并发模式友好的。我们来讨论一下为什么它很重要，背后的技术是什么。

React 并发模式还没来，所有讨论都是基于目前的不稳定行为。请注意，当并发模式发布和研究最佳实践时，事情可能会发生变化。

# 什么是并发模式

我对并发模式的理解是一种 React 渲染模式，它可以将一些渲染优先于其他渲染。在并发模式下，React 可以在组件树中间暂停渲染，并丢弃部分渲染的结果。

让我们举一个简单的例子代码。

```
const ParentA = () => (
  <div>
    Hello
    <ChildA />
  </div>
);const ChildA = () => (
  <span>World</span>
);const ParentB = () => (
  <div>
    Hello
    {renderChildB()}
  </div>
)const renderChildB = () => (
  <span>World</span>
);
```

两个组件`ParentA`和`ParentB`会产生相同的结果。在同步模式下可能没有区别。但是，在并发模式下，React 可以在 ParentA 渲染后暂停 ChildA 渲染。这对于`ParentB`是不可能的，因为它会立即调用`renderChildB`。

如果 React 检测到更高优先级的任务，它将暂停渲染并将其丢弃。然后，它将执行该任务，并再次重新开始渲染。(还是继续？)

要暂停渲染，组件必须像`ParentA`一样定义。问题不大，因为开发者已经习惯了。但是，在其他情况下，开发人员必须关注并发模式。如果一个组件不期望暂停，它可能会表现不正确。我将在下一节描述 react-redux 的一个具体问题。在深入讨论之前，让我指出一个解决方案至少有两个层次。

第一层是，如果在并发模式下出现问题，它会退回到同步模式。这是像`renderChildB`一样假设完成的，或者我们可以用同步运行效果的`useLayoutEffect`做回退。如果这种回退的频率非常低，我们将能够在大多数情况下获得并发模式的好处。但是，如果频率非常高，即使我们启用了并发模式，我们也会看到与同步模式下相同的行为。

第二个层次是让它完全在并发模式下工作，没有同步模式回退。这将使我们一直受益。

# 问题

现在我们来讨论一下 react-redux v7.1.0 中的一个问题，下面是使用`useSelector`的示例代码。

```
const App = () => (
  <Provider store={store}>
    <Parent />
  </Provider>
  );const Parent = () => (
  <div>
    <Child />
    <Child />
  </div>
);const Child = () => {
  const count = useSelector(state => state.count);
  return <span>{count}</span>
};
```

即使是这个小例子，在并发模式下也有一个问题。更准确地说，这可能是也可能不是一个问题，取决于一个应用程序的要求。

问题是第一个`Child`和第二个`Child`可能会渲染出不同的`count`。这可能发生在以下步骤中。

1.  最初，`state = { count: 1 }`
2.  `Parent`渲染
3.  第一个`Child`渲染计数=1
4.  更新`state = { count: 2 }`的中断任务到来
5.  第二个`Child`渲染计数=2
6.  状态更新触发`Child`组件重新渲染
7.  两个`Child`组件都以 count=2 进行渲染

所以，不一致的`count`出现在某个点上。更糟糕的是，在某种情况下，当组件在第 6 步之前重新渲染时，`Child`组件不会在第 6 步使用更新的计数重新渲染。(我希望可以通过删除源代码中的一行来解决这个问题。)

之所以会这样，是因为`useSelector`在 render 中调用了`store.getState()`。在 Redux 中，state 是不可变的，但是 store 只能有一个最新版本。因此，`store.getState()`的结果不稳定。

下面的截屏显示了 50 个子组件的不一致性。

![](img/75b92c7621dd42b59a25f8d05c3a5fe4.png)

# 反应-反应-还原 4.1.0 中的溶液

我一直在开发一个名为 reactive-react-redux 的库，它是 react-redux 的替代品。仅支持挂钩 API。

[](https://github.com/dai-shi/reactive-react-redux) [## 代时/反应-反应-还原

### 如果你正在寻找一个非 Redux 库，请访问 react-tracked…

github.com](https://github.com/dai-shi/reactive-react-redux) 

这个库解决了我在上一节中描述的问题。让我注意一下，react-redux 最初试图在 v6 中解决这个问题。我认为它在某种意义上解决了。然而，没有办法用 useContext 来摆脱渲染，react-redux v6 不支持 hooks API。像 v5 一样，react-redux v7 使用存储上下文和订阅来支持 hooks API。

react-redux v6 的工作方式是将存储状态放在一个上下文中，并且不在子组件中使用`store.getState()`。上下文可以有状态和问题解决的多个版本(快照)。

我的库 reactive-react-redux v4 通过在状态上下文中加入订阅机制解决了这个问题。换句话说，它是 react-redux v6 和 v7 的混合体。我的库使用[calculateChangedBits 的一个未记录的特性](https://github.com/dai-shi/reactive-react-redux/issues/29)来混合状态上下文和订阅，这允许优化渲染性能。

react-redux v6 中有一个性能问题，可能是因为它让上下文传播到所有子组件。reactive-react-redux v4 停止传播，性能非常好。[一项基准测试结果](https://github.com/dai-shi/reactive-react-redux/issues/32#issuecomment-513507770)显示，它的性能与 react-redux v7 相当或略好。

# 测试库的工具

起初，我不太确定我的库是否真的可以在并发模式下工作而不会出现这个问题。所以，我开发了一个测试工具。(前一部分的截屏由工具提供。)

[](https://github.com/dai-shi/will-this-react-global-state-work-in-concurrent-mode) [## Dai-Shi/will-this-react-global-state-work in-concurrent-mode

### 检查 React 并发模式中的撕裂在 react-redux 中，有一个称为“撕裂”的理论问题可能会发生…

github.com](https://github.com/dai-shi/will-this-react-global-state-work-in-concurrent-mode) 

这个工具有一个小应用程序来显示许多计数并检查不一致性。这是开玩笑的结果。

```
react-redux
    ✓ check1: updated properly (975ms)
    ✕ check2: no tearing during update (18ms)
    ✓ check3: ability to interrupt render (1ms)
    ✕ check4: proper update after interrupt (5083ms)
  reactive-react-redux
    ✓ check1: updated properly (1448ms)
    ✓ check2: no tearing during update (3ms)
    ✓ check3: ability to interrupt render
    ✓ check4: proper update after interrupt (751ms)
```

如果一个库通过了所有四项检查，它很可能以并发模式工作，并从中受益。check3 测试任务是否会中断渲染。如果它退回到同步模式，此检查将失败。

我在 react vee-react-redux v 4 . 0 . 0 中的最初实现存在这个问题，check3 失败了。那是因为我用了同步运行特效的`useLayoutEffect`。reactive-react-redux v4.1.0 消除了它，并通过了所有检查。

截至发稿，关于并发模式的文档并不多，一切都是基于观察。这意味着，任何事情都可能出错。请注意。

# 结束语

并发模式尚未发布。所以，某种意义上，一切都是假设。然而，我发现构建一个测试工具是有价值的，因为我们可以讨论行为，即使它是不稳定的。行为可以在以后更改，但是我们可以更新工具来跟随更改。

所以，这篇文章的主要目的是鼓励人们尝试这个工具并给出反馈。

第二个目标是告知我在 reactive-react-redux 中使用的技术。仍有更多改进和修正的空间。所以，也欢迎反馈。

最后，我用同样的技术开发了一些其他的库。

[](https://github.com/dai-shi/react-tracked) [## 戴式/反应跟踪式

### 如果你正在寻找一个基于 Redux 的库，请访问…

github.com](https://github.com/dai-shi/react-tracked) 

react-tracked 提供了与 reactive-react-redux 中相同的钩子 API，但没有 redux。

[](https://github.com/dai-shi/use-context-selector) [## 代时/使用上下文选择器

### React useContextSelector 钩子在 userland React Context 和 useContext 中经常被用来避免正确的钻取，然而…

github.com](https://github.com/dai-shi/use-context-selector) 

use-context-selector 在 userland 中提供了`useContextSelector`钩子。

希望你喜欢这篇文章。

*原载于 2019 年 7 月 27 日*[*【https://blog.axlight.com】*](https://blog.axlight.com/posts/how-i-developed-a-concurrent-mode-friendly-library-for-react-redux/)*。*