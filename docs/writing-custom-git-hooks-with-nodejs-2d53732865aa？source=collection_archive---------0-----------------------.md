# 用 NodeJS 编写定制 Git 挂钩

> 原文：<https://itnext.io/writing-custom-git-hooks-with-nodejs-2d53732865aa?source=collection_archive---------0----------------------->

最初发布在[开发到](https://dev.to/shaharkazaz/writing-custom-git-hooks-with-nodejs-3f8b)中。

![](img/c3b796a385baa67a14eaf02889dfdd5c.png)

Git 挂钩是一个有用的工具，尤其是在大型团队中工作时。
他们可以帮助我们将代码风格和林挺标准应用于我们的暂存文件。

在本文中，我们将编写几个强大的 Javascript git 挂钩，它们将帮助我们管理我们的代码库，并拥有更流畅的开发体验。

# 运行脚本

我们将在哈士奇的帮助下运行我们的钩子🐶。
在我们[安装了 Husky](https://github.com/typicode/husky#install) 之后，我们需要做的下一件事就是运行我们的节点脚本。
让我们将脚本添加到`package.json`脚本部分，并使用 husky 调用它:

```
scripts: {  
  ...
  "hooks:pre-commit": "node ./hooks/pre-commit.js",
  "hooks:pre-push": "node ./hooks/pre-push.js"
},
...
husky: {
  "pre-commit": "npm run hooks:pre-commit",
  "pre-push": "npm run hooks:pre-push"
},
```

差不多就是这样，现在让我们看看
`pre-commit`和`pre-push`钩子的一些有用的实现。

## 执行. js

我为我的钩子脚本创建了一个`exec.js`助手函数，它包装了`shelljs`的`exec`函数。
`exec`函数生成一个外壳，然后在该外壳中执行给定的命令:

# 预提交📦

## 1.分行名称约定

仅允许创建具有以下前缀之一的分支:`feature|fix|hotfix|chore|tests|automation`

## 2.✋禁售代币

谁没忘记去掉一个`debugger`？还是一个测试中的`fdescribe`？不再有了！

# 预推🚀

## 1.自动同步主机

我们注意到开发人员经常忘记从远程定期更新他们的分支。

这是一个简单但重要的钩子，它从远程`master`更新您的本地分支。

## 2.✋禁枝

有些分支我们不希望它们的提交在`master`
结束，比如`staging`分支。

我们将在这些分支中提交一个充当“标志”的 commit🚩。
在推送到远程之前，我们将验证该提交不是被推送到的分支的一部分(我们显然将删除`staging`分支中的这段代码)。

# 外卖食品

我们看到了一些使用 git 挂钩的有用示例，以及使用 Husky 和 NodeJS 应用策略和防止错误提交是多么容易。

现在你可以用最适合你的项目🥳的方式定制这些钩子

# 你试过 Transloco 吗？🌐

`ng-neat`介绍 **Transloco** ，Angular 的国际化(i18n)库。它允许你为你的内容定义不同语言的翻译，并在运行时轻松地在它们之间切换。

它公开了一个丰富的 API 来高效、干净地管理翻译。它提供了多个插件，将改善您的开发体验。
我们/我强烈建议您阅读更多相关内容并[查看](https://github.com/ngneat/transloco)！

[](https://netbasal.com/introducing-transloco-angular-internationalization-done-right-54710337630c) [## 🚀Transloco 简介:角度国际化进展顺利

### 有一段时间，我一直在考虑创建一个 Angular i18n 库，它包含了我脑海中的一些概念。

netbasal.com](https://netbasal.com/introducing-transloco-angular-internationalization-done-right-54710337630c) [](https://medium.com/@shahar.kazaz/translation-files-validation-in-angular-with-transloco-e6ba02467f33) [## 使用 Transloco 在 Angular 中验证翻译文件

### 当与多个团队一起开发一个企业应用程序时，我们经常会遇到合并冲突…

medium.com](https://medium.com/@shahar.kazaz/translation-files-validation-in-angular-with-transloco-e6ba02467f33) [](/lazy-load-translation-files-in-angular-using-transloco-2d3afed116ce) [## 使用 Transloco 在 Angular 中延迟加载翻译文件

### 在本文中，我们将学习如何使用 Angular Transloco 的国际化库来延迟加载…

itnext.io](/lazy-load-translation-files-in-angular-using-transloco-2d3afed116ce)