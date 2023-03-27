# React.js 的钩子——新的番茄酱？

> 原文：<https://itnext.io/hooks-for-react-js-the-new-ketchup-86e8cb03b70?source=collection_archive---------1----------------------->

> *钩子是一个即将到来的特性，它允许你使用状态和其他 React 特性，而不需要编写一个类组件——函数 FTW。*

钩子是最新的样式，是一个实验性的特征，据说比切片面包或番茄酱更好，你决定；).每个人过去都疯狂迷恋[渲染道具](/the-amazing-render-props-pattern-for-react-js-lifecycle-begone-14e520fc3821)，但现在都是钩子。

钩子目前在 React v16.8.0-alpha.0 中，所以你已经可以试用它们了:)

![](img/9b8776eaaab11cfbbaf546f0411e2b2d.png)

# Hooks 试图解决的问题

每次有新的东西出来，我们都很兴奋。是番茄酱，是切片面包之类以来最好吃的东西。我们希望这将最终解决我们所有的问题，所以我们一次又一次地使用它。我们都曾经犯过这样的错误，滥用了一种模式或范式，是的，我们使用的模式一直是有限的。

下面我将试着列出所有不同的痛点，这些痛点让我们看到钩子是一个新的伟大的东西。不过，提醒一句，即使是钩子也会有缺点，所以在有意义的地方使用它。但是现在回到一些抨击和狂言，我们过去构建 React 应用程序的方式是多么糟糕；)

Hooks 正在努力解决许多问题。以下是违规者名单:

*   **包装地狱**，我们都知道所谓的*包装地狱*。组件被一层层的`providers`、`consumers`、`higher-order components`、`render props`等抽象包围，还没穷尽？；)

就像整个包装本身还不够糟糕，我们需要重组我们的组件，这是乏味的，但最重要的是，我们失去了对数据如何流动的跟踪。

*   **越来越复杂**，开始时很小的东西随着时间的推移会变得很大很复杂，特别是当我们添加生命周期方法时
*   **生命周期方法做了太多事情**，组件可能会在`componentDidMount`和`componentDidUpdate`中执行一些数据获取。同一个`componentDidMount`方法也可能包含一些不相关的逻辑来设置事件监听器，清理在`componentWillUnmount`中执行

> 仅仅创建更小的组件？

在许多情况下，这是不可能的，因为:

*   **难测试**，有状态逻辑无处不在，因此很难测试
*   **类混淆了人和机器**，你必须理解`this`在 JavaScript 中是如何工作的，你必须将它们绑定到事件处理程序等等。React 中函数和类组件之间的区别以及何时使用每一个*会导致分歧*并且我们都知道当我们为自己的观点而战时会怎样，空格 vs 制表符谁:)。
*   **缩小问题**，类也给今天的工具带来了问题。例如，类不能很好地缩小，它们使得热重装变得不可靠。有些人可能喜欢类，有些人可能认为函数是唯一的方法。不管怎样，我们只能在 React 和类中使用某些特性，如果它导致这些缩小问题，我们必须找到一个更好的方法。

# 挂钩的卖点

钩子让你不用类就能使用更多 React 的特性**。不仅如此，我们还能打造挂钩，让您:**

*   **从一个组件中提取有状态逻辑**，这样它可以被独立测试和重用。
*   **重用有状态逻辑**，不改变你的组件层次结构。这使得在许多组件之间或与社区共享钩子变得容易。

# 什么是钩子？

钩子让你*根据什么*片段相关*将一个组件分割成更小的功能*(比如设置订阅或获取数据)，而不是基于生命周期方法强制分割。

让我们来概述一下可以使用的不同挂钩。挂钩分为`Basic Hooks`和`Additional Hooks`。让我们先列出`Basic Hooks`并简要说明它们的作用:

# 基本挂钩

*   **useState** ，这是一个钩子，允许你在函数组件内部使用状态
*   **使用效果**，这是一个钩子，允许你以一种替代几种生命周期方法的方式来执行副作用
*   **useContext** ，接受上下文对象(从 React.createContext 返回的值)并返回当前上下文值，该值由给定上下文的最近上下文提供者给出。当提供者更新时，这个钩子将触发用最新的上下文值重新呈现。

