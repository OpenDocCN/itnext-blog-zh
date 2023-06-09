# 掌握角度:承诺与观察

> 原文：<https://itnext.io/angular-for-junior-developers-promises-vs-observables-618f1d26aaa5?source=collection_archive---------0----------------------->

![](img/1de4d003573d00c152d341230d0ccfaa.png)

“罗讷河上的星夜”，文森特·梵高，1888 年

在这篇文章中，你将了解到可观察到的和承诺之间的区别。

# 术语

*这里有意简化了一些术语。*

*   消费者:代码，当一个承诺或一个可观察物产生一个价值时将被调用(通知)。
    在 Promise 的情况下，是“then”内部的回调函数。
    对于 Observables，是“订阅”中的回调函数。
*   正如我们读到的，同步代码是一行一行执行的。如果有一些繁重的计算，剩下的代码将会等待。
*   异步代码将不会在我们阅读代码时“在下一行”执行，而是在遥远的将来。它可能会在一毫秒后或一分钟后执行——要点是:代码的其余部分不会“等待”那一刻，而是会立即执行。

# 类似

*   两者都可以用于异步操作；
*   两者都在控制消费者何时会获得生产的价值(两者都在把价值“推”给消费者)。

# 差异

## 🏃‍♀️处决

***承诺*** 创建后立即执行。“那么”中是否提供了回调函数并不重要。消费者将异步获取值(如果成功)。但是承诺本身的执行将从创造的那一刻开始。

***可观察的*** 只有订阅时才会执行(“懒”计算)。

实际上，这意味着您可以使用 observables“准备”一些异步代码，并且只在需要时才执行它。例如，它可能是一些复杂的 API 请求(搜索、过滤)，或者一个动画链。

> *注意:可观察值可以同步或异步执行。承诺总是异步产生价值。*

## 📦价值观念

***承诺*** 只能产生一个单值(或一个错误)。

***可观测量*** 可以产生多个值，一个值，或者根本没有值。

对于一个 web app 来说，这意味着 Observables 可以用在很多情况下，而承诺是不能用的。一些可观察对象的行为类似于事件发射器——例如，在 Angular 中，您可以为`@Output`事件使用 EventEmitter()实例。web 应用程序中有许多事件源(DOM events，XHR)。可观察性是处理事件的正确工具。

## 🚧取消

***承诺*** 不可取消。有一些技巧和第三方库可以通过承诺达到这种效果，但是请记住，承诺会立即开始执行——它不会很好地处理取消承诺的尝试。

***观察值*** 被设计为可取消(使用“取消订阅”电话或由操作员取消)。

在 Angular 应用程序中，它将帮助您创建类似“提前键入”(使用`switchMap()`)的功能，并通过在组件的“销毁”生命周期(使用`takeUntil()`)中处理每个可观察对象来防止内存泄漏。

> *提示:当你想添加* `*takeUntil()*` *运算符时，总是把它放在链表的最后一个运算符！*

## ⚙️算子

***承诺*** 没有操作符。

***可观察对象*** 有不同种类的操作符:创建、转换、过滤等等。

使用运算符，您可以完成仅使用承诺难以实现的事情。随着时间的推移，你会掌握这项技能。

> *提示:不要创建过长的操作符链。*

不要被现有操作符的数量吓到:您不需要马上学习所有的操作符。有几个你永远用不到，有几个你偶尔会用到，有几个会成为你最好的帮手。

当你知道一组基本的操作符时，试着每周学习和练习一个新的操作符——每学一个新的操作符，就会越来越容易。

作为基础集，我推荐学习: *map()* ， *tap()* ， *takeUntil()* ， *finalize()* ， *debounceTime()* ， *switchMap()* ， *filter()* ， *catchError()* 。

*读入* ***掌握角度****:*

*   承诺与观察
*   [冷热可观](https://medium.com/p/5cf94052b729)
*   [绘制可观测量图](https://medium.com/p/5af7f7fd8e96)
*   [RxJS 的危险与宝藏](https://medium.com/p/97106873823d)
*   [RxJS 管道](https://medium.com/p/3daac4e75312)
*   [储存库和文件结构](https://medium.com/p/f3084c982415)
*   [基本代码组织原则](https://medium.com/p/c09838dea6e2)