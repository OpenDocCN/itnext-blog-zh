# 析构数组和对象:JavaScript ES6 特性系列(第 10 部分)

> 原文：<https://itnext.io/destructuring-arrays-objects-javascript-es6-feature-series-pt-10-507108471c07?source=collection_archive---------0----------------------->

# 对于简洁的变量语法，花括号从来没有像现在这样重要

![](img/ca14f27a3ac8a68a2b4941368d2d8581.png)

帕特里克·罗伯特·道尔在 [Unsplash](https://unsplash.com/s/photos/learning?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

# 介绍

T 这些作品的灵感很简单:JavaScript，尤其是一些新的 ES6+版本，乍一看可能会让许多开发人员有些困惑。直截了当地说:当另一层旨在使我们的生活更容易的句法糖被添加进来时，他们曾经认为他们理解的东西变成了一个全新的野兽。

也就是说，根据维基百科的数据，在今天 1000 万个最受欢迎的网页中，95%仍然使用 JavaScript。

JavaScript 只会继续在 web 中扮演越来越重要的角色，我想提供一些我经常使用的 ES6+特性的文章和例子，供其他开发人员参考。

我们的目标是让这些文章简短、深入地解释对该语言的各种改进，我希望它们能启发您使用 JS 编写一些真正酷的东西。谁知道呢，在这个过程中你可能会学到一些新东西。😄

> 在本系列的最后一篇文章中，我将讨论数组和对象析构:这是不费吹灰之力将值和属性提取到单个变量中的最简洁的方法。

# 析构一开始听起来很简单…

ES 2015 中首次引入的[析构赋值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，是我最喜欢的 JavaScript 标准语法新增内容之一。正如我上面所说的，析构使得将数组中的值或者对象中的属性解包到不同的变量中成为可能。

当您第一次听到它时，这可能听起来很简单，但是实际上要将它付诸实施，特别是对于深度嵌套的数组或对象，就有点难掌握了。但是在我进入可能会使你出错的部分之前，让我们先讨论一下在两个数组中析构是什么样子的，以及最近的对象。

# 数组析构

在对象析构之前，ECMAScript 语言引入并最终确定了数组析构。

正如对象和数组文字表达式提供了一种创建数据包的简单方法，就像这样:

```
const a = ['alpha', 'beta', 'gamma', 'delta', 'epsilon'];
```

那些相同数组的析构赋值使用相似的语法，但是在赋值的左边定义从源变量中解包什么值。

*剖析数组析构语法*

```
let b, c, more;
const a = ['alpha', 'beta', 'gamma', 'delta', 'epsilon'];[b, c] = a;console.log(b); // 'alpha'
console.log(c); // 'beta'[b, c, ...more] = ['alpha', 'beta', 'gamma', 'delta', 'epsilon'];
console.log(more); // [ 'gamma', 'delta', 'epsilon' ]
```

上面的简短例子演示了数组析构和使用 rest 模式在名为`a`的数组上分配额外的值，这是我在这里关于[写的](/spread-rest-parameters-javascript-es6-feature-series-pt-4-c9e9f0c0228f?source=friends_link&sk=9ff75e9781a0b55ee4572ec02f2f02c1)。

注意，变量`b`和`c`被括在括号(`[b, c]`)中，并被赋值为等于`a`数组。`[b, c] = a;`

当这些值被调用时，它们将接受数组中的第一个和第二个元素。`console.log(b); // 'alpha'`和`console.log(c); // 'beta';`

rest 模式带有变量`more`。该变量被设计为在将变量添加到数组中时，通过将 rest 语法(`...more`)放在变量名称的前面来继承`a`数组中所有剩余的值。然后，当调用`more`时，它打印出自己数组中的其余值。`console.log(more); // ['gamma', 'delta', 'epsilon']`。

好了，让我们来看看数组析构的不同用法。

## 基本变量赋值

最基本的数组析构是取一个数组的值，并将这些值赋给等量的新变量。

*基本数组变量析构赋值示例*

```
const govtBranches = [ 'executive', 'judicial', 'legislative' ];const [branch1, branch2, branch3] = govtBranches;console.log(branch1); // 'executive'
console.log(branch2); // 'judicial'
console.log(branch3); // 'legislative'
```

`govtBranches`的三个值中的每一个都有一个对应的变量，该变量包含在数组括号`branch1`、`branch2`或`branch3`中。

然后，如果这些变量中的任何一个被调用，它们将反映来自`govtBranches`数组的一个单独的值。

## 赋值与声明分离

接下来是什么时候可以先声明一个变量(或者使用关键字`let`或者`var`，而不是`const`)，然后通过析构为其赋值。

*变量声明和析构赋值的例子*

```
let agency1, agency2;[agency1, agency2] = [ 'FBI', 'CIA' ];console.log(agency1); // 'FBI'
console.log(agency2); // 'CIA'
```

如你所见，变量`agency1`和`agency2`首先被声明为未定义变量。接下来，它们被放在数组括号中，并被赋给包含值`'FBI'`和`'CIA'`的数组。从那里，每个变量可以被单独调用，它代表数组中的一个值。

## 默认值

有趣的是，你可以给变量分配一个默认值，类似于[默认函数参数值](/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12)，以防从数组中解包的值是`undefined`。

当创建的变量比数组中的值多时，就会发生这种情况。

*数组析构中的默认值示例*

```
let one, two;
[one='a', two='b'] = ['c'];console.log(one); // 'c'
console.log(two); // 'b'
```

两个变量`one`和`two`在被分配给包含单个值`'c'`的数组之前被分配默认值`'a'`和`'b'`。

当变量`one`在数组析构发生后被调用时，由于数组中的值，其值被覆盖为`'c'`。因为数组不包含第二个值，所以`two`的值不会改变，所以那个值，如果存在的话，应该是`undefined`。

## 交换变量

你知道两个变量值可以在一次析构赋值中交换吗？它们可以，而且很方便的是，你不再需要一个临时变量来实现这一点。

*使用数组析构交换变量值的示例*

```
let boots = 'cat';
let rocky = 'dog';[boots, rocky] = [rocky, boots];console.log(boots); // 'dog'
console.log(rocky); // 'cat'
```

最初，变量`boots`是一个`'cat'`而变量`rocky`是一个`'dog'`，但是简单地通过交换数组中它们被赋值的值的顺序，它们的值可以被交换，因此`boots`变成了`'dog'`而`rocky`变成了`'cat'`。

在某些情况下，这是一个非常有用的技巧。

## 解析从函数返回的数组

从一个函数中获取一个数组并不是什么新鲜事，但是现在你可以析构被返回的值以使处理它们更加简洁。

*解析从函数*返回的数组的示例

```
function color() {
  return ['red', 'yellow', 'green', 'blue']
}let r, y, g;
[r, y, g] = color();console.log(r); // 'red'
console.log(g); // 'green'
```

函数的作用是:返回一组颜色。通过对函数破坏变量`r`、`y`、`g`，数组中的这些值被分配给这些变量。

## 忽略一些返回值

同样地，解构可以让您忽略您不感兴趣的某些阵列值。

*忽略具有破坏的数组中的值的示例*

```
function ignoreColor() {
  return ['indigo', 'orange', 'lime']
}const [i, ,l] = ignoreColor();console.log(i); // 'indigo'
console.log(l); // 'lime'
```

只需在被破坏的数组中添加一个空白，就可以选择不从函数`ignoreColor()`返回值`'orange'`。

如果你愿意，你也可以选择忽略函数中的所有值(尽管我并没有看到太多这样的用例)。

```
[ , , ] = ignoreColor();
```

## 将数组的其余部分分配给变量

我再次回到我在第一个数组销毁示例中展示的内容:使用 rest 运算符(`...`)从数组中拾取任何剩余值。

*通过数组破坏为变量分配剩余值的示例*

```
const [commanderInChief, ...staff] = ['President', 'Vice President', 'Chief of Staff', 'Press Secretary'];console.log(commanderInChief); // 'President'
console.log(staff); // [ 'Vice President', 'Chief of Staff', 'Press Secretary' ]
```

和前面一样，`commanderInChief`获取数组中的第一个值，通过使用 rest 语法，`...staff`将数组中的所有剩余值作为自己的新数组。

就这么简单。现在让我们来看看如何在对象上进行破坏。

# 对象破坏

对象析构的方法类似于数组析构，只是可以从对象中取出属性(键)及其值，而不是从数组中取出值。

下面是一些例子来说明这种情况。

## 基本对象破坏分配

我再一次从最基本的例子开始，说明对象如何进行破坏。

*基本对象破坏示例*

```
const pieIngredients = { pumpkin: '1 can', pieCrust: '1 crust', spice: '2 tsp'};const { pumpkin, pieCrust, spice} = pieIngredients;console.log(pumpkin); // '1 can'
console.log(pieCrust); // '1 crust'
console.log(spice); // '2 tsp'
```

通过将属性环绕在对象`pieIngredients`中并将其设置为等于对象，每个属性`pumpkin`、`pieCrust`和`spice`都成为其自己的变量，并且附加到它的值成为新变量的值。

## 无声明转让

变量也可以使用与其声明分开的析构来分配其值，就像使用数组析构一样。

*单独声明变量后分配变量示例*

```
let hobby, sports;({hobby, sports} = {hobby: 'knitting', sports: 'croquet'});console.log(hobby); // 'knitting'
console.log(sports); // 'croquet'
```

请注意，当使用不带声明的对象字面析构赋值时，赋值语句周围的括号`( ... )`是必需的。

否则，语法被认为是无效的，因为左边的语法`{hobby, sports}`被认为是一个块，而不是一个对象文字。不过，将整行用括号括起来，可以澄清意图并使其有效。

## 分配给新的变量名

有一点很有帮助，那就是可以将一个属性从一个对象中解包并赋给一个与对象属性不同名称的变量。

*将析构对象属性重新分配给新变量名的例子*

```
const car = {speed: 110, color: 'red'};
const { speed: fast, color: cherry } = car;console.log(fast); // 110
console.log(cherry); // 'red'
```

例如，在这里，`const {speed: fast} = car`从对象`car` 获取名为`speed`的属性，并将其赋给名为`fast`的局部变量。

## 默认值

就像数组析构一样，析构对象的变量可以被赋予一个默认值，在这种情况下，从对象解包的值是`undefined`。

*向对象析构变量分配默认值的示例*

```
const { redWine = 'cabernet', whiteWine = 'pinot grigio'} = { redWine: 'malbec'};console.log(redWine); // 'malbec'
console.log(whiteWine); // 'pinot grigio'
```

在本例中，变量`redWine`和`whiteWine`被赋予默认值`'cabernet'`和`'pinot grigio'`。然后`redWine`变量被重新赋值为`'malbec'`的值，但是因为`whiteWine`没有在被析构的对象中定义，所以它保留了原来的值。

## 从作为函数参数传递的对象中解包字段

对象析构的另一个特性是，你可以在函数调用的中使用析构语法*来取回那些值。看看这个。*

*将析构对象属性作为函数参数传递的示例*

```
const girl = {
  name: 'Paige',
  age: 30,
  eyeColor: 'blue',
  hair: {
    type: 'curly',
    color: 'red',
    length: 'shoulder-length'
  }
}const getUserName = ({name}) => {
  return {name};
}console.log(getUserName(girl)); // { name: 'Paige' }const getUserHair = ({hair: {type, color}}) => {
  return `Her hair is ${color} and ${type}`;
}console.log(getUserHair(girl)); // Her hair is red and curly
```

在这里的例子中，对象`girl`是一个非常标准的对象。它有两层嵌套属性，但除此之外，它并不起眼。

需要注意的是两个功能`getUserName()`和`getUserHair()`。您将看到传递给它的函数参数实际上是来自它接收的对象的属性`name`的析构版本。

所以当整个`girl`对象被传递给函数时，它只返回`name`的属性和值作为函数的输出。

第二个函数`getUserHair()`更加有趣，因为它试图访问的值实际上位于传递给该函数的对象的下两层，所以首先必须访问`hair`的属性，然后才能访问`hair`特有的属性，即`type`和`color`。

当用对象`girl`调用该函数时，该函数将返回一个字符串，说明对象的头发颜色和头发类型作为输出。

这也是一个如何使用析构访问嵌套对象的例子，我接下来会谈到。

## 嵌套对象析构

这是一个让我花了一些时间来理解的话题(老实说，当我想析构多层嵌套对象时，我通常还得回头再看一遍文档。

基本要点是:如果你的对象在你的对象中不止一层，你必须首先访问它的父属性，它的父属性，等等，直到你到达最外层的对象属性值。

*深度嵌套对象析构的例子*

```
const girl = {
  name: 'Paige',
  age: 30,
  eyeColor: 'blue',
  hobbies: {
    primaries: [
      {
        mostFavorite: [
          'drawing',
          'art'
        ],
        frequentlyDone: 'cooking',
        relaxing: {
          reading: 'fictionBooks'
        }
      }
    ]
  }
}const {
  hobbies: {
      primaries: [
      {
        mostFavorite
      }
    ]
  }
} = girlconsole.log(mostFavorite[0]); // 'drawing'
console.log(mostFavorite[1]); // 'art'const {
  hobbies: {
    primaries: [
      {
        relaxing: {
          reading
        }
      }
    ]
  }
} = girlconsole.log(reading); // // 'fictionBooks'
```

我使用了前一个例子中的相同的`girl`对象，但是我将`hobbies`的属性添加到了该对象中，并在其中添加了一些新的数组和对象，这样我就可以展示如何从中提取值。

我创建的第一个新对象拉出了`mostFavorite`的嵌套对象属性，恰好是一个有两个值的数组。为了达到这些值，首先，我必须将`girl`的最外层属性,`hobbies`用花括号括起来。接下来，我必须包装`primaries`的`hobbies'`属性。然后，我必须深入到数组和`primaries`包含的对象中，以到达实际保存我所寻找的值的`mostFavorite`属性。

从这里开始，很容易就可以记录下`mostFavorite`的任何值。

同样，要获得深埋在`girl`对象中的属性`reading`的值，我必须再次开始用花括号将`hobbies`括起来，然后继续到`primaries`，进入数组并找到`relaxing`的对象属性，最后将属于`relaxing`的父对象的属性`reading`括起来。

然后，我可以简单地调用`reading`作为它自己的变量，并获取嵌套在`girl`对象中的值。

这肯定需要一些练习才能掌握，但是看看获得这些值所需的语法比以前少了多少。

这么长的`const reading = girl.hobbies.primaries[0].relaxing.reading;`，我不会错过的。

如果您想了解更多关于嵌套对象析构的内容，几个月前我专门写了另一篇文章，介绍了在值不可用时避免未定义错误的方法。这是到它的链接。😄

## 对象析构中的 Rest 语法

最后一个例子，在我写这篇文章的时候，还在 ECMAScript 第 4 阶段的提议中，我可能会加上:rest 语法加上对象析构。

我在第一个数组析构演示中展示了这一点，但是我还没有在对象析构中展示过。事实证明，它对对象和数组的作用是一样的。

*使用 rest 语法进行对象析构的示例*

```
let myObjectOfNums = {
  ex: 'ten',
  why: 'twnety',
  zed: 'thirty',
  dee: 'forty',
  ee: 'fifty'
}let {ex, why, zed, ...allOthers} = myObjectOfNums;console.log(ex); // 'ten'
console.log(why); // 'twenty'
console.log(zed); // 'thirty'
console.log(allOthers); // { dee: 'forty', ee: 'fifty' }
```

看看从对象`myObjectOfNums`中提取前三个属性及其值，以及使用 rest 参数将其他属性保存在一个名为`allOthers`的新对象中是多么容易？

其他的析构规则在这里仍然适用。如果您想将变量名从`ex`或`why`更改为`a`和`b`，您可以像之前一样操作。

*用 rest 语法析构对象和变量再分配的例子*

```
let { ex: a, why: b, zed, ...allOthers } = myObjectOfNums;console.log(a); // 'ten'
console.log(b); // 'twenty'
console.log(zed); // 'thirty'
console.log(allOthers); // { dee: 'forty', ee: 'fifty' }
```

这也是完全正确的。很酷吧。😃

# 结论

数组和对象析构在 Perl 和 Python 等语言中已经存在了很长时间，但直到 ECMAScript 2015，JavaScript 才开始在这一领域获得一些平等。

这种利用花括号的新语法使得用非常简洁的代码轻松访问数组中的单个变量甚至对象成为可能。我是它的忠实粉丝。

本系列的目标是研究您每天使用但可能不知道所有微妙和细微差别的 ES6 语法，这样您可以成为更好的 web 开发人员。

过几周再来看看，我会写更多关于 JavaScript 和 ES6 或其他与 web 开发相关的东西，所以请关注我，这样你就不会错过了。

感谢您的阅读，我希望我已经把数组和对象析构弄得更清楚了，也希望您能在自己的代码库中尝试一下。

如果你喜欢读这篇文章，你可能也会喜欢我在这个系列中的其他作品:

*   [对象键、值和条目:JavaScript ES6 特性系列(Pt 9)](https://medium.com/better-programming/object-keys-values-and-entries-javascript-es6-feature-series-part-9-d71268791089)
*   [类和继承:JavaScript ES6 特性系列(Pt 8)](/classes-and-inheritance-javascript-es6-feature-series-part-8-4a81fa3adf0f)
*   [内置模块导入导出:JavaScript ES6 特性系列(Pt 7)](https://medium.com/better-programming/built-in-module-imports-and-exports-javascript-es6-feature-series-part-7-5f0864049e1f)
*   [增强的对象文字值速记:JavaScript ES6 特性系列(Pt 6)](/enhanced-object-literal-value-shorthand-javascript-es6-feature-series-pt-6-e00dfdc24f64)
*   [字符串模板文字:JavaScript ES6 特性系列(Pt 5)](https://medium.com/better-programming/string-template-literals-javascript-es6-feature-series-pt-5-a40e55a5485b)
*   [Spread & Rest 参数:JavaScript ES6 特性系列(Pt 4)](/spread-rest-parameters-javascript-es6-feature-series-pt-4-c9e9f0c0228f)
*   [默认函数参数值:JavaScript ES6 特性系列(Pt 3)](/default-function-parameter-values-javascript-es6-feature-series-pt-3-bd8392a88a12)
*   [箭头功能:JavaScript ES6 特性系列(第二部分)](/arrow-functions-javascript-es6-feature-series-pt-2-e8c31c823392)
*   [Var，Let & Const: JavaScript ES6 特性系列(Pt 1)](/var-let-const-javascript-es6-feature-series-pt-1-fa603567809e)

# 参考资料和更多资源

*   Destructuring assignments，MDN docs:[https://developer . Mozilla . org/en-US/docs/Web/JavaScript/Reference/Operators/destructing _ assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)