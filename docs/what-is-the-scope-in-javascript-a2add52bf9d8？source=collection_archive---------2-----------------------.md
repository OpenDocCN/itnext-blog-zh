# JavaScript 中的作用域是什么

> 原文：<https://itnext.io/what-is-the-scope-in-javascript-a2add52bf9d8?source=collection_archive---------2----------------------->

## 解释作用域中变量、常量、Let、函数、对象和类的行为。

![](img/a9a5e6b0297629280ff13388fe90a994.png)

paweczerwi ski 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

对于刚开始 JavaScript 开发的代码新手和开发人员来说，JavaScript 中的作用域可能是一个很难理解的复杂话题。

在这篇文章中，我想把这个复杂的话题变成一个非常容易理解的概念，这样每个人都可以理解它是如何工作的，你可以用它做什么。

范围是一个需要理解的非常重要的概念。当你头脑中有了这个概念，用 JavaScript 构建令人惊叹的东西就会容易得多。

> 如果你是那种喜欢演示的人，那么我推荐我几年前制作的一张幻灯片: [JavaScript essentials 演示](http://slides.com/raymonschouwenaar-1/javascript-essentials#/8)

# 什么是范围

对于以英语为母语的开发人员来说，这是显而易见的。但是对于那些英语不是他们母语的人来说，这并不明显。

我喜欢下面的定义，它使它变得如此清晰。

> **范围，范围，到达，轨道，罗盘，界限(名词)**
> 某物活动或操作或有力量或控制的区域:“超音速喷气式飞机的范围”。**来源**:[Definitions.net](https://www.definitions.net/definition/scope)

这是有道理的权利！

在我自己的世界里，我将它定义为*一个可以对外界隐藏但从内部可见的区域*。

# 范围类型

在 JavaScript 中，我们有两种类型的作用域。全局范围和局部范围。

1.  **全局范围** : *全局范围内声明的所有东西都是公开的。*
2.  **局部作用域** : *局部作用域中声明的所有信息只在那个作用域中可用。*

让我们深入研究这两种类型的范围。了解你能用它做什么，不能做什么，以及如何使用它。

# 1.全球范围

1.  定义变量
2.  功能
3.  班级
4.  Const & Let

# 1.全球范围

![](img/3fd478b0368211a6bb4afe4207042d8d.png)

美国宇航局在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

最重要也是最关键的作用域是“全局作用域”。每一个其他的作用域都可以到达在全局作用域中定义的所有东西。

在浏览器中我们有`window`对象，在 NodeJS 中我们有`global`对象。在这篇文章中，当我说“*全球范围*”时，我指的是他们两个。

关于全局范围，你需要记住的一件事是，你不需要在其中存储太多的信息。建议将信息存储在本地范围内。

## 1.1 Var

当你在全局范围定义一个`var`时，它在全局对象中是公开可用的。

你保存在全局作用域或者任何其他作用域的`var`都可以被覆盖。建议使用`let`或`const`代替`var`。因为`var`没有“块范围”的行为。

## 1.2 功能

当你在全局作用域中定义一个`function`时，它在全局对象中是公开可用的。

## 1.4 类别

当你定义一个`class`时，它是公开可用的，但不绑定到全局对象。

绑定到全局对象的内容和不绑定到全局对象的内容有很大的不同，但在全局范围内仍然可用。

## 1.3 Const & Let

当你定义一个`let`或`const`变量时，它是公开可用的，但不绑定到全局对象。

# 2.局部范围

1.  定义变量
2.  功能
3.  Const & Let
4.  班级

# 2.局部范围

![](img/4b9a3d394baa119f55e82253c7551425.png)

本杰明·戴维斯在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

当您定义`function`、`if`或`class`时，基本上在`{}`之间而不是作用域内的所有东西都会被创建。这就是我们所说的局部范围。

在局部作用域中声明的所有内容都只能在内部使用。一个别的范围甚至全局范围都达不到。

# 2.1 Var

当你在局部作用域中定义一个`var`时，它只在该作用域内可用。所以一个`var`就是我们所说的`function`作用域。如果您试图在该范围之外访问它，您将得到错误消息，即它是`undefined`。

> 既然我们有`*let*`和`*const*`，我建议不要再使用`*var*`，因为它有一些副作用。我会在这篇文章中更好地解释这一点。

**请点击控制台查看结果😉**

在这个代码示例中，我们可以看到，如果在全局作用域中定义了一个`var`，那么它对于其他所有作用域都是可见的。当我们在函数内部定义一个`var`时，我们只能在函数内部看到它。功能外显示为`undefined`。

但是在我们的`function`中，我们可以覆盖全局定义的`var`。

**请点击控制台查看结果😉**

正如你所看到的，在我们定义了`globalScope`变量之后，这个值就像我们定义的“GlobalScope”一样。当我们在函数中操作它之后，它有一个👏它后面的表情符号。

这将导致您构建的应用程序出现不可预测的行为。你肯定会以错误告终。

# 2.2 功能

一个一个 `function`单独存在称为*函数*，一个`function`附加到`Object`或`Class`上称为*方法*。在函数中定义的`function`是一个普通函数，但在外部作用域中是隐藏的。

当你在全局作用域中定义一个`function`时，它在全局对象中是公开可用的。

## 2.2.2 对象内部的方法

当您在`Object`中定义一个方法或属性时，它被绑定到那个范围，但是仍然可以通过那个对象使用。如果你在那个方法中定义了变量，它将保持隐藏。

使用`class`时，其工作原理与`Object`相似。

> 将来，由于在类中引入了[私有字段，这种情况将会改变。目前这仅在浏览器 Chrome 和 Opera 以及工具 Babel 和 TypeScript(3.8 版)中受支持。](https://medium.com/better-programming/classes-with-private-properties-in-typescript-3-8-9fdb91c26ab1)

**请点击控制台查看结果😉**

如您所见，我们可以从`Object`中覆盖`globalScope`变量。我们也可以在全局范围内改变`name`属性的值。

虽然这些东西是不受 nog 保护的，但是我们的方法中的变量不能改变，也不能从全局范围内看到。

## 2.2.3 函数中的函数

用函数中的函数

**请点击控制台查看结果😉**

# 2.3 Const & Let

T he `const`和`let`变量是块范围的。当一个`var`绑定到一个`function`的范围时，`const`和`let`的范围在每个`{}`之间。

`const`和`let`变量可以由`function`、`if-statment`、`for-loop`(或任何其他循环)保护。超出了它的范围，它就无法到达。

## 让我们

如果你想在相同的范围内重写信息，你必须声明一个`let`变量，因为信息可以被改变。`let`变量不能重新声明。

## 2.3.2 常数

在 JavaScript 执行期间需要相同的信息可以在`const`变量中找到它的位置。常量变量既不能被重写，也不能被重新声明。

如果`const`变量有一个`Object`或`Array`，你可以添加和删除属性。但是如果在其中存储单一类型的值，它保持不变。

# 2.4 课程

一个T21 自身称为*函数*，附加到`Object`或`Class`的`function`称为*方法*。在函数中定义的`function`是一个普通函数，但是在外部作用域中被隐藏了。

## 2.4.1 类中的方法

在这个例子中，你可以看到，在方法中声明的变量在外部是不可用的。`globalScope`变量在所有局部作用域中都可用。当创建类的`new`实例时，类外部的属性和方法是可用的。

在我们的类中，我们可以修改`globalScope`变量。在我们的类外部，我们可以修改我们的`name`属性。但这只是在那种情况下。如果我们要创建另一个实例，那么这个名字就是在`class`中定义的。

# 结论

我希望您对 JavaScript 中的作用域有更好的理解，并且知道如何使用它以及如何在您自己的代码中实现它！如果没有，请在评论中补充问题！我很乐意帮你解决这个问题！

![](img/a72eb48c81bbe92d2070b77f9724eb94.png)

大家好，我是荷兰🇳🇱 JavaScript 开发人员 **Ray** ，我很乐意分享我自 2009 年以来作为一名开发人员所获得的知识。我写关于 JavaScript、TypeScript、Angular 和任何与开发人员生活相关的东西的故事。

你可以在 [Twitter](https://twitter.com/devbyrayray) 和 [Instagram](https://www.instagram.com/devbyrayray/) 上关注我，或者[订阅我发布新故事时发送的简讯](https://buttondown.email/devbyrayray)。

*快乐编码🚀*

# 从我这里读更多

[](https://medium.com/better-programming/build-fast-json-powered-forms-on-angular-with-ngx-formly-b7a00733e66e) [## 使用 NGX Formly 在 Angular 上构建快速的、基于 JSON 的表单

### 表格可能是一场噩梦——让我们把它们变得更好

medium.com](https://medium.com/better-programming/build-fast-json-powered-forms-on-angular-with-ngx-formly-b7a00733e66e) [](https://medium.com/better-programming/you-dont-need-a-javascript-framework-df2a36c2dd0a) [## 你不需要 JavaScript 框架

### 有时反应，角，或 Vue.js 可能太多了

medium.com](https://medium.com/better-programming/you-dont-need-a-javascript-framework-df2a36c2dd0a) [](https://medium.com/better-programming/2-ways-to-resolve-duplication-in-javascript-arrays-and-objects-e377e1bdc5e1) [## 解决 JavaScript 数组和对象重复的两种方法

### 你知道如何处理重复吗？

medium.com](https://medium.com/better-programming/2-ways-to-resolve-duplication-in-javascript-arrays-and-objects-e377e1bdc5e1) [](https://medium.com/better-programming/7-steps-to-dockerize-your-angular-9-app-with-nginx-915f0f5acac) [## 使用 Nginx 对 Angular 9 应用程序进行分类的 7 个步骤

### 在 Docker 环境中设置 Angular 9 应用程序，并立即进行部署

medium.com](https://medium.com/better-programming/7-steps-to-dockerize-your-angular-9-app-with-nginx-915f0f5acac)