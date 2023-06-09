# 什么是代码高尔夫，为什么它很烂？

> 原文：<https://itnext.io/what-is-code-golf-and-why-does-it-suck-d221491afac4?source=collection_archive---------2----------------------->

![](img/f779f62840373e5c711f1738aaa65c38.png)

你可能听说过“Code Golf”这个术语，以前在编程比赛或类似于 [Code Signal](https://app.codesignal.com/signup/it5dxvERqmSimjsLg/main) (正式名称为 Code Fights)的编程挑战网站中使用过。但是这个术语是什么意思呢？又有什么不好呢？

# 那是什么呢？

代码高尔夫是指尝试使用尽可能少的字符来解决问题，即使用最少的源代码。这个术语来源于高尔夫运动，球员的目标是用最少的击球次数将球击入洞中。在 Code Golf 中，使用尽可能少的字符解决问题(通常是编写一个函数)的开发人员获胜。通常空格和换行符不算字符，只是为了让法官实际上能正确阅读解答。至于其他一切，目标是使用尽可能少的字符来解决所提供的问题。

Code Golf 不仅限于一种特定的编程语言。你可以使用任何你想要的编程语言玩代码高尔夫，只要它能完成工作。但是，在 Code Golf 中，您不能将一种语言的解决方案与另一种语言的相同解决方案进行比较。这是不公平的，代码高尔夫比赛应该总是只比较用同一种语言编写的解决方案。

# 一个极其简单的例子:

假设给你一个任务，写一个返回两个数之和的函数。因为我是一个 JavaScript 爱好者，所以我将提供一个合乎逻辑的解决方案，我会在下面的工作面试中提供:

```
function sumOfTwoNumbers(number1, number2) {
  return number1 + number2;
}
```

正如你所看到的，我已经写了一个函数，它简洁明了地解决了这个问题。任何开发人员都可以很快理解它，而不必质疑他们生活中导致他们成为开发人员的决定。此解决方案包含 64 个字符—请记住空格和换行符不算在内！

现在，我们确实生活在一个现代的时代，现代 JavaScript 是一个东西。这给了我们一些难以置信的 [ES2015(又名 ES6)](https://babeljs.io/docs/en/learn/) 功能，我们可以用它们来简化这个功能，这将帮助我们赢得终极代码高尔夫挑战赛。所以让我们在下面试一试:

```
const sum = (num1, num2) => num1 + num2;
```

这个解决方案只有 32 个字符，实际上是前一个解决方案所用字符量的一半。太好了！因此，我们已经简化了我们的功能，将所有功能放在一行中，我们的 JS 开发人员同事仍然理解正在发生的事情，而不会质疑他们的理智——前提是他们对 ES2015 很熟悉。

但是我们在玩代码高尔夫，而你那个肮脏的超水平开发者朋友已经移除了更多的字符——因此提出了下面这个更短的解决方案:

```
const x = (a, b) => a + b
```

这个混蛋甚至省略了结束分号，因为它仍然可以完成工作，而分号是一个字符的浪费。我们走吧，你的肮脏的朋友赢得了 17 个字符的代码高尔夫比赛！恭喜你，怪物…

# 代码高尔夫有什么好处？

在我继续咆哮为什么代码高尔夫是一个可怕的概念庆祝。让我指出它确实有它的额外津贴。

任何人都会告诉你的第一个论点是，较小的源代码有利于性能。更少的代码意味着更小的文件大小，这意味着下载文件将更快。但是现在我们有了代码精简工具和 transpilers，这使得这个论点变得多余。

Code Golf 的一个更好的论点是，它通常鼓励开发人员跳出框框思考，利用不常用的语言特性，从而使他们向开发人员工具箱中添加工具。迫使开发人员学习和扩展他们的技能绝对是一个积极的结果。

所以我们走吧，有一些好的结果。但这就足够好了吗？

# 为什么很烂？

这是几年前我工作生涯中真实发生的一个场景。

**解决方案架构师对我说:**
“嘿 *Barry，在与‘其他开发人员’一起实现复杂功能方面做得很好。不过，只有一件事。我在检查你们写的代码，发现了你们的* `*complicatedFunction*` *函数。我真的不明白它的参数是怎么回事，因为它们被标记为*`*p*`*`*v*`*和* `*w*` *(函数是这样写的:* `*complicatedFunction(p, v, w) { ... }*` *)。能否请你为* `*complicatedFunction*` *提供可理解的变量名，以便我们以后遇到这种情况时不会混淆？”**

***我来解师:** *“哦耶你说得对！这是一个奇怪的。让我问问“其他开发者”他昨天写这篇文章后的想法是什么。一旦我确定了它们的用途，我会用更易读的变量名来更新它。”**

***Me to“Other Developer”:** *“嘿伙计，大老板和我都不明白* `*complicatedFunction*` *是怎么工作的，因为你传递给它的参数只是随机字符。你能告诉我*`*p*`*`*v*`*`*w*`*这些变量代表什么吗？”****

****“其他开发者”对我:** *“嘿伙计，大老板和我都不明白* `*complicatedFunction*` *是怎么运作的，因为你传递给它的参数只是随机字符。请问* `*p*` *、* `*v*` *、* `*w*` *这些变量是什么意思？”***

**这绝对是一个令人沮丧的情况，因为我让我的解决方案架构师期望这是一个快速简单的 5 分钟修复。但是不，“其他开发人员”真的不记得这些变量代表了什么。当你有一个很大的复杂的函数，在整个节目中使用随机命名的变量，同时努力保持跟踪一切。你会看到一群沮丧的开发人员，他们不明白为什么他们的代码能够工作。**

**![](img/c57c68a3cd28e7d73109a19fb9dfdf0f.png)**

# **在工作场所打高尔夫**

**上面的场景是为什么 Code Golf 的某些方面不酷的明显例子！“其他开发人员”认为他是一个传奇人物，他把复杂的功能做得小巧紧凑，却不关心它的可读性。**

**不管你是否和其他人一起工作。用一种能被快速理解的方式来写代码是有意义的，而不是以后每次回来都试图弄明白它。人们花在阅读代码上的时间是实际写代码的 10 倍。因此，从生产力的角度来看，编写干净、可维护和可理解的代码在工作场所和个人项目中是一个巨大的优势。**

**具有讽刺意味的是，编写神秘的代码会让你吃不了兜着走。破译高尔夫格式代码的时间加起来。我们都知道，在开发领域，时间是一件大事。如果你可以节省时间来完成商业目标，你可以证明提高你的时薪或者追求你已经关注了一段时间的晋升是合理的。**

# **面试中的代码高尔夫**

**在工作面试中，面试官希望看到你提出实际上可以理解的解决方案的能力。如果你得到了这份工作，你的面试官可能会和你一起工作。他们想确定你的存在会让他们的生活更轻松，而不是更复杂！**

**白板面试测试的目标是测试您是否能提出高性能的解决方案。当他们要求您实现最有效的解决方案时，从您的源代码中删除字符并不是他们的意思。**

**如果他们告诉你，你的变量名应该简化为单个字符，那么这可能是一个测试，看看你是否关心编写干净的代码。如果这不是一个测试，那么你可能也不想在那里工作。如果是那样的话，你肯定不会喜欢那里的生活。**

# **在实践中**

**还记得我在这篇文章顶部的简单例子吗？我们写了一个函数，接受两个参数并返回它们的和。在现实世界中。这些函数需要在某个地方调用，对吗？让我们看看上面提到的每个例子是什么样子的。我们将按照从肮脏的代码高尔夫赢家到体面的代码黄金输家的顺序来展示它。**

## **肮脏代码高尔夫冠军**

```
**x(1, 2); // returns 3**
```

**这是什么？我完全不知道。我需要运行这个函数来看看它能做什么。即使这样，在没有检查实际函数的情况下，我也不知道要为函数提供多少 arugments。**

## **代码高尔夫第二名**

```
**sum(1, 2); // returns 3**
```

**这很好。我可以清楚地看到这个函数，它有两个参数。给定函数名`sum`，我自然可以假设它将返回两个参数相加的结果。然而，如果这个函数是在没有上下文的情况下给我的。我可能还是会感到困惑，因为我不确定这个函数需要多少个参数。**

## **体面的代码高尔夫失败者**

```
**sumOfTwoNumbers(1, 2); // returns 3**
```

**这太棒了！即使我不知道函数中要输入多少个参数，名字`sumOfTwoNumbers`肯定有助于澄清这一点。很明显这是一个以两个数为输入，将它们相加的函数。**

**现在，走出去，确保您的代码是美丽的，干净的和可理解的，以便您可以维护它！因为，你知道他们说什么。“知道发生了什么的开发人员是快乐的开发人员！这可能也只是一个梦，你很快就会醒来……”**

**这篇文章的灵感来自《T2 代码全集:软件构造实用手册》第二版。它绝对值得一读，因为它将鼓励并教会你用更多的结构和思想来编写干净的代码。量两次，切一次！**

**你对 Code Golf 有什么想法？请在下面的评论中告诉我。**

**记住，也不全是坏事。只是不要在工作场所、面试或个人项目中使用它！**

***原载于 2018 年 8 月 27 日*[*【www.barrymichaeldoyle.com】*](https://www.barrymichaeldoyle.com/code-golf)*。***