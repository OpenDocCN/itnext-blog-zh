# Spread & Rest 参数:JavaScript ES6 特性系列(第 4 部分)

> 原文：<https://itnext.io/spread-rest-parameters-javascript-es6-feature-series-pt-4-c9e9f0c0228f?source=collection_archive---------2----------------------->

# 语法太好了，ES6 用了两次

![](img/cf81030f5bebfb47f5a4a0f8e014249f.png)

由[威廉·艾文](https://unsplash.com/@firmbee?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/learning?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

# 介绍

这些帖子背后的灵感很简单:对于很多开发人员来说，JavaScript 没有任何意义——因为它的异步行为和解释语言等等。

再加上 ECMAScript 委员会发布的年度更新，有很多信息需要及时了解。🤯看看维基百科下面的统计数据。

> 截至 2017 年 5 月，1000 万个最受欢迎的网页中有 94.5%使用了 JavaScript。— JavaScript，维基百科

由于 JS 对 web 的贡献如此之大，我想提供一些我经常使用的 ES6+特性的文章和例子，供其他开发人员参考。

我们的目标是让这些文章简短、深入地解释该语言的各种改进，我希望这些文章能启发您使用 JS 编写一些真正酷的东西。谁知道呢，在这个过程中你可能会学到一些新东西。😄

> 在我的第四篇文章中，我将讨论 rest 参数和 spread 语法:轻松压缩或扩展参数、元素和对象值的最简洁的方法。

# …休息参数

在深入研究 rest 参数之前，我建议您熟悉 JavaScript 中目前使用的三种主要函数类型:函数声明、函数表达式和箭头函数。为了快速复习，我写了一篇关于他们的博文。

现在我们在同一页上，让我们进入到底什么是 rest 参数。

[Rest 参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)是一种新的语法，它允许我们将传递给函数的不定数量的参数表示为数组。有道理吗？不完全是？🤔没问题，代码示例通常也能帮助我更容易地理解概念。

## 解析函数声明中的 Rest 参数

```
function f (x, y, ...a) {
  return (x + y) * a.length;
}console.log(f( 1, 2, “hello”, true, 7)) // prints: 9
```

当你看上面的例子时，除了最后一个参数声明的，参数`a`前面有一个可疑的`...`，其他的都和普通的函数声明*一样。这是一个 rest 参数的例子。*

这意味着当函数执行时，“将传递给该函数的所有剩余参数都放入一个数组中，并按照函数体在其语句体中的指示来处理它们。”

这是另一个例子，但是这次使用了一个箭头函数(也很容易转换成非 ES6 语法中的函数表达式)。

## 箭头函数中的其余参数

```
const product = (e, f, ...g) => {
  return e * f + g.length; 
}console.log(product(4, 7, 2, 6, 3)); // prints: 31
```

现在，您可能对如何使用 rest 参数有一些疑问，(我知道我肯定有)，所以让我为您列出规则。

# Rest 参数规则、特征和用途

## 只有最后一个参数可以有…

不，我没有失去我的思路，我说的是:只有函数中定义的最后一个参数，可以加上前缀`...`，使它成为一个 rest 参数。

当最后一个参数以`...`为前缀时，所有剩余的(用户提供的)参数都放在一个“标准”Javascript 数组中。

**输出带有 Rest 参数的自变量**

这是当您将 rest 参数作为函数调用中的最后一个参数时，您将看到的输出。如您所见，传递给`myNumberFunction`、`1`、`2`和`3`的前三个参数被单独打印出来，作为函数中定义的前三个参数:`a`、`b`和`c`。然而，传递给 rest 参数`d`的所有剩余数字都作为数组`[4, 5, 6, 7, 8]`打印出来。

```
function myNumberFunction(a, b, c, ...d) {
  console.log("a", a);  // a 1
  console.log("b", b); // b 2
  console.log("c", c); // c 3
  console.log("...d", d); //...d [ 4, 5, 6, 7, 8]
}myNumberFunction(1, 2, 3, 4, 5, 6, 7, 8);
```

## Rest 参数不仅仅是 Arguments 对象

在`arguments`对象和新的 rest 参数之间有三个主要区别需要了解。

*   rest 参数只是那些没有被单独命名的参数(即在函数表达式中正式定义的)，而`arguments`对象包含传递给函数的所有参数。
*   `arguments`对象不是一个真正的数组，其余的参数都是`Array`实例，也就是说`sort()`、`map()`、`forEach()`或`pop()`等方法可以直接应用在上面。
*   `arguments`对象具有特定于自身的附加功能(如`callee`属性)。

最初引入 Rest 参数是为了减少由`arguments`引起的样板代码。之前，将`arguments`转换为“普通数组”是一大痛苦。现在，它们可以被简单地传入，然后被执行。

**对其余参数数组的简单操作**

```
function doubleUp(...simpleArgs){
  const arr = simpleArgs;
  const secondArr = arr.map(num => num * 2);
  return secondArr;
}console.log(doubleUp(2, 6, 12, 18)); // [ 4, 12, 24, 36 ]
```

## Rest 参数可以被析构

关于 ES6，我最喜欢的新东西之一是数组和对象析构。我还没有详细介绍析构，但是在这一系列文章结束之前，我会介绍的。

简而言之，析构使得将数组中的值或对象中的属性解包到不同的变量中成为可能。如果你同时渴望了解更多，这里有一个[链接](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)让你更熟悉析构是如何工作的。

继续讨论这篇文章的目的，这里是 rest 参数和析构在起作用。

**用 Rest 参数析构数组**

```
const [captain, coCaptain, ...devs] = [ “Bridget”, “Joe”, “Kyle”, “Drew”, “Patrick”, “Francisco”];console.log(captain); // ‘Bridget’
console.log(coCaptain); // ‘Joe’
console.log(devs); // [ ‘Kyle’, ‘Drew’, ‘Patrick’, ‘Francisco’ ]
```

在上面声明的变量数组`captain`、`coCaptain`、`devs`中，其余的参数应用于`devs`，所以当数组名传入时，`'Bridget'`变成了`captain`、`'Joe'`是`coCaptain`、`devs`是`[ 'Kyle', 'Drew', 'Patrick', 'Francisco' ]`。

得心应手，对吧？想象一下您遇到的场景，您要么需要取出数组的各个部分，要么需要将其他部分放在一起。相信我，当你这么做的时候，你会非常感谢析构和休息参数。

同样的事情也可以用对象析构来完成。

**使用剩余参数进行对象析构**

```
const {pm1, pm2, ...restOfTeam} = {
  pm1: "Jeremy",
  pm2: "Tung",
  developer1: "Casey",
  developer2: "Mark",
  ux: "Christina"
};console.log(pm1); // Jeremy
console.log(pm2); // Tung
console.log(restOfTeam); // { developer1: 'Casey', developer2: 'Mark', ux: 'Christina' }
```

本例从对象中的其余团队成员中解构出`pm1`和`pm2`，其余的对象属性:`developer1`、`developer2`和`ux`被组合在一个名为`restOfTeam`的新对象变量中。

再一次，考虑使用这种类型的语法来创建新的变量、对象，将对象属性分成单独的部分，等等。

这些是您需要了解的关于 rest 参数的主要内容。至此，我们可以继续讨论 JavaScript 中`...`的另一种用法，即 spread 语法。

# 展开语法…

虽然它与 rest 参数非常相似，但是 spread 语法有着完全不同的用途。

[扩展语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)允许在需要零个或多个参数(用于函数调用)或元素(用于数组文字)的地方扩展一个可迭代对象，或者在需要零个或多个键值对(用于对象文字)的地方扩展一个对象表达式。

呃…现在怎么办？

好吧，这个定义听起来很专业，我知道，相信我。但这在实践中意味着什么:

> 如果你有一个字符串、一个数组或一个对象，并且你想使用所有的值，你可以用一个非常简洁的语法把所有的值*分散到函数调用、新数组或新对象中。*

## 函数、数组、字符串和对象中的展开语法

```
// For function calls:
function multiply(x, y, z) {
  return x * y * z;
}
const args = [1, 2, 3];
console.log(multiply(...args)); // 6// For array literals or strings:
const iterableObj = [ {protein: "steak"}, {carb: "potato"}, 'milkshake'];
const randomList = [...iterableObj, '4', 'five', 6];
console.log(randomList); // [ { protein: 'steak' }, { carb: 'potato' }, 'milkshake', '4', 'five', 6 ]const str = "foo"
const chars = [ ...str ] 
console.log(chars); // [ "f", "o", "o" ]// For object literals (new in ECMAScript 2018):
const powerTool = { skuNumber: ‘996655’, skuDescription: ‘Drill Bit’ };let secondPowerTool = { ...powerTool, toolDepartment: 25, toolClass: 7 };
console.log(secondPowerTool); // { skuNumber: ‘996655’, skuDescription: ‘Drill Bit’, toolDepartment: 25, toolClass: 7 }
```

上面不同的例子显示了当你分散不同的物品时会发生什么。

对于第一个例子，即`multiply()`函数，我将我的`args`数组扩展到函数调用中，该函数调用简单地从数组中取出三个值(`1`、`2`和`3`)，并将它们放在函数接受的参数`x`、`y`和`z`的位置。

在第二个和第三个例子中，扩展数组和字符串，原始列表`iterableObj`和原始字符串`"foo"`都被分别扩展到新变量`randomList`和`chars`中。

`iterableObj`中的所有值与专门分配给`randomList`的新值一起被添加到新的`randomList`数组中。并且原来的字符串`"foo"`被一个字符一个字符地分散到新的`chars`数组中。

最后一个例子，自 ECMAScript 2018 发布以来唯一可能的，是将对象`powerTool`的属性扩展到一个名为`secondPowerTool`的新对象中，并向该对象添加两个新属性:`toolDepartment`和`toolClass`。

# 传播语法规则、特征和用途

就像 rest 参数一样，spread 语法是一种语法糖，它取代了复杂的方法和充满样板代码的做事方式，而这些应该是相对容易的。

也就是说，你需要知道一些关于传播的事情。

## 函数调用中的 Spread 替换 Apply

到目前为止，在您想要使用数组元素作为函数参数的情况下，使用`Function.prototype.apply()`是很常见的。

**在函数调用**中使用 `**apply()**` **给数组元素赋值**

```
function myFunction(x, y, z) { }
var args = [0, 1, 2];
myFunction.apply(null, args);
```

有了 spread，语法就干净多了。

**将数组元素列表展开成函数调用**

```
function myFunction(x, y, z) { }
var args = [0, 1, 2];
myFunction(...args);
```

还有一件很酷的事情:参数列表中的任何参数都可以使用 spread 语法，并且可以多次使用。

**单个函数调用中的多次扩展**

```
function myFunction(v, w, x, y, z) { }
var args = [0, 1];
myFunction(-1, ...args, 2, ...[3]);
```

## 扩展语法可以与 new 关键字一起使用

不像使用`new`调用构造函数时，不能直接使用数组，而使用`apply`，使用`new`可以很容易地使用数组。

**展开语法同** `**new**` **构造函数**

```
var dateFields = [1989, 3, 13];
var d = new Date(...dateFields);console.log(d); // 13 Apr 1989
```

## 在数组文字中展开

Spread 使得数组操作变得更加容易，像`push()`、`concat()`和`splice()`这样的方法也变得不那么必要了。让我们来看看它能帮你做什么。

**创建新的数组文字很容易**

使用现有数组作为其一部分来创建新数组，不需要像过去那样使用额外的数组方法。

**将一个数组扩展到另一个新数组**

```
const fruits = ['watermelon', 'peaches'];
const fruitBasket = ['apples', 'grapes', ...fruits, 'bananas', 'kiwis', 'mango'];console.log(fruitBasket); // [ 'apples', 'grapes', 'watermelon', 'peaches', 'bananas', 'kiwis', 'mango' ]
```

将`fruits`数组添加到`fruitsBasket`有多简单？超级简单。

**复制数组也是轻而易举的事情**

使用 spread，将一个数组的值复制到另一个数组是小菜一碟。然后，您可以继续修改新复制的数组，而不会影响原始数组(这非常适合不可变的函数式编程风格，这种风格在 React 等框架和 Redux 等状态管理工具中非常流行)。

```
const arr1 = [1, 2, 3];
const arr2 = [...arr1];console.log(arr2); // [1, 2, 3]arr2.push(4);console.log(arr1); // [1, 2, 3]
console.log(arr2); // [1, 2, 3, 4]
```

在我把`4`推到`arr2`之前，当数值打印出来的时候，和`arr1`一模一样。同样，即使在`4`添加到`arr2`之后，`arr1`也保持不变。

**注意:** Spread 语法仅在复制数组时才有效。因此，它可能不适合复制多维数组，因为它使用了类似 lodash 的[这样的函数。推荐 cloneDeep()](https://lodash.com/docs/4.17.15#cloneDeep) 。

**串联数组从未如此简单过**

`[Array.prototype.concat()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)`是将一个数组连接到现有数组末尾时的首选。如果没有扩展语法，这将按如下方式完成:

`**.concat()**` **数组组合在一起**

```
const germanCars = [ 'BMW', 'Audi', 'Mercedes' ];
const japaneseCars = [ 'Honda', 'Toyota', 'Datsun' ];
const concatCarMakers = germanCars.concat(japaneseCars);console.log(concatCarMakers); // [ 'BMW', 'Audi', 'Mercedes', 'Honda', 'Toyota', 'Datsun' ]
```

使用扩展语法，这变成:

**将阵列分散在一起**

```
const germanCars = [ 'BMW', 'Audi', 'Mercedes' ];
const japaneseCars = [ 'Honda', 'Toyota', 'Datsun' ];
const carMakers = [...germanCars, ...japaneseCars];console.log(carMakers); // [ 'BMW', 'Audi', 'Mercedes', 'Honda', 'Toyota', 'Datsun' ]
```

Spread 语法也可以取代`unshift()`，使得在数组前面添加新元素比以前简单得多。

**将元素分散到数组的开头**

```
let numbers = [ 6, 5, 4 ];
const moreNumbers = [ 1, 2, 3 ];
numbers = [...moreNumbers, ...numbers];console.log(numbers); // [ 1, 2, 3, 6, 5, 4 ]
```

## 在对象文字中传播

截至 ECMAScript 2018，spread 属性来到了[对象文字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)。这允许对象将其自身的可枚举属性从提供的对象复制到新的对象上。

**浅层克隆和合并对象很简单**

虽然`Object.assign()`使得浅层克隆和合并对象成为可能，但是使用 spread 语法更加简洁。

```
// duplicate object properties
const markerSet = { copicMarkers: ['green', 'blue', 'red']};
const duplicateMarkerSet = {...markerSet};console.log(duplicateMarkerSet); // { copicMarkers: [ 'green', 'blue', 'red' ] }// merge two objects into a new one
const markerSet2 = { copicSketchMarkers: ['pink', 'yellow', 'orange']};
const giantMarkerSet = { ...markerSet, ...markerSet2 };console.log(giantMarkerSet); // { copicMarkers: [ 'green', 'blue', 'red' ], copicSketchMarkers: [ 'pink', 'yellow', 'orange' ] }
```

还值得注意的是，`Object.assign()`触发了`setters`，它将一个对象属性绑定到一个函数，当试图设置该属性时，该函数将被调用，而 spread 语法则没有。

## Spread(除了对象属性之外)仅适用于 Iterables

扩展语法(除了在扩展属性的情况下)只能*应用于*可迭代对象:数组、映射、集合、字符串等等。

```
// spreading in an object to an array does NOT work
const obj = { key1: 'value1'};
let array = [...obj];console.log(array); // []// spreading an array of objects into another array does work
const objInArray = [ { key2: 'value2'}];
array = [...objInArray];console.log(array); // [ { key2: 'value2' } ]
```

这就是为了有效地使用它，您需要了解的 spread 语法细节的大致内容。

# 结论

乍一看，一些最新的 JS 语法似乎完全陌生——即使对于已经编写 JavaScript 代码多年的人来说也是如此。虽然是的，它是不同的，但它也非常强大，使我们的工作比几年前容易得多。

我这个博客系列的目的是解释一些日常使用的 JavaScript 和 ES6 语法，并向您展示如何充分利用 JavaScript 语言的最新部分。

Rest 语法看起来与 spread 语法(`…`)完全一样，但是用于析构数组和对象。在某种程度上，rest 语法与 spread 语法相反:spread 将一个数组“扩展”成它的元素，而 rest 收集多个元素并将它们“压缩”成一个元素。它在许多常见的编程环境中非常有用，我相信您已经可以想象到了。

过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。

感谢您的阅读，我希望您将来会开始看到在代码中使用 rest 参数和 spread 语法的可能性——它们让无数事情变得轻而易举。如果你觉得有帮助，请与你的朋友分享！

**如果你喜欢读这篇文章，你可能也会喜欢我的其他博客:**

*   [默认函数参数值:JavaScript ES6 特性系列(Pt 3)](/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12)
*   [箭头功能:JavaScript ES6 特性系列(Pt 2)](/arrow-functions-javascript-es6-feature-series-pt-2-e8c31c823392)
*   [Var，Let & Const: JavaScript ES6 特性系列(Pt 1)](/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e)

**参考资料和更多资源:**

*   Rest 参数，MDN 文档:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Functions/rest _ Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
*   Spread 语法，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Operators/Spread _ Syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
*   JavaScript，维基百科:[https://en.wikipedia.org/wiki/JavaScript#Use_in_Web_pages](https://en.wikipedia.org/wiki/JavaScript#Use_in_Web_pages)
*   destructing，MDN Docs:[https://developer . Mozilla . org/en-US/Docs/Web/JavaScript/Reference/Operators/destructing _ assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)