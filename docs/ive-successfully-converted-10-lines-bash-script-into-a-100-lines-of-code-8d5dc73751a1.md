# 我已经成功地将 10 行 Bash 脚本转换成了 100 行代码

> 原文：<https://itnext.io/ive-successfully-converted-10-lines-bash-script-into-a-100-lines-of-code-8d5dc73751a1?source=collection_archive---------0----------------------->

![](img/1d1ed2a862091a3a9b30d55b79247a5a.png)

照片由 [Ales Krivec](https://unsplash.com/@aleskrivec?utm_source=medium&utm_medium=referral) 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

我有 10 行 bash 脚本，用于在命令行上显示虚拟机的信息，作为虚拟机命令行程序的一部分。

此命令用于在命令行中列出虚拟机信息，如下所示:

```
$ vermin ls
VM NAME  IMAGE           CPU  MEM     TAGS
vm_01    ubuntu/bionic   2    4096    spark kafka hadoop
vm_02    ubuntu/bionic   1    1024
vm_03    ubuntu/bionic   1    1024
```

我遇到的问题是，该脚本为每个显示信息的虚拟机花费了大约 **150 ms** 。
在上面的示例中，它花费了大约 **500 毫秒，**如果我有更多虚拟机，它将花费更多时间。

在我们继续我为改进脚本运行时间所做的工作之前，让我们展示一下 bash 脚本:

bash 脚本片段

上面的脚本——除了读取一些文件——调用命令`vboxmanage showinfo myvm --machinereadable`,这显然是脚本中最耗时的指令。

因此，我决定为我拥有的每个虚拟机并行运行这个命令，这样就不用花费(**虚拟机数量* 150 毫秒)**，总时间大约需要 150 毫秒。

我决定使用 **Golang** ，这样我就可以利用它惊人的 goroutines 来并行调用我所拥有的虚拟机的`vmboxmanage`命令。

下面是[代码中与](https://github.com/mhewedy/vermin/blob/6bf3ba223447a0b262faf51bb11cdbb0c1dddbf1/vms/list.go#L83-L103)相关的部分:

```
**type** vmInfoList []*vmInfo
....
....numVms := len(vms)

infoList := make(vmInfoList, numVms)
**var** wg sync.WaitGroup
wg.Add(numVms)

**for** i, vmName := **range** vms {
   **go func**(vm string, i int) {
      infoList[i] = getVMInfo(vm)
      wg.Done()
   }(vmName, i)
}wg.Wait()
```

这里的要点是，我在虚拟机上循环，对于每一个虚拟机，我在一个调用命令`vboxmanage`并执行其他 I/O 操作并将`vmInfo`结构返回给调用者的 goroutine(把它想象成一个轻量级线程)中调用`getVMInfo`。

然后，我使用来自每个 goroutine 的**通道**将结果收集到一个 **select** 语句中，该语句带有一个等待结果的循环，一旦我得到了所有等待的结果，我就中断。

这如何提高我的程序的性能？

```
$ time vermin ls
VM NAME  IMAGE           CPU  MEM      TAGS
vm_01    ubuntu/bionic   2    4096     spark kafka hadoop
vm_02    ubuntu/bionic   1    1024
vm_03    ubuntu/bionic   1    1024vm ls 0.14s user 0.12s system 118% cpu 0.219 total
```

如图所示，性能从 **n * 150 毫秒**(在本例中大约为 450 到 500 毫秒)上升到大约 220 毫秒。

我在 5 个虚拟机上重复了上面的命令，我得到了一个相似的数字。

你可以在 [GitHub 上找到完整的 100 多行代码。](https://github.com/mhewedy/vm/blob/7d08c983387d6f92784c280490925a7cbc01dd14/vminfo/main.go)

你可能认为 500 毫秒甚至 1 秒钟并不算长，但是让我告诉你，`*docker ps*`显示 10 个容器**的信息需要不到 **100 毫秒**。**

在我完成上面的程序后，我意识到 VirtualBox(我所依赖的)使用一个`XML`格式的文件来存储关于每个 VM 的信息，所以我可能会尝试读取和解析该文件，甚至使程序更具响应性。

顺便说一下，我写 bash 脚本用了 10 ~ 15 分钟，而 go 程序花了 1 ~ 2 个小时:)。

下面是包含整个程序的 GitHub repo:
*(如果你认为有用，请在项目的 GitHub 页面上打个星号*🚀)

[](https://github.com/mhewedy/vermin) [## mhewedy/害虫

### 目录:害虫是一个智能，简单和强大的命令行工具，用于 Linux，Windows 和 macOS。它被设计成…

github.com](https://github.com/mhewedy/vermin) 

更新:

在我直接从`.vbox`文件中读取虚拟机信息后，程序现在正在运行，5 个虚拟机只需要不到 **80 毫秒**。

```
$ time vermin ls
VM NAME  IMAGE           CPU  MEM      TAGS
vm_01    ubuntu/bionic   2    4096      spark kafka hadoop
vm_02    ubuntu/bionic   1    1024
vm_03    ubuntu/bionic   1    1024
vm_04    ubuntu/bionic   1    1024
vm_05    ubuntu/bionic   1    1024vm ls  0.03s user 0.03s system 76% cpu 0.077 total
```

更新 2:

我已经根据 Reddit 中[评论的建议，将代码从使用通道和选择器改为简单使用`WaitGroup`。](https://www.reddit.com/r/golang/comments/gpdmb4/ive_successfully_converted_10_lines_bash_script/)