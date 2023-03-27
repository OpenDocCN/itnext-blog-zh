# 什么是估值师？

> 原文：<https://itnext.io/what-is-valuer-2a73a685864?source=collection_archive---------6----------------------->

![](img/df5f10709456b1a7bf68f56f6d192004.png)

在[我的上一篇文章](https://medium.com/@parzhitsky/undefined-youre-using-it-wrong-39798a7f3742)中，我简要提到了 [Valuer](https://www.npmjs.com/package/@valuer/main) ，它是在实际使用之前验证价值的一个很好的工具。现在，我认为全面介绍它并简要介绍它的特性会很好。

# **快速入门**

先说个例子。假设，你想限制一些数字是自然的。太经典了。任务是返回符合自然数定义的输入，否则抛出一个描述错误的错误。

通常你会使用`if .. else`语句和`return` `throw`子句来完成这类任务:

```
if (typeof value !== "number" || isNaN(value))
    throw new Error("value is not a number");else if (value < 0)
    throw new Error("value is a negative number");else if (value % 1)
    throw new Error("value is not an integer");else return value;
```

我个人一直在做这种事情。这基本上是一段很好的代码，但是仍然存在一些问题:

*   它不是自我描述的；这里没有一个字是关于自然数的；您可能需要添加一两个注释，或者更好地将它包装到一个具有良好描述性标识符的函数中；
*   阅读这些代码的开发人员被迫从实现的角度来思考，而不是从问题的角度来思考；
*   开发人员在使用这种方法时，实际上可能会也可能不会将提供的值包含到出错时的控制台输出中，您只能期待最好的结果；
*   当一个错误发生时，并不立即清楚它意味着什么:你看到类似于*“0.3 不是整数”*的东西，但它并没有真正的帮助；
*   为了保持控制台输出的整洁和可读性，您可能想要标准化错误消息的格式，这很难实现，因为这通常不是团队的主要任务；
*   不同的开发人员可能更喜欢不同的方法来创建相同的条件——例如，使用`Number.isInteger`而不是`%`操作符——这不一定是一件坏事，但却是另一个不稳定的车轮。

好是最好的敌人。不是说*“值必须是不小于零且能被 1 整除的原数”*你实际想表达的是*“值必须是自然数*。*那不是简单多了吗？*

*是的，它是。*

*现在，看看这个:*

```
*valuer(value).as(natural);*
```

# ***估价师就是验证***

***让我给你看看这个真正快速的***

*这个`natural`标识符是自然数的实际描述。
估价师拥抱[契约式编程](https://en.wikipedia.org/wiki/Design_by_contract)。要验证自然数，您必须首先为 Valuer 创建对它们的描述，以了解它们看起来像什么。你可以这样做:*

```
*import { Descriptor } from "@valuer/main/dist/types";const natural: Descriptor<number> = {
    number: "integer",
    spectrum: "non-negative",
};*
```

*该描述符表示，必须满足以下所有要求的*才能将一个值视为有效的自然数:**

*   *它必须是**一个原始数字**(由`"number"`验证器隐含)；*
*   *它必须是一个整数。*
*   *必须是**大于等于**`**0**`；*

> *使用原始包装器，比如`*Number*`、`*String*`、`*Boolean*`或`*Symbol*`构造函数，被认为是一种不好的做法。Valuer 遵从这一点，并要求这些值是原始的。*

*现在让我们使用给定的描述符来验证一些东西:*

```
*import { valuer } from "@valuer/main";valuer(36.6, "body temperature").as(natural);*
```

*那个`36.6`看起来不像是自然数，Valuer 知道这一点，因为您已经在描述符中提供了该信息！*

> *或者，可以使用更简单的语法:`*.as("non-negative", "integer")*`。估值师将尝试解释输入内容，并为您挑选合适的验证者。*

*根据失败情况，将会引发一条错误消息，如下所示:*

```
*Validation failed: body temperature is not an integer: value <number> 36.6*
```

*瞧啊！你不仅知道出了问题，你还知道**到底是什么**，这在更高的层面上意味着什么，以及，大概，如何修复它。*

> *请记住，验证的顺序通常取决于描述符中键的顺序(也称为验证器)，但 JavaScript 本身并不保证这一点。*

*像这样的错误信息总是以`"Validation failed"`开始。它还将提到值的作用(在本例中是`"body temperature"`)、失败(`"is not an integer"`，你猜对了)以及值的字符串表示，以类型和单词“value”为前缀*

> *错误详细信息是可配置的。可以提供更少或更多的细节。*

# *我可以提供自己的错误信息吗？*

***好笑你应该问***

*根据“错误信息”的确切含义，您至少有两种方式提供它们:通过*自定义验证器*或使用不同的*模式*。*

## ***自定义验证器***

*尽管 Valuer 中有各种各样的内置验证器(现在我将省略)，您可能对这些东西有自己的看法。估值师对此表示赞赏。您可以根据自己的条件和失败创建和使用自己的验证器:*

```
*const list: Descriptor = {
    **custom: Array.isArray**,
};*
```

*这种用法将使内置故障出现在错误上:*

```
*valuer("hi").as(list);
// Validation failed: value **does not meet the constraint***
```

> *当然，`*"custom"*`验证器只能使用一次，因为 JavaScript 中的对象不能有多个属性具有相同的键。*

*为了提供更有意义的错误消息(这显然是您想要的)，只需用实际的失败替换这个键:*

```
*const MAX_REAL_AGE = 120;const age: Descriptor<number> = {
    '**is not a valid age**': (n: number) => n > 0,
    '**is too big to be real**': (n: number) => n <= MAX_REAL_AGE,
    '**shouldn't be used**': (n: number) => n !== 13,
};valuer(-5).as(age);
// Validation failed: **value is not a valid age**valuer(256, "oldest man's age").as(age);
// Validation failed: **oldest man's age is too big to be real**valuer(13, "baker's dozen").as(age);
// Validation failed: **baker's dozen shouldn't be used***
```

*在使用定制验证器时，你应该记住失败是关键，返回布尔的**函数** ⁴(又名 *thunk)* 是它们中任何一个的值。Valid 意味着“真”，所以如果提供了一个有效值，请确保该函数返回`true`,并且该失败具有足够的描述性，有助于快速修复错误。*

> *请注意，失败文本应该以助动词开头，如“是”或“不”等。通过这种方式，错误消息保持可读和一致。*
> 
> *⁴:我建议使用初等函数，一次只检查一件事。*

## ***不同模式***

*显示定制错误的另一种方式是每当出现错误时执行完全不同的定制逻辑。这种方法意味着 Valuer 仅用于回答简单的问题:“输入有效吗？”*

*特别是对于这一点，有一个`"check"`模式。它改变了验证过程，并强制 Valuer 只返回一个布尔值，而不是抛出一个错误:*

```
*const answer: Descriptor<string> = {
    lengthRange: [ 100, Infinity ],
};const valid: boolean = valuer("42").as(answer, **{ mode: "check" }**);if (!valid)
    alert("Only when you know the question, will you know what the answer means");*
```

*在本例中，描述符旁边有一个配置 object⁵，其属性`.mode`等于`"check"`——这就是切换 mode⁶.的原因*

> *⁵配置对象只能与描述符一起使用。第二个脚注中的替代语法(参见)只接受描述符值(也称为验证)。*

*如果你的目标是在你的 web 项目中获得流畅的用户体验，或者不喜欢过多地阅读控制台，这可能也是有用的。*

> *⁶在这个例子中，模式仅针对这个执行而改变。要一劳永逸地改变模式，只需在工作代码行之前的某个地方使用`*valuer.config*`方法。
> **不过要负责任地使用它**，因为过度使用这个特性会使调试代码变得很痛苦。我建议每个项目使用一次。或者更少。*

# ***所以，就这样了？***

***当然不是！那会很无聊的***

*这篇文章涵盖了非常基础的估值。现在很清楚 Valuer 是什么，它解决什么问题。但是我省略了“它能做什么”这一部分*

*这里没有描述的是:*

*   *验证器的详尽列表— `"set"`、`"kind"`、`"pattern"`等等；*
*   *其他模式，如`"assert"`、`"describe"`和`"switch"`；*
*   *修改器，包括`"details"`和`"rangeInclusive"`；*
*   *可重用性—`valuer.as(descriptor, config)(value, role)`；*
*   *描述符缩写和其他描述值；*
*   *创建真实世界的定义—`valuer.define(name, description)`；*
*   *验证非原始值；*

```
*import { valuer } from "@valuer/main";
import { Descriptor } from "@valuer/main/dist/types";interface Person {
    name: string;
    age: number;
}const nameDescriptor: Descriptor<string> = {
    **pattern**: /(?:[A-Z][a-z]* ?){2}/,
    **lengthRange**: [ 1, 32 ],
};**valuer.define**<number>("**age**", {
    **typeOf**: "number",
    **range**: [ 1, 120 ],
});valuer.define<Person>(":**person**", {
    **composite**: {
        name: nameDescriptor,
        age: **[ ":age" ]**,
    },
});const validate = **valuer.as**<Person>("**:person**");validate({ name: "hello", age: 17 }, "a person");
// Validation failed: 'name' in a person does not match the patternvalidate({ name: "Jon Snow", age: 0 }, "Jon Snow");
// Validation failed: 'age' in Jon Snow is out of boundsvalidate({ name: "Jon Snow", age: 17 });
// (ok)*
```

*哦，我有没有提到 Valuer 完全是用打字稿写的，它有丰富的智能感知支持？*

*![](img/3b8a77659eb027e827894753dd8e0f68.png)*

*顺便说一下，还有其他有用的相关包，比如`[@valuer/help](https://www.npmjs.com/package/@valuer/help)`。包括 Valuer istelf 在内的所有人都获得了麻省理工学院的许可，并涵盖了单元测试。*

*Valuer 是不断发展的。如果你喜欢它背后的概念，或者特别是如果你觉得它可以变得更好，请[加入团队](https://gitlab.com/valuer)和[查看问题列表](https://gitlab.com/valuer/main/issues)以做出贡献。*

*在那里见👋*