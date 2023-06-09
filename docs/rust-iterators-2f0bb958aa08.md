# Rust 迭代器

> 原文：<https://itnext.io/rust-iterators-2f0bb958aa08?source=collection_archive---------3----------------------->

你好

在这篇文章中，我们将讨论 rust 中的迭代器。

如果你喜欢视频，这篇文章也有视频解释。

迭代器，正如你可能在其他语言中看到的，是一种告诉我们的语言我们想要对一些数据进行某种循环的方式。

在 rust 中，像几乎所有其他语言特性一样，对于迭代器，你需要知道两个特性,“迭代器”和“IntoIterator”。

让我们看一些代码

在这个示例代码中，我们可以看到如何在 rust 中迭代数组，这里没有什么特别的，几乎每种编程语言的开发人员都很熟悉

让我们看看第二个样本。

在这个例子中，我们迭代一个向量，它基本上是一个动态大小的数组，看起来很像数组一。

最后，在我们深入研究这个特性如何在 rust 中实现的细节之前，让我们看一下另一个例子。

在这个例子中，我们迭代了一个 hashmap，我想你可以看到这里的模式，hashmap 和 arrays 是完全不同的，但两者都可以迭代，我们可以使用相同的语法在 rust 中很容易地表达这一点，但所有这些能力来自哪里呢？

像 Rust 的大多数特性一样，答案是相同的，这种能力和表现力来自于特征，更具体地说是“迭代器”特征。

让我们来看看‘迭代器’特征的定义。

这个特征有一个与之相关的类型，基本上就是我们正在迭代的类型，我们有许多方法，但是如果你仔细看看，它们都有一个默认的实现，除了一个“next”方法。每次调用“next”方法时，都会返回一个新值，该值将在循环的迭代中使用。

现在我们知道 rust 中的数组、向量、hashmaps 和许多其他类型都实现了“迭代器”特性，因此它们可以在 for-in 循环中使用，但是我们可以扩展这种功能供我们自己使用吗？

答案是肯定的，这正是我们接下来要做的，我们要为自定义结构实现“迭代器”。

这个迭代器相当简单，只是一个简单的向量包装器，让我们看看这个类型的迭代器 impl，你可以看到我们刚刚指定了类型关联和下一个方法，下一个方法相当简单，它只是从向量中获取下一个值，如果没有，它就返回它。

正如你在下面的测试中看到的，这种类型的用法与标准库向量和数组或 hashmap 没有什么不同，它们的用法完全相同，因为它们都实现了相同的东西。

现在，有时就像我们在 CustomIterator 中的例子一样，我们只是在我们的类型中包装另一个迭代器，所以也许有一种方法可以告诉 rust 我们的类型迭代逻辑与底层类型相同，所以只需将该迭代器用于循环，这可以通过使用“IntoIterator”特征来实现，

在这个文件中，我们有完全相同的代码，除了不是实现‘iterator’特征，而是实现‘into iterator’特征。这个特性告诉编译器如何将一个类型转换成迭代器，这样它就可以用于 for-in 循环。如你所见，它有一个消耗自身(获取所有权)并返回迭代器的“into_iter”方法。(在我们的例子中，它是矢量的标准 iter 类型)。

`Iterator`和`IntoIterator`特征之间的区别在于`Iterator`特征完全实现了迭代逻辑，而`IntoIterator`只是创建了一个新的迭代器并将其返回给 for 循环，所以在我们的例子中，我们的结构中已经有了一个迭代器，只实现`IntoIterator`更符合逻辑。

最后的话:

特征是 rust 语言的核心部分，是大多数语言特性的组成部分，理解它们将有助于我们更好地理解这种语言。