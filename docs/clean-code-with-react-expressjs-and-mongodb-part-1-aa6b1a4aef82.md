# 使用 React、ExpressJS 和 MongoDB 清理代码—第 1 部分

> 原文：<https://itnext.io/clean-code-with-react-expressjs-and-mongodb-part-1-aa6b1a4aef82?source=collection_archive---------4----------------------->

![](img/48239f8a1815ef8887b5e5bca2ab574f.png)

> [*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fclean-code-with-react-expressjs-and-mongodb-part-1-aa6b1a4aef82%3Futm_source%3Dmedium_sharelink%26utm_medium%3Dsocial%26utm_campaign%3Dbuffer)

这是一个简短系列文章的第一部分，重点是使用 ES6+ Javascript、React、Redux、ExpressJS 和 MongoDB 创建和部署一个准系统的全栈博客站点。

在这些文章中，我真的试图把重点放在我发现的使用 ES6+ JavaScript 和我称之为“Redux 风格”的项目结构来设置和组织我的代码的一些方法上。目标是编写可读、可扩展且易于测试的代码。

这个系列将分为几个部分:

1.  设置 ExpressJS 以使用 ES6+语法👈🏽*你在这里*
2.  [Redux 风格的文件结构](/clean-code-with-react-expressjs-and-mongodb-part-2-89d20e684820)
3.  [创建简单快速服务器](/clean-code-with-es6-expressjs-reactjs-part-3-2306b1f62c26)
4.  用我们的博客设置 Mongoose(即将推出！)
5.  用 CMS 创建一个 React 博客(即将推出！)
6.  将 Redux 添加到您的前端(即将推出！)
7.  包装您的应用:部署(即将推出！)

====

后端的 ES6+？当然可以！最新的 Javascript 语法确实在使代码变得清晰易读方面大有作为。毕竟，[代码是给人看的，不是机器看的。](https://mitpress.mit.edu/sicp/front/node3.html)

(不是 ES6+爱好者？你可能想温习一下语法。这里不做过多解释。)

另外，请记住，我们*不会*使用测试来开发我们的应用程序。在本教程系列中，这纯粹是为了简单起见。编写好的测试*“àla TDD”*是**编写可靠代码的第一步**。

# 后端目录布局

我们将从创建一个目录开始，该目录具有以下文件结构和一个空的`index.js`文件:

```
📁/
 📂configs/
 📂database/
 📂routes/
 📄index.js
```

# 安装先决条件

# 故事

我们将使用**线程**而不是 **npm** 来安装和运行我们的包。让我们初始化我们的项目。在根目录中，键入:

`yarn init -y`

这将创建一个非常简单的`package.json`文件，内容如下:

*。/package.json:*

```
{ 
  "name": "backend", 
  "version": "1.0.0", 
  "main": "index.js", 
  "license": "MIT" 
}
```

# ExpressJS

我们将使用 [ExpressJS](https://expressjs.com/) 来创建一个服务器。这是一个奇妙的极简 NodeJS 框架，既强大又灵活。我们还将安装 [Nodemon](https://www.npmjs.com/package/nodemon) ，这是一个很棒的开发工具，可以监视我们的目录变化并重新加载节点服务器。

`yarn add express nodemon -S`

# 巴比伦式的城市

因为我们在 ExpressJS 上使用 ES6+语法而不是 NodeJS，所以我们需要确保将其向下转换到 ES5。这涉及几个步骤。

## 1.安装巴别塔软件包

我们将使用巴别塔把我们的包传送到 ES5。在这里，我们安装了 Babel CLI 和“ENV”，后者[“根据您支持的环境自动确定您需要的 Babel 插件。”](https://babeljs.io/docs/plugins/preset-env/)。

`yarn add babel-cli babel-preset-env -D`

## 2.创建一个`.babelrc`文件

现在我们需要创建一个文件，告诉 Babel 获取最新的 NodeJS 预置。在根目录下创建一个`.babelrc`文件，并添加以下代码:

*。/.babelrc:*

```
{ 
  "presets": 
    [ 
      [ "env", 
        { "targets": 
          { "node": "current" } 
        } 
      ] 
    ] 
}
```

## 3.在我们的`package.json`中创建一个脚本

让我们打开我们的`package.json`文件，并使用 babel-node 预置修改它来运行 Nodemon。下面是我们的`package.json`应该是什么样子(当然不算版本)。

*。/package.json:*

```
{ 
  "name": "backend", 
  "version": "1.0.0", 
  "main": "index.js", 
  "license": "MIT", 
  "scripts": { 
    "start": "nodemon --exec yarn babel-node -- index.js" 
    }, 
  "dependencies": { 
    "express": "^4.16.3", 
    "nodemon": "^1.17.2" 
  }, 
  "devDependencies": { 
    "babel-cli": "^6.26.0", 
    "babel-preset-env": "^1.6.1" 
  } 
}
```

🔬该脚本使用 Babel-node 预置在`index.js`文件启动 Nodemon

## 4.测试一下！

让我们给我们的`index.js`添加一些代码，看看它是否工作。将下面的测试代码添加到根目录下的`index.js`中:

*。/index.js:*

```
import express from 'express' console.log('It Works!')
```

如果您在终端中输入命令`yarn start`,您应该会得到类似这样的结果:

*控制台:*

```
yarn run v1.3.2
$ nodemon --exec yarn babel-node -- index.js
[nodemon] 1.17.2
[nodemon] to restart at any time, enter `rs`
[nodemon] watching: *.*
[nodemon] starting `yarn babel-node index.js`
$ /Users/USER/CleanCode/backend/node_modules/.bin/babel-node index.js
It works!
[nodemon] clean exit - waiting for changes before restart
```

这正是我们想要的。它向我们展示了我们能够使用 ES6+语法运行`index.js`文件。它做的不多，但这是一个很好的开始，也正是我们需要的开始。

# 下一步是什么？

在本文中，我们学习了如何在 ExpressJS 服务器中启用 ES6+ JavaScript 语法。在下一篇文章中，我们将学习“Redux style”导出/导入技术，它可以确保更准确的代码和更少的麻烦。敬请期待！