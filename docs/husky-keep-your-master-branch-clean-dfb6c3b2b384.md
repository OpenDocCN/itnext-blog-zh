# 哈士奇——保持你的“主人”树枝干净

> 原文：<https://itnext.io/husky-keep-your-master-branch-clean-dfb6c3b2b384?source=collection_archive---------4----------------------->

我敢肯定，正在阅读这篇文章的许多人都遇到过这种情况:你有一个阳光明媚的早晨，鸟儿在歌唱，你来到办公室，从主应用程序下载最新的应用程序，结果应用程序坏了，就这样——你的早晨被毁了😡。

![](img/b9b88d43b42c767248634c2248bfcd00.png)

有一个很棒的工具可以消除这个问题——[哈士奇](https://github.com/typicode/husky)。Husky 让添加 git 挂钩变得尽可能简单。

> 钩子是可以放在钩子目录中的程序，可以在 git 执行的某些时候触发动作。—来自 [git 文档](https://git-scm.com/docs/githooks)。

使用 husky，您只需在 package.json 文件中提供一个脚本，例如，该脚本将在提交或推送时触发。

1.  安装 husky

```
npm install -D husky
```

2.添加[任意挂钩](https://git-scm.com/docs/githooks#_hooks)到你的 package.json

```
"precommit": "your code goes here (testing, linting)..."
```

在我看来 ***prepush*** 命令是最有用的，因为它不像 precommit 那样经常执行，而且它仍然可以确保不将损坏的代码推送到主程序。

以下是我们团队在当前 Angular 项目中使用的配置:

```
"prepush": "run-p lint test-headless \"ng -- build --prod\" "
```

上面的代码运行林挺(***" lint ":" ng lint "***)，运行单元测试(***" test-headless ":" ng test-Sr-browsers chrome headless "***)，并检查生产构建没有失败(**\ " ng-build-prod \ "**)。我还使用[NPM-run-all](https://github.com/mysticatea/npm-run-all)*来并行执行所有任务。*

*希望这篇小博文能帮助某人成为更快乐的开发者。*