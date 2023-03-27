# 使用 i18next 实现 React (Mobx)应用程序的最终本地化

> 原文：<https://itnext.io/ultimate-localization-of-react-mobx-app-with-i18next-efab77712149?source=collection_archive---------1----------------------->

![](img/0b7fbc965bf6fb0f8d6f5d45002de868.png)

# 为什么？

最近，我不得不为我的 React Mobx web 应用程序添加一个本地化版本。我想要避免的是从头开始编写区域解析机制的代码(在我的生活中又一次)。

我真的很累，因为:

*   龙`.json`文件
*   检查`navigator.language`
*   将`en-US`转换为`en`
*   做类似乏味的事情

我一直在寻找一个解决方案，可以让我专注于我的代码——但似乎我已经找到了一个，实际上超出了我的预期。

向[i18 下一个](https://www.i18next.com/)问好！

# i18 下一步

它不仅仅是一个翻译和多元化价值的图书馆，还有更多的功能。它是完成所有这些工作的框架，为您提供完整的本地化解决方案。

在阅读它的文档时，我对它的功能印象深刻。从其他方面来看，文档对我来说似乎很复杂，而且我不得不检查一些设置在实际应用中意味着什么。所以我决定写这篇文章作为一个小知识库。

在开始之前，我想提一下 i18next 是纯 JavaScript 解决方案。所以你可以用它搭配任何你喜欢的框架( [React](https://reactjs.org/) 、 [Angular](https://angular.io/) 、 [Vue](https://vuejs.org/) )。

# 反应和 i18 下一步

i18next 有一个强大的插件和连接器系统，可以连接到你正在使用的框架。当我们谈论 React app 时，我们将不得不使用两个主要库:

*   [i18 下一个](https://www.i18next.com/)实际上是本地化框架
*   [React-i18 下一个](https://react.i18next.com/)为您的 [React](https://reactjs.org/) 应用程序设置绑定

就像 [mobx](https://mobx.js.org/) 和 [mobx-react](https://github.com/mobxjs/mobx-react) 或者 [redux](https://redux.js.org/) 和 [react-redux](https://github.com/reduxjs/react-redux) 一样。

# 我们开始吧

首先，您必须将这两个依赖项添加到您的应用程序中。

```
npm install i18next react-i18next
```

我建议创建一个名为`i18n.js`的单独文件，我们将为您的本地化过程保存配置。它将在开始时很小，但会随着你添加越来越多的配置选项和插件而增长。所以我们从这样的东西开始。

```
/
 - i18n.js
 - index.jsx
```

文件的内容看起来像:

```
// i18n.jsimport i18n from 'i18next';i18n.init({
  debug: true,

  lng: 'en-US', resources: {
    'en-US':{
      'translation': {
        intro: 'Hello my name is'
      }
    }
  }, react: {
    wait: false,
    bindI18n: 'languageChanged loaded',
    bindStore: 'added removed',
    nsMode: 'default'
  }
});export default i18n;// index.jsximport React from 'react';
import ReactDOM from 'react-dom';
import { I18nextProvider, withNamespaces } from 'react-i18next';import i18n from './i18n.js';@withNamespaces()
class App extends React.Component {
  render() {
    const { t } = this.props; return (
      <span>{t('intro')} Viktor</span>
    );
  }
}ReactDOM.render((
  <I18nextProvider i18n={i18n}>
    <App />
  </I18nextProvider>
  ), document.getElementById('mount')
);
```

*执行后，我们会看到一个网页，上面写着“你好，我叫维克多”*

让我解释一下刚刚发生的事情🤷🏻‍♂️

为了在你的应用中使用 i18next，它需要被初始化。`i18n.init()`用于此。方法接受大量不同的属性。但是我们将在这里集中讨论我们擅长开始更标准的方法。

*   `debug: true` —在开发过程中非常有用，它记录控制台关于 i18next 状态的所有信息。比如初始化的时候，设置了什么语言，如果某些翻译丢失了就发出通知。*生产中考虑切换为* `*false*` *。如果使用 webpack 设置* `*NODE_ENV*` *变量并像使用* `*debug: process.env.NODE_ENV === 'development'*` *一样使用它。*
*   `lng: 'en-US'` —要使用的初始语言。稍后我们将看到如何动态地识别语言，避免在 init 上硬编码。
*   `resources` —翻译的捆绑称为资源。可以在`init`方法中定义它们，在运行时用`i18n.addResourceBundle`方法加载它们，或者让它们(我喜欢这一部分)基于所选语言异步获取。你应该传递一个下一个结构的对象`{localeName: {namespaceName: {key: 'value'}}`默认名称空间是`translation`，所以我在例子中传递了它。

> 名称空间是 i18next 的一个非常酷的特性。它允许您将翻译转移到单独的文件中。因此，您可以按页面加载翻译，而不是一次下载巨大的`.json`文件并阻止您的 web 应用程序内容出现。因为本文的目标只是给出本地化的鸟瞰图——我们将只使用默认名称空间(`translation`)。

*   `react` —是一个具有 react-18next 插件配置属性的对象。我相信没有人能比[官方文件](https://react.i18next.com/components/i18next-instance)解释得更好了。在上面的例子中，它使用默认值。如果您不打算更改它们，只需省略 i18next init 的`react`属性。

在`index.jsx`你也可以看到一些不同之处:

*   `I18nextProvider` —这是`react-i18next`库暴露的组件。它接受一个属性`i18n`和你的实例`i18n`。检查我们是否在开始时导入了它。`I18nextProvider`是一个[上下文提供者](https://reactjs.org/docs/context.html)。允许上下文消费者通过 React 上下文接收一些附加参数。
*   `@withNamespaces()` —我可爱的部分。装饰器`withNamespaces`也是从`react-i18next`库中导入的。基本上这是一个[反应高阶组件](https://reactjs.org/docs/higher-order-components.html)。传递给组件 2 的附加属性。`t`功能和`i18n`实例。
*   `t` —调用函数`t('intro')`将检查`intro`键的翻译是否存在于所选语言中(在我们的例子中为`en-US`)，以及*默认的*名称空间(`translation`)。如果找到资源，这将输出一个翻译，或者将返回一个键。*注:如果您处于调试模式，您将在浏览器控制台中看到一条消息，说明未找到翻译*。

# 动态获取用户语言

当然我们不想在`i18n`的`init`方法中硬编码用户区域设置。更重要的是，我们不想写代码来解决这个问题。图书馆可以帮助我们。所以…

```
npm install i18next-browser-languagedetector
```

让我们对`i18n.js`文件做一些修改

```
// i18n.jsimport i18n from 'i18next';
import LanguageDetector from 'i18next-browser-languagedetector';i18n.use(LanguageDetector)
.init({
  debug: true, resources: 
    'en-US':{
      'translation': {
        intro: 'Hello my name is'
      }
    }
  }
});export default i18n;
```

我们已经删除了`lng: 'en-US'`并添加了一个调用特殊`i18n`方法`use`。这个函数用于为`i18n.`加载额外的插件，所以在这种情况下，我们受益于 [LanguageDetector](https://github.com/i18next/i18next-browser-languageDetector) 插件。它检查多个地方，并为您解析用户区域设置。

> 我们还删除了依赖默认值的`react`属性

# 从外部位置加载翻译文件

没有人在项目源代码中保存长的`.json`翻译文件。大多数情况下，它们都是独立的文件(可能会放在 CDN 上，所以不会和你的源代码放在一起)。让我们看看如何上传一个正确的。

我们将再次受益于`i18next`可插拔架构。

```
npm install i18next-xhr-backend;
```

并做一些代码修改

```
// i18n.jsimport i18n from 'i18next';
import XHR from 'i18next-xhr-backend';
import LanguageDetector from 'i18next-browser-languagedetector';i18n*.use(XHR)* .use(LanguageDetector)
.init({
  debug: true
});export default i18n;
```

应用`XHR`插件后。我们可以在开发工具的网络选项卡中看到 3 个额外的请求。

```
http://app.com/locales/en-US/translation.json
http://app.com/locales/en/translation.json
http://app.com/locales/dev/translation.json// pattern looks like
/locales/{language}/{namespace}.json
```

我问自己的第一个问题是“什么是`dev`语言？”。但在小范围阅读后，它碰巧是 i18next 的一个未被发现的酷功能。

*   保存缺失— i18n 可以将缺失的翻译保存为`dev`语言，稍后您的翻译人员可以使用它来编写正确的英语短语和其他翻译。
*   语言(和名称空间)回退—如果在`en-US`语言中找不到您的翻译，将在`en`语言中进行检查。(以及之后的`dev`)。

> 对于生产来说，最好不要使用`dev`语言。您可以通过向`i18n.init({ fallbackLng: 'en' })`传递附加参数来控制这一点。这也将把对语言文件的请求量减少到 2。

甚至还有更大的调整潜力。许多应用程序只运行少数语言环境，并且只使用语言代码，如`en`跳过区域代码`US`。如果这适用于你的情况。使用`i18n.init({ load: 'languageOnly' })`。您将只看到一个对本地文件的 http 请求。

当然，您可以更改加载翻译文件的路径

```
// i18n.jsconst backendOpts = {
  loadPath: `myCusomPathToLocales/{{lng}}/{{ns}}.json`
}i18n.init({
  backend: backendOpts
})
```

> 提示:您可以使用 webpack 中的`[publicPath](https://webpack.js.org/guides/public-path/)`T21 来构建本地文件的正确路径。例如 cdn。

# 编写翻译文件

你的翻译文件是纯`.json`文件。最酷的是 i18next 支持开箱即用的嵌套。因此，您可以在翻译文件中创建有意义的结构，而不是纯粹的 1 级深度列表

```
// locales/en-US/translation.json{
  "common": {
    "confirm": "Confirm",
    "cancel": "Cancel"
  },
  "question": "Use i18next?"
} // App.jsx@withNamespaces()
class App extends React.Component {
  render() {
    const { t } = this.props; return (
      <span>{t('question')}</span>
      <button>{t('common.confirm')}</button>
      <button>{t('common.cancel')}</button>    
    );
  }
}
```

# 使用 Mobx 更改区域设置

Mobx 提供了一种奇妙的可能性，可以在一些可观察的属性发生变化后，在应用程序中产生副作用。我们将用它们来转换语言。假设我们在`appStore.locale`属性中保存了用户选择的区域设置。

那么反应会是这样的:

```
// index.jsxreaction(
  () => appStore.locale,
  locale => {
    i18n.changeLanguage(locale);
  }
);
```

令人惊叹的是，新的翻译将被异步加载。所有 React 组件都将重新呈现，以新语言显示文本。没有更多丑陋的整页重新加载🎉！

# 真实的例子

点击查看真实用例示例[。不要忘记打开控制台来跟踪正在发生的事情。](https://codesandbox.io/s/n5jmk0q69j)

# 准备好上钩了吗？

> 注意:该功能还处于试验阶段，因此 API 可能会发生变化，您可能会遇到意想不到的行为。在遵循这个方法之前，验证您的 React 版本是`16.7.0-alpha`和 react-i18next `8.2.0`或更高版本。

如你所知，从`v 16.7.0-alpha`开始，React 团队引入了一个叫做[钩子](https://reactjs.org/docs/hooks-intro.html)的新概念，旨在简化你如何在组件间重用有状态逻辑的方法。以及惊人的快速反馈(不到 24 小时😲)来自 i18next 战队。因此，如果你准备尝试一些真正前沿的东西，你可以用 react-i18next 钩子来本地化你的应用程序:

```
*// i18n.js*import i18n from 'i18next';
import XHR from 'i18next-xhr-backend';
import LanguageDetector from 'i18next-browser-languagedetector';
**import { initReactI18n } from 'react-i18next/hooks';**i18n*.use(XHR)* .use(LanguageDetector)
**.use(initReactI18n)**
.init({
  debug: true
});export default i18n;*// index.jsx*import React from 'react';
import ReactDOM from 'react-dom'; **import { useTranslation } from 'react-i18next/hooks';**import './i18n.js'; // you still need it in bundlefunction App() {
  **const [t, i18next] = useTranslation();** return <span>{t('intro')} Viktor</span>;
}ReactDOM.render((
    <App />
  ), document.getElementById('mount')
);
```

*   `use(initReactI18n)`——通过调用这个插件，在初始化阶段——我们使 i18next 实例可用于`useTranslation`挂接任何 React 组件
*   `useTranslation()` — i18next React Hook，一个为你的功能组件提供`t`函数(我们已经很熟悉了)和 i18next 实例的函数

您不会不同意使用这种方法代码看起来更干净。此外，它给你更多的好处。在这一点上，我们的`App`功能组件不再用`@withNamespaces` HOC 包装——所以当你为了测试对它进行浅层渲染时，可以很容易地观察到它的结构。你可以去看看。

# 包扎

i18next 是一个非常有趣和强大的本地化应用程序的解决方案。它允许大量的定制和调整。用一系列插件和工具使基础设施饱和。

本指南仅涵盖 i18next 广泛功能中的小部分基本功能。因此，请查看它的[文档](https://www.i18next.com/)以获得更深入的信息。

# 感谢阅读！如果你喜欢这篇文章，我会真诚地感谢你点击鼓掌👏按钮或分享这个故事。您的反馈非常重要！