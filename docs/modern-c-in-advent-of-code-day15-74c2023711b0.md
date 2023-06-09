# 现代 C++代码的出现:第 15 天

> 原文：<https://itnext.io/modern-c-in-advent-of-code-day15-74c2023711b0?source=collection_archive---------3----------------------->

这是[代码](https://adventofcode.com/2021)问世的第十五天。今天，我们将寻找最便宜的路径。

![](img/8da18af2aaa3bc109324c30ac81dfc0f.png)

一如既往，请先尝试解决问题，然后再看解决方案。对于这个系列的所有文章，[看看这个列表](https://medium.com/@happy.cerberus/list/advent-of-code-2021-using-modern-c-c5814cb6666e)。

# 第 15 天:第一部分和第二部分

我们的输入是个位数成本的平方，我们的任务是找到从左上角到右下角的最便宜路径的成本。我们将需要两个函数，一个用于读取输入，一个用于查找路径的开销:

对于测试，我们使用 AoC 中提供的数据:

实现最小成本路径搜索的最简单方法是从无限距离开始，向所有四个方向搜索，并在距离增加的情况下扩展:

这种方法可行，但可能会造成浪费，因为我们可以重复搜索相同的区域，每次都会略微提高成本。

然而，对于第 1 部分中的少量输入来说，这当然足够了:

对于第 2 部分，我们的任务是将输入扩展到一个 5x5 的网格中，并在这个更大的输入中找到路径。我们可以贪婪地这样做:

因为公式有点复杂，所以让我们一步一步来。我们首先分配 5x5 网格(第 3 行)。然后，我们迭代这个新网格的所有元素，并将每个元素设置为所需的值(第 6 行)。我们需要确保最终值在 1 的范围内..9 使用:`(value-1)%9+1`。而`i/dim + j/dim`项是基于 5x5 网格中 X 和 Y 坐标的调整。

我们也可以惰性地实现它。如果有兴趣，请告诉我，我会将懒惰解决方案添加到代码库中。

最后，让我们改进我们的最小路径搜索。通过最小的修改，我们可以将我们的搜索转换为符合 Dijkstra 最小路径算法:

通过利用优先级队列，我们首先优先考虑最便宜的路径。这限制了对每个节点的重复访问的总数。如果我们计算一下这两种方法的时间，就很容易看出这一点:

优先级队列版本的运行速度提高了约 2 倍:

# 链接和技术说明

每日解决方案存储库位于:[https://github.com/HappyCerberus/moderncpp-aoc-2021](https://github.com/HappyCerberus/moderncpp-aoc-2021)。

[查看此列表，了解《代号](https://medium.com/@happy.cerberus/list/advent-of-code-2021-using-modern-c-c5814cb6666e)问世前几天的相关文章。

请不要忘记亲自尝试[降临码](https://adventofcode.com/2021)。

# 感谢您的阅读

感谢您阅读这篇文章。你喜欢吗？

我也在 YouTube 上发布视频。你有问题吗？在[推特](https://twitter.com/SimonToth83)或 [LinkedIn](https://www.linkedin.com/in/simontoth) 上联系我。