在本文中，我们将重点讨论`useState`和`useEffect`。

# 附加挂钩

我们将不再讨论`Additional Hooks`，因为这篇文章太长了，但是我们鼓励你在[附加钩子](https://reactjs.org/docs/hooks-reference.html#additional-hooks)上阅读更多关于它们的内容

*   **useReducer** ，一个`useState`的替代，它接受一个 Reducer 并返回一对当前状态和一个`dispatch`函数
*   **useCallback** ，将返回回调的记忆版本，仅当其中一个输入发生变化时才会发生变化。这在将回调传递给依赖引用相等的优化子组件以防止不必要的呈现时非常有用
*   **useMemo** ，传递一个 *create* 函数和一个输入数组。`useMemo`仅当其中一个输入发生变化时，才会重新计算记忆值。这种优化有助于避免每次渲染时进行昂贵的计算。
*   **useRef** ，返回一个可变 Ref 对象，其`.current`属性被初始化为传递的参数(initialValue)。返回的对象*将在组件*的整个生命周期内持续存在
*   **useImperativeHandle** ，定制使用 ref 时暴露给父组件的实例值
*   **useLayoutEffect** ，签名与 useEffect 相同，但它在所有 DOM 突变后同步触发*。使用它从 DOM 中读取布局并同步重新渲染*
*   ***useDebugValue** ，可用于显示 React DevTools 中自定义钩子的标签*

*正如你在上面看到的，我已经从文档中借用了每个`Additional Hooks`的解释。其目的仅仅是描述存在的东西，给它们每一个都加一行，并敦促你一旦感觉已经掌握了`Basic Hooks`就去研究文档。*

# *使用状态挂钩*

*这个钩子让我们可以在函数组件内部使用状态。是的，我引起你的注意了，对吗？通常，这是不可能的，我们需要使用一个`class`来实现。不再是了。让我们看看使用`useState`钩子是什么样子。我们需要做两件事来开始使用钩子:*

*   ***脚手架**使用 Create React App 的项目*
*   ***升级** `react`和`react-dom`*

*我们将通过键入以下内容来解决第一个问题:*

> **npx 创建-反应-应用挂钩-演示**

*接下来，我们需要升级`react`和`react-dom`，这样它们就可以使用实验版的 React，其中包含了钩子:*

> **纱线添加 react@next react-dom@next**

*现在我们可以走了。*

## *我们的第一个钩子*

*让我们使用`useState`创建我们的第一个钩子，并专注于理解如何使用它。让我们来看一些代码:*

```
*import React, { useState } from 'react';const Counter = () => { 
  const [counter, setCounter] = useState(0); 

  return ( 
    <div> {counter} 
      <button onClick={() => setCounter(counter +1)}>
      Increment
      </button> 
   </div> 
  ) 
}export default Counter;*
```

*好的，我们看到我们通过调用钩子`useState`来使用它，我们像这样调用它:*

> **使用状态(0)**

*这意味着我们给它一个初始值 0。接下来发生的是，当我们调用`useState`时，我们得到一个数组，我们对其进行析构。让我们仔细检查一下:*

> **const [counter，set counter]= use state(0)；**

*好，我们将数组中的第一个值命名为`counter`，第二个值命名为`setCounter`。第一个值是我们可以在我们的`render`方法中展示的实际值。第二个值`setCounter()`是一个函数，我们可以调用它来改变`counter`的值。所以从某种意义上来说，`setCounter(3)`相当于写作:*

> **this . setstate({ counter:3 })**

## *第二个挂钩示例——使用手推车*

*为了确保我们完全理解如何使用它，让我们再创建几个状态:*

```
*import React, { useState } from 'react';const ProductList = () => { 
  const [products] = useState([{ id: 1, name: 'Fortnite' }]); 
  const [cart, setCart] = useState([]);

  const addToCart = (p) => { 
    const newCartItem = { ...p }; 
    setCart([...cart, newCartItem]); 
  }return ( 
    <div> 
      <h2>Cart items</h2> 
      {cart.map(item => <div>{item.name}</div>)} 
     <h2>Products</h2> 
     {products.map(p => <div onClick={() => addToCart(p)}>{p.name}</div>)} 
    </div> 
  ) 
}export default ProductList;*
```

*上面我们正在创建状态`products`和`cart`，这样做我们也得到它们各自的变化函数`setProducts`和`setCart`。我们可以在标记中看到，如果单击我们的`products`列表中的任何一项，我们就会调用方法`addToCart()`。这将导致调用`setCart`，这将导致所选产品不能作为购物车商品添加到`cart`中。这是一个简单的例子，但它真正展示了`setState`钩子的用法。*

# *介绍效果挂钩*

*效果挂钩是用来执行副作用的，例如 HTTP 调用。它执行与生命周期方法`componentDidMount`、`componentDidUpdate`和`componentWillUnmount`相同的任务。*

*下面是我们如何使用它:*

```
*import React, { useEffect, useState } from 'react';const products = [
  { id: 1, name: "Fortnite" }, 
  { id: 2, name: "Doom" }
];const api = { 
  getProducts: () => { return Promise.resolve(products);},
  getProduct: (id) => { return Promise.resolve(
    products.find(p => p.id === id)); 
  } 
}const ProductList = () => { 
  const [products, setProducts] = useState([]); 
  const [product, setProduct] = useState(''); 
  const [selected, setSelected] = useState(2);async function fetchData() { 
    const products = await api.getProducts(); 
    setProducts(products); 
  }async function fetchProduct(productId) { 
    const p = await api.getProduct(productId); 
    setProduct(p.name); 
  } 

  useEffect(() => { 
    console.log('use effect'); 
    fetchData(); 
    fetchProduct(selected); 
  }, [selected]);return ( 
    <React.Fragment> 
      <h1>Async shop</h1> 
      <h2>Products</h2> 
      {products.map(p => <div>{p.name}</div>)} 
     <h3>Selected product</h3> {product} 
     <button onClick={() => setSelected(1)}>Change selected</button
    </React.Fragment> 
  ); 
}export default ProductList;*
```

*好吧，这里发生了很多有趣的事情。让我们先来看看`useEffect`的用法:*

```
*useEffect(() => { 
  console.log('use effect'); 
  fetchData(); 
  fetchProduct(selected); 
}, [selected]);*
```

*我们在上面看到的是我们调用`fetchData`和`fetchProduct`。这两种方法都调用标记为`async`的方法。为什么不能在`useEffect` async 中只做调用函数呢？不幸的是，这是钩子的局限性。*

*查看这两种方法的定义，如下所示:*

```
*async function fetchData() { 
  const products = await api.getProducts(); 
  setProducts(products); 
}async function fetchProduct(productId) { 
  const p = await api.getProduct(productId); 
  setProduct(p.name); 
}*
```

*我们在上面看到，我们在`api`上调用`getProducts`和`getProduct`，它们都返回一个承诺。在收到解决的承诺后，使用`await`我们调用`setProducts`和`setProduct`，它们是我们从`useState`钩子得到的函数。好了，这解释了在这种情况下`useEffect`如何像`componentDidMount`一样工作，但是还有一个细节。让我们再来看看我们的`useEffect`函数:*

```
*useEffect(() => { 
  console.log('use effect'); 
  fetchData(); 
  fetchProduct(selected); 
}, [selected]);*
```

*上面有趣的部分是第二个论点`[selected]`。这是我们查看`selected`变量，并让我们自己得到变化通知，如果`selected`发生变化，那么我们将运行我们的`useEffect`函数。*

*现在，试着点击底部的按钮，你会看到`setSelected`被调用，触发`useEffect`，因为我们正在观察它。*

## *生命周期*

*一般来说，钩子代替了对许多生命周期方法的需求，所以理解哪些方法对我们来说很重要。*

*让我们特别讨论一下效果挂钩和它们的生命周期。*

*以下是对其生命周期的了解:*

*   *默认情况下，React 会在每次渲染后运行效果*
*   *我们的效果是在 React 刷新了对 DOM 的更改之后运行的——包括第一次渲染*

## *访问 DOM 树*

*让我们讨论一下当我们访问 DOM 树时，要执行的一个副作用。如果我们不使用钩子，我们将在方法`componentDidMount`和`componentDidUpdate`中使用。原因是我们不能使用`render`方法，否则它会提前发生。*

*让我们展示如何使用生命周期方法来更新 DOM:*

```
*componentDidMount() { 
  document.title = 'Component started'; 
}componentDidUpdate() { 
  document.title = 'Component updated' 
}*
```

*我们看到，我们可以使用两种不同的生命周期方法来做到这一点。*

*用一个 Effects 钩子访问 DOM 树如下所示:*

```
*const TitleHook = () => { 
  const [title, setTitle] = useState('no title');useEffect(() => { 
    document.title = `App name ${title} times`; 
  }) 
}*
```

*正如你在上面看到的，我们可以访问`props`以及`state`和 DOM。*

*让我们提醒自己，我们对效果挂钩的了解是这样的:*

> **我们的效果是在 React 刷新 DOM 后运行的——包括第一次渲染**

*这意味着两种生命周期方法可以被一种效应所取代。*

## *处理安装/拆卸*

*现在让我们看看这个问题的另一个方面，那就是我们能够，并且我们应该在我们自己之后清理干净。这个想法是这样的:*

```
*useEffect(() => { 
  // set up 
  // perform side effect 
  return () => { 
    // perform clean up here 
  } 
});*
```

*上面我们看到在我们的`useEffect()`函数内部，我们像往常一样执行我们的副作用，但是我们也可以进行设置。我们还看到我们返回了一个函数。所述函数将在最后发生的事情时被调用。*

*我们这里有*设置*和*拆除*。那么我们如何利用这一点呢？让我们看一个人为的例子，这样我们就明白了:*

```
*useEffect(() => { 
  const id = setInterval(() => console.log('logging'));return () => { 
    clearInterval(id); 
  } 
})*
```

*上面演示了整个*设置*和*拆除*的场景，但正如我所说的，这有点做作。你更有可能做一些其他的事情，比如建立一个套接字连接，比如某种订阅，如下所示:*

```
*onMessage = (message) => { 
  // do something with message
}

useEffect(() => { 
  chatRoom.subscribe('roomId', onMessage) 
  return () => { 
    chatRoom.unsubscribe('roomId'); 
  } 
})*
```

# *我可以创建自己的钩子吗？*

*是的，你可以。有了`useState`和`useEffect`，世界完全开放了。你可以创建任何你需要的钩子。*

*问自己以下问题:我的组件会有状态吗？我需要进行 DOM 操作或者 AJAX 调用吗？最重要的是，它是不是可以让不止一个组件受益的有用的东西？如果这里有几个 *yes* 你可以用一个钩子来创建它。*

*让我们看看一些有趣的候选人，看看我们如何使用钩子来构建他们:*

*你可以创造这样的东西:*

*   ***一个模态**，它有一个状态，表示它是否显示，我们需要操作 DOM 来添加模态本身，当模态关闭时，它也需要清理自身*
*   *一个特征标志，特征标志将会有一个状态，在这个状态中，它将会说明某个东西是否应该被显示，它将需要从类似`localStorage`的地方和/或通过 HTTP 获得它的初始状态*
*   *电子商务应用程序中的购物车是最有可能在我们的应用程序中跟随我们的东西。我们可以将购物车同步到`localStorage`以及后端端点。*

# *特征标志*

*让我们试着勾勒出我们的钩子，以及它应该如何表现:*

```
*import React, { useState } from 'react';function useFeatureFlag(flag) { 
  let flags = localStorage.getItem("flags"); flags = flags ? JSON.parse(flags) : null;const [enabled] = useState(Boolean(flags ? flags[flag]: false));return [enabled]; 
}export default useFeatureFlag;*
```

*上面我们已经创建了一个名为`useFeatureFlag`的钩子。它从`localStorage`中读取它的值，并使用`useState`来设置我们的钩子状态。我们不在钩子中析构一个 set 方法的原因是我们不想改变这个值，除非我们重新读取整个页面，此时我们将从`localStorage`重新读取。*

## *测试我们的钩子*

*现在我们已经创建了我们的定制钩子，让我们来试一试。这个想法是对于任何使用我们的钩子只从它的值中读取的组件。如何存储特征标志值取决于钩子。所以钩子是对`localStorage`的抽象。*

```
*import React from 'react'; 
import useFeatureFlag from './flag';const TestComponent = ({ flag }) => { 
  const [enabled] = useFeatureFlag(flag); 

  return ( 
    <React.Fragment> 
      <div>Normal component</div> 
     {enabled && 
       <div>Experimental</div> 
     } 
   </React.Fragment> ); };export default TestComponent;// using it 
<TestComponent flag="experiment1">*
```

## *为我们的功能标志创建一个管理页面*

*我们之前说过，我们对改变由`useFeatureFlag`公开的值不感兴趣。为了控制我们的功能标志，我们选择创建一个特定的管理页面。我们指望管理页面在一个特定的页面上，而带有功能标志的组件在另一个页面上。如果是这种情况，那么在两个页面之间导航将意味着特征标志组件从`localStorage`读取。*

*回到管理页面，如果我们可以列出所有的标志，并按照我们想要的方式切换它们，那就太好了。我们来写这样一个组件。我们的组件应该非常简单，因为它应该只呈现一个标志列表。然而，它需要能够在用户选择时更新一个标志。我们将需要以下内容:*

*   ***一个简单的列表组件**，呈现所有的特征标志，并支持特定标志的切换*
*   ***一个钩子**，它是对`localStorage`的抽象，但是也能够更新它的状态*

*代码如下:*

```
*import React, { useState } from 'react';const useFlags = () => { 
  let flags = localStorage.getItem("flags"); flags = flags ? JSON.parse(flags) : {};const [ flagsValue, setFlagsValue ] = useState(flags);const updateFlags = (f) => { 
    localStorage.setItem("flags", JSON.stringify(f));
    setFlagsValue(f); 
  }return [flagsValue, updateFlags]; 
}const FlagsPage = () => { 
  const [flags, setFlags] = useFlags(); 
  const toggleFlag = (f) => { 
    const currentValue = Boolean(flags[f]); 
    setFlags({...flags, flags[f]: !currentValue}); 
  }return ( 
    <React.Fragment> 
      <h1>Flags page</h1> 
      {Object
        .keys(flags)
        .filter(key => flags[key]).map(flag => 
          <div>
           <button onClick={() => toggleFlag(flag)}>{flag}</button
          </div>
        )
      } 
   </React.Fragment> 
  ) 
}export default FlagsPage;*
```

*我们在上面所做的是从`localStorage`中读出旗帜，然后我们在组件中渲染它们。在一个接一个地呈现它们的同时，我们还在`onClick`上挂接了一个方法(我知道我们在这里谈论的是钩子，但实际上没有双关的意思:)。这个方法是`toggleFlag`，它让我们改变一个特定的标志。在`toggleFlag`中，我们不仅设置了新的标志值，还通过在钩子上调用`setFlags`来确保`flags`拥有最新的更新值。*

*还应该说，我们创建的`useFlags`钩子使得`FlagsPage`组件中的代码变得非常简单，所以钩子也很擅长清理。*

# *摘要*

*在这篇文章中，我们试图解释钩子产生的背景和原因，以及它希望解决和修复的问题。*

*我们已经知道钩子是一种给功能组件状态的方法，但是它们也能够消除对一些生命周期方法的需要。有很多现成的钩子给你，比如下面两个钩子:*

*   ***useState** ，是一个钩子，我们可以用它来保持功能组件中的状态*
*   ***使用效果**也是一个钩子，但是有副作用*

*但是还有很多很多我敦促你去探索，像这些[额外的钩子](https://reactjs.org/docs/hooks-reference.html#additional-hooks)*

*有了钩子，我们可以创建真正酷的和可重用的功能，所以出去吧，做一个自己的钩子。*

# *进一步阅读*

*   *[钩子文档](https://reactjs.org/docs/hooks-overview.html)*
*   *[挂钩背后的动机](https://reactjs.org/docs/hooks-intro.html#motivation)*
*   *[挂钩食谱](https://usehooks.com/)*
*   *[使用钩子的 CRUD 示例](https://www.taniarascia.com/crud-app-in-react-with-hooks)*

*[我的 twitter](https://twitter.com/chris_noring) 我欢迎任何评论或链接到你创建的钩子:)*

*在外面保持冷静！*