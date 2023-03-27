# 带有 React 钩子的全局状态的四种模式:上下文或 Redux

> 原文：<https://itnext.io/four-patterns-for-global-state-with-react-hooks-context-or-redux-cbc2dc787380?source=collection_archive---------1----------------------->

## 我开发的库

![](img/0267faf10e6be4c5b9c9d0f2fa2a190b.png)

# 介绍

当您开始开发 React 应用程序时，全局状态或共享状态是最大的问题之一。该不该用 Redux？钩子提供了类似 Redux 的解决方案吗？我想展示使用 Redux 的四种模式。这是我个人的观点，主要针对新的应用。

# 模式 1:正确传球

有些人可能认为它不会扩展，但是最基本的模式仍然应该是适当的传递。如果应用程序足够小，在父组件中定义本地状态，然后简单地将它传递给子组件。我可以忍受两级通过，意味着一个中间组件。

```
const Parent = () => {
  const [stateA, dispatchA] = useReducer(reducerA, initialStateA);
  return (
    <>
      <Child1 stateA={stateA} dispatchA={dispatchA} />
      <Child2 stateA={stateA} dispatchA={dispatchA} />
    </>
  );
};const Child1 = ({ stateA, dispatchA }) => (
  ...
);const Child2 = ({ stateA, dispatchA }) => (
  <>
    <GrandChild stateA={stateA} dispatchA={dispatchA} />
  </>
);const GrandChild = ({ stateA, dispatchA }) => (
  ...
);
```

# 模式 2:上下文

如果一个应用程序需要在两层以上的组件之间共享状态，那么就该引入上下文了。上下文本身不提供全局状态功能，但是结合本地状态并通过上下文传递可以完成这项工作。

```
const ContextA = createContext(null);const Parent = () => {
  const [stateA, dispatchA] = useReducer(reducerA, initialStateA);
  const valueA = useMemo(() => [stateA, dispatchA], [stateA]);
  return (
    <ContextA.Provider value={valueA}>
      <Child1 />
    </ContextA.Provider>
  );
};const Child1 = () => (
  <GrandChild1 />
);const GrandChild1 = () => (
  <GrandGrandChild1 />
);const GrandGrandChild1 = () => {
  const [stateA, dispatchA] = useContext(ContextA);
  return (
    ...
  );
};
```

注意，如果`stateA`改变了，所有带有`useContext(ContextA)`的组件都会重新渲染，即使只是状态的很小一部分。因此，不建议将上下文用于多种用途。

# 模式 3:多重上下文

使用多个上下文是很好的，并且更推荐用来分离关注点。上下文不必是应用范围的，它们可以用于组件树的一部分。只有当你的上下文可以在你的应用中的任何地方使用时，在根定义它们才是一个好的理由。

```
const ContextA = createContext(null);
const ContextB = createContext(null);
const ContextC = createContext(null);const App = () => {
  const [stateA, dispatchA] = useReducer(reducerA, initialStateA);
  const [stateB, dispatchB] = useReducer(reducerB, initialStateB);
  const [stateC, dispatchC] = useReducer(reducerC, initialStateC);
  const valueA = useMemo(() => [stateA, dispatchA], [stateA]);
  const valueB = useMemo(() => [stateB, dispatchB], [stateB]);
  const valueC = useMemo(() => [stateC, dispatchC], [stateC]);
  return (
    <ContextA.Provider value={valueA}>
      <ContextB.Provider value={valueB}>
        <ContextC.Provider value={valueC}>
          ...
        </ContextC.Provider>
      </ContextB.Provider>
    </ContextA.Provider>
  );
};const Component1 = () => {
  const [stateA, dispatchA] = useContext(ContextA);
  return (
    ...
  );
};
```

如果我们有更多的上下文，这会有点混乱。是时候介绍一些库了。有几个库支持多个上下文，其中一些提供了 hooks API。

我一直在开发这样一个库，叫做“react-hooks-global-state”。

[](https://github.com/dai-shi/react-hooks-global-state) [## 戴式/反应钩式-全局-状态

### 与钩子 API 反应的简单全局状态。通过创造一个新的世界，为 Dai-Shi/react-hooks-global-state 的发展作出贡献

github.com](https://github.com/dai-shi/react-hooks-global-state) 

这是它看起来的示例代码。

```
import { createGlobalState } from 'react-hooks-global-state';const initialState = { 
  a: ...,
  b: ...,
  c: ...,
};
const { GlobalStateProvider, useGlobalState } = createGlobalState(initialState);const App = () => (
  <GlobalStateProvider>
    ...
  </GlobalStateProvider>
);const Component1 = () => {
  const [valueA, updateA] = useGlobalState('a');
  return (
    ...
  );
};
```

这个库中至少有一个警告。它使用了一个名为`observedBits`的未记录的特性，不仅不稳定，而且有其局限性，这个库只有在子状态(如`a`、`b`、`c`)的数量等于或小于 31 时才有效。

# 模式 4: Redux

多上下文的最大限制是分派功能也是分离的。如果你的应用程序变得很大，需要用一个动作更新几个上下文，那么是时候引入 Redux 了。(或者，实际上您可以为一个事件分派多个动作，我个人非常不喜欢这种模式。)

有各种各样的库可以将 Redux 与 hooks 一起使用，官方 react-redux 即将发布其 hooks API。

因为我在这个领域投入了大量的精力，所以让我介绍一下我的库“reactive-react-redux”。

[](https://github.com/dai-shi/reactive-react-redux) [## 代时/反应-反应-还原

### 用 React 钩子和代理绑定 React Redux。通过创造一个新的环境，为 dai-shi/reactive-react-redux 的发展做出贡献

github.com](https://github.com/dai-shi/reactive-react-redux) 

与传统的 react-redux 不同，这个库不需要`mapStateToProps`或选择器。您可以简单地使用 Redux 中的全局状态，该库使用 Proxy 跟踪状态使用情况以进行优化。

这是它看起来的示例代码。

```
import { createStore } from 'redux';
import {
  ReduxProvider,
  useReduxDispatch,
  useReduxState,
} from 'reactive-react-redux';const initialState = {
  a: ...,
  b: ...,
  c: ...,
};const reducer = (state = initialState, action) => {
  ...
};const store = createStore(reducer);const App = () => (
  <ReduxProvider store={store}>
    ...
  </ReduxProvider>
);const Component1 = () => {
  const { a } = useReduxState();
  const dispatch = useReduxDispatch();
  return (
    ...
  );
};
```

# 最后的想法

对于中型到大型的应用程序，单个事件可能会改变状态的几个部分，从而改变用户界面。因此，在这种情况下，使用 Redux(或任何种类的应用程序状态管理)似乎是很自然的。

然而，apollo-client 和即将推出的 react-cache 将扮演数据管理的角色，UI 状态管理的角色将变得更小。在这种情况下，多上下文模式可能对中等应用更有意义。

*原载于 2019 年 5 月 27 日【https://blog.axlight.com】[](https://blog.axlight.com/posts/four-patterns-for-global-state-with-react-hooks-context-or-redux/)**。***