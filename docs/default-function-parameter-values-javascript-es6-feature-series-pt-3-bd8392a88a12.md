# 默认函数参数值:JavaScript ES6 特性系列(Pt 3)

> 原文：<https://itnext.io/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12?source=collection_archive---------5----------------------->

# 更少的参数未定义检查使开发人员的工作更容易

![](img/e16303c03ae70c3eb0a75a2ead60a55e.png)

照片由[🇸🇮·扬科·菲利— @specialdaddy](https://unsplash.com/@thepootphotographer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/search/photos/learning?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

# 介绍

这些帖子背后的灵感很简单:对于很多开发人员来说，JavaScript 毫无意义——或者至少有时令人困惑。

由于 JS 为 web 提供了如此多的功能，我想提供一些关于我经常使用的 JavaScript ES6 特性的文章，供其他开发人员参考。

我们的目标是让这些文章简短、深入地解释该语言的各种改进，我希望这些文章能启发您使用 JS 编写一些真正酷的东西。谁知道呢，在这个过程中你可能会学到一些新东西。😄

> 在我的第三篇文章中，我想谈谈默认的函数参数值:这些东西将会把我们从代码中大量未定义的检查中拯救出来。

# 传统函数参数

正如我之前的一篇文章所详述的，函数是 JavaScript 的基本构件之一。函数允许我们做的事情之一是定义一组语句，每次都以相同的方式执行任务或计算值，并提供一个可以传入进行操作的参数列表。

不管什么值作为参数传入，该函数将在每次调用时尝试运行相同的任务*。*

## 剖析带参数的函数声明

```
function simpleAdd(a, b){
  return a + b;
}simpleAdd(2, 3); // prints: 5
```

如果你在看上面的函数声明，下面是它的组成。`simpleAdd`是函数名，`a`和`b`是函数接受的两个参数，函数体`return a + b;`是语句。

好的，这很简单，直截了当，非常有意义，对吗？没错。但是，如果传统的函数声明没有传递正确数量的参数，会发生什么呢？

参数过多是可以的，JavaScript 引擎只知道忽略函数中不使用的额外参数，但是没有提供足够的参数是完全不同的问题……看看会发生什么。

## 函数声明的参数太多&太少

```
function add(a, b){
  return a + b;
}console.log(add(1, 2, 3, 4)); // prints: 3console.log(add(1)); // prints: NaN
```

啊哈，缺少一个`b`参数对于`add(1)`函数来说不是很好。`NaN`(“不是一个数字”)是开发人员在试图用 JavaScript 做数学运算时永远不想看到的。

为什么会这样？因为传统上，函数参数在声明函数时默认为`undefined`，在代码中调用函数时等待`arguments`传入。那么，我们如何避免类似`NaN`的可怕事情呢？在 ES6 之前，我们不得不在函数中到处做类似于`undefined`检查的事情。

## 检查函数的“未定义”参数

```
function addImproved(a, b) {
if(a === undefined){
  a = 3;
}
if(b === undefined){
  b = 7;
}
  return a + b;
}console.log(addImproved(6,9)); // prints: 15 console.log(addImproved()); // prints: 10
```

在上面的例子`addImproved()`中，为了避免`undefined`参数错误，我们必须在每个参数传入时进行物理检查，并确保在最终让函数达到它的主要目标:将两个数相加之前，它不是未定义的。

多么痛苦——想象一下，在一个庞大的代码库中，不得不为数百个函数一遍又一遍地编写相同类型的逻辑。😩不，谢谢，应该有更好的方法…

# 默认函数参数

进入[默认功能参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)；如果没有传入值或`undefined`，它们允许函数中的命名参数用默认值初始化。

上面我简单解释过，到目前为止，在 JavaScript 中，函数参数自动默认为`undefined`；这意味着我们必须测试函数体中的参数值，检查潜在的未定义参数，如果它们未定义，就赋值。

如果没有赋值，这里有两种设置值的方法。第一个是简单的`if / else`语句，第二个是三元语句，检查参数的`type`是否未定义，并在需要时设置值。

## 防止未定义参数的两种老方法

```
function oldSum (x, y, z) {
  if (y === undefined) {
    y = 7;
  }
  z = (typeof z !== 'undefined') ? z : 42;
  return `oldSum(${x}) === ${x + y + z}`;
};console.log(oldSum(1));  // prints: oldSum(1) === 50
console.log(oldSum(2, 5, 8)); // prints: oldSum(2) === 15
console.log(oldSum()); // prints: oldSum(undefined) === NaN
```

似乎有很多额外的代码只是为了在函数运行之前防止错误。使用 ES2015 中的默认参数，不再需要检查函数体。

## 使用默认参数防止未定义错误的 ES6 方法

```
function newSum (x, y = 7, z = 42) {
  return `newSum(${x}) === ${x + y + z}`;
};console.log(newSum(1)); // prints: newSum(1) === 50
console.log(newSum(3, 6, 9)); // prints: newSum(3) === 18
console.log(newSum(16, undefined)); // prints: newSum(16) === 65
```

好多了！在命名参数的地方，如果没有提供足够的参数，它们也会被赋予默认值。这样读起来更干净、更容易，并且去掉了未定义检查的样板文件。

我还应该提到，适用于函数声明中默认参数的规则也适用于箭头函数。看看这个。

## 箭头函数中的 ES6 默认参数

```
const thisWayWorksToo = (x = 7, y = 8) => {
  return x + y;
}console.log(thisWayWorksToo(3, 4)); // prints: 7console.log(thisWayWorksToo()); // prints: 15
```

这些类型的例子是默认参数最常见的用例，但是，如果您有兴趣的话，还有更多的东西需要了解。🤔

# 默认参数的陷阱和其他用例

默认参数还有一些其他有趣的功能，了解这些功能会很有帮助——它们还有一些非常方便的用例。让我们仔细检查一下，这样您就不会出错，或者当您出错时，您可以更快地调试问题的根源。

## 传递未定义的 vs 其他假值

有一件事可能会让你犯错误:将其他错误值如`null`或`'’`传递给带有默认参数的函数将会导致默认值被替换。

```
function luckyNumber(num = 11) { 
  console.log(typeof num);
};luckyNumber(); // prints: 'number' (num is set to 11)
luckyNumber(undefined); // prints: 'number' (num is set to 11 too)// test with other falsy values:luckyNumber(''); // prints: 'string' (num is set to '')
luckyNumber(null); // prints: 'object' (num is set to null)
```

正如你在上面看到的，当调用`luckyNumber()`而没有值或者值为`undefined`时，它回到默认参数 11。然而，当用空字符串或`null`值调用它时，它会取那个值。

## 呼叫时的评估

第二件要知道的事情是:默认值是在调用时计算的，所以每次调用函数时都会创建一个新的对象*而不是像你想的那样，添加到一个已经存在的对象或数组中。*

```
function append(value, array = []) {
  array.push(value);
  return array;
}console.log(append(1)); // prints: [1]
console.log(append(2)); // prints: [2], not [1, 2]
```

每次`append()`函数运行时，它都会创建一个全新的数组，数组中的内容是所提供的值。第一次调用和执行函数时，它会运行，生成一个数组，然后结束，一旦结束运行，它的执行上下文就会被销毁。

这意味着该函数现在已经完全使用了(并且不知道)它创建的原始数组。这就是为什么当第二次使用第二个值调用`append()`时，它会创建第二个全新的数组，而不是添加到它创建的第一个数组中。

## 较早的参数可用于较新的默认参数

好了，这里有一个很酷的技巧:参数字符串中前面(左边)声明的参数可以被后面的默认参数使用。

```
function welcome(name, greeting, message = greeting + ' ' + name) {
  return [name, greeting, message];
}console.log(welcome('Sean', 'Hi'));  // ["Sean", "Hi", "Hi Sean"]
console.log(welcome('Sean', 'Hi', 'Happy Birthday!'));  // ["Sean", "Hi", "Happy Birthday!"]
```

通过传入参数`name`和`greeting`的值，`message`的第三个参数能够将这两个值作为另一个字符串值。很可爱。

想象一下，你可以更干净地处理有很多潜在变量的函数。这个例子是我从 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters#Default_parameters_are_available_to_later_default_parameters)中借用的，关于默认参数，它真正说明了这一点。

## 使用和不使用默认参数的更清晰的边缘案例处理

```
function go() {
  return ':P';
}

function withDefaults(a, b = 5, c = b, d = go(), e = this, 
                      f = arguments, g = this.value) {
  return [a, b, c, d, e, f, g];
}

function withoutDefaults(a, b, c, d, e, f, g) {
  switch (arguments.length) {
    case 0:
      a;
    case 1:
      b = 5;
    case 2:
      c = b;
    case 3:
      d = go();
    case 4:
      e = this;
    case 5:
      f = arguments;
    case 6:
      g = this.value;
    default:
  }
  return [a, b, c, d, e, f, g];
}

withDefaults.call({value: '=^_^='});
// [undefined, 5, 5, ":P", {value:"=^_^="}, arguments, "=^_^="]

withoutDefaults.call({value: '=^_^='});
// [undefined, 5, 5, ":P", {value:"=^_^="}, arguments, "=^_^="]
```

是啊，有人会选择写出`withoutDefaults()`函数而不是`withDefaults()`函数吗？我认为不是。

## 默认参数后没有默认值的参数

虽然早期的参数可用于后来的默认参数，但您不能传入已定义的参数来填充函数中后来未定义的参数。很有道理，对吧？😉换句话说:参数总是从左到右设置的，即使后面有没有默认值的参数，它们也会覆盖默认参数。下面的例子可能更好地说明了这一点。

```
function f(x = 1, y) {
  return [x, y];
}console.log(f()); // prints: [1, undefined]console.log(f(4)); // prints: [4, undefined]
```

当涉及到函数中的默认参数时，这些是需要知道的一些主要事情。

# 结论

尽管 JS 已经存在了 20 多年，ES6 也已经在 2015 年问世，但围绕它仍然存在大量错误信息和知识差距。

我试图揭开它的神秘面纱，让你更好地理解 JavaScript 和 ES6——这些东西你可能每天都在使用，但并没有完全掌握它们的细微差别。

函数是 JavaScript 的关键构建块之一，在此之前，检查未定义的参数只是生活的一部分。不过，随着缺省参数的引入，至少可以去掉一些样板文件，使函数更容易编写、阅读和维护，我个人非常感激，并对未来的改进感到兴奋。

过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。

感谢您的阅读，我希望您有机会在以后编写的函数中利用默认参数，无论是声明、表达式还是箭头函数。如果你觉得有帮助，请与你的朋友分享！

**如果你喜欢读这篇文章，你可能也会喜欢我的其他一些博客:**

*   [箭头功能:JavaScript ES6 特性系列(Pt 2)](/arrow-functions-javascript-es6-feature-series-pt-2-e8c31c823392)
*   [Var，Let & Const: JavaScript ES6 特性系列(Pt 1)](/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e)
*   [同时运行多个 Node.js 或 NPM 命令的 4 种解决方案](/4-solutions-to-run-multiple-node-js-or-npm-commands-simultaneously-9edaa6215a93)

**参考资料和更多资源:**

*   函数，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Guide/Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
*   默认函数参数，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Functions/Default _ Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)