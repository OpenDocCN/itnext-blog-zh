# 掌握会话认证

> 原文：<https://itnext.io/mastering-session-authentication-aa29096f6e22?source=collection_archive---------0----------------------->

## 关于如何使用 MongoDB、Express、React 和 Node 构建基于会话的身份验证应用程序的完整演练

![](img/21c093cc03a807a69f908aa30ee0fc39.png)

原始照片由[塞缪尔·泽勒](https://unsplash.com/@samuelzeller)拍摄

今天，我们开始了构建 MERN 堆栈认证应用的旅程，该应用将提供:输入验证、错误处理、会话认证、受保护的前端路由和持久登录——即使服务器重启或用户刷新页面。

这是一个*一步一步的指南/教程*，因为我会尽可能详细地介绍，同时在需要时提供文档链接。
在我们开始之前，这里有一份我们将使用的技术清单:
**MongoDB Atlas、Mongoose、Joi、BCrypt、ESM、Express、Express-Session、React、Redux、Node.js、Postman、** heavy **ES6+** 用法

就是这样！这几乎是我们整个项目所需要的一切。
**我们将不使用 Passport.js 或 JWTs** 。
最后，我喜欢造型，但我们的用户界面将会非常简单&无风格。这样我们就可以专注于业务逻辑，并且这个项目可以作为未来项目的框架。

好了，事不宜迟，我们开始吧！

# 步骤 1: Node.js 设置

让我们首先创建一个项目目录并初始化我们的节点应用程序。如果你还没有安装 Node.js，去[他们的网站](https://nodejs.org/en/)按照下载说明操作。

一旦你这样做了，在你的终端中运行这三个命令:
`mkdir SessionAuth && cd SessionAuth`
`mkdir backend && cd backend`
`npm init esm -y`。esm 将允许我们在后端使用心爱的导入/导出语法。
**-y** 简单跳过填写姓名、描述、作者等的问题。(如果您愿意，您可以更新这些内容)
**NPM init ESM-y**命令为我们创建了一些东西，但是下一步是设置我们的项目目录结构。这里我将使用一个非常简单的方法:

```
SessionAuth/
   **|— backend/**
      |— node_modules/
      **|— .gitignore**
      |— package-lock.json
      |— package.json
      **|— src/**
         **|— models/
         |— routes/
         |— util/
         |— validations/**
         |— index.js
         |— **server.js** <= renamed *main.js* to *server.js*
```

****** `**All boldened lines in the code snippets represent new code.**` **
我将 **main.js** 重命名为 **server.js** ，因为这将包含所有与服务器相关的逻辑。让我们快速更新我们的 **index.js** 导入:

```
*--- backend/src/index.js ---*
require = require("esm")(module/*, options*/)
module.exports = require("**./server.js**")
```

**。gitignore** 文件是可选的，但是如果你正在使用 git，我*强烈*建议现在创建一个并添加这两行:

```
*--- backend/.gitignore ---***/node_modules
config.js**
```

**node_modules** 文件夹不需要签入版本控制，我们还没有创建 **config.js** 文件，但是现在最好忽略它。
在你的文件结构看起来像上面这样之后，让我们在我们的终端中确保我们在**后端**目录中。如果没有，`cd backend/`。现在我们可以安装后端依赖项了。下面是终端命令:
`npm i bcryptjs connect-mongo crypto-js express express-session joi mongoose` 我会在我们使用它们的时候解释每一个包，所以现在请耐心等待。我们还想安装一个名为 **nodemon** (节点监视器)的开发依赖项。这将使我们在修改时不必重启服务器:`npm i -D nodemon` 现在让我们整理一下我们的**包**

```
*--- backend/package.json ---* {
  "name": "backend",
  "version": "1.0.0",
  "description": "",
  "main": "**./src/index.js**",
  "module": "**./src/server.js**",
  "scripts": {
 **"start": "node index.js",
    "dev": "nodemon index.js"**
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "connect-mongo": "^2.0.3",
    "crypto-js": "^3.1.9-1",
    "esm": "^3.2.20",
    "express": "^4.16.4",
    "express-session": "^1.15.6",
    "joi": "^14.3.1",
    "mongoose": "^5.4.19"
  },
  "devDependencies": {
    "nodemon": "^1.18.10"
  }
}
```

我们更新我们的“main”和“module ”,以查看我们创建的 **src/** 文件夹。
我们去掉了“**测试**”脚本，添加了“**启动**”和“**开发**”脚本。在本演练中，我们将主要使用 dev 脚本。记下所有的依赖项和我们的 devDependencies，并仔细检查你的文件看起来是否相同，但是不要担心你的版本号是否不同。只要你像我一样安装它们，你的就会是最新的。

现在我们可以开始玩了！

# 步骤 2:服务器设置

让我们从基础开始。在 **src/** 文件夹中创建一个名为 **config.js** 的文件(还记得我们忽略的那个吗？).这个文件将包含我们所有的环境变量。在生产过程中，这些将由[**process . env**](https://nodejs.org/api/process.html#process_process_env)**对象设置。我们稍后将在这里添加更多的变量，但现在，我的看起来是这样的:**

```
*--- backend/config.js ---* **export const {
  PORT = 5000,
  NODE_ENV = 'development'
} = process.env;**
```

**这可能看起来很有趣，但是我们在这里所做的就是从 **process.env** 对象中析构 **PORT** 和 **NODE_ENV** 变量，并在它们不存在的情况下给出默认值(在我们的例子中，它们不存在——默认值是我们将要使用的)。除了 3000，你可以选择任何端口*。我们将为我们的客户节省 3000 英镑。***

**好了，现在进入我们的 **server.js** :**

```
*--- backend/server.js ---* **import express from 'express';
import { PORT, NODE_ENV } from './config';****const app = express();****app.disable('x-powered-by');****app.use(express.urlencoded({ extended: true }));
app.use(express.json());****app.listen(PORT, () => console.log(`Listening on port ${PORT}`));**
```

**我们导入 **express** 函数，调用它——返回一个对象——并将其保存到常量 **app** (常规命名)中。
接下来，我们禁用**‘x-powered-by’**，这使得用户更难看出我们在使用 Express。为什么这是一件好事？黑客对我们的堆栈了解得越少越好。
然后，我们添加两行中间件，记为 **app.use(** *中间件* **)** 。我们过去必须安装一个名为 body-parser 的独立依赖项，但是现在 Express 内置了这些好东西。这两行允许我们解析 HTTP 请求的主体和 JSON 有效负载。
然后，我们调用应用程序上的 **listen** 方法，从 **config.js** 传入我们的端口和一个回调，让我们知道我们已连接。让我们前往我们的终端，看看它是否工作！**

**![](img/763a59f2f86bdd7631953b5ceed20730.png)**

**运行`npm run dev`之后，我们应该会看到输出“监听端口 5000”。
转到[http://localhost:5000/](http://localhost:5000/)
你应该看到“**无法得到/**”
听起来是个错误，我知道，但这正是我们想要的。**

**那好吧。开始路由吧。在我们的 routes 文件夹中，让我们创建两个文件: **index.js** 和 **user.js** 。我们的索引路由将简单地导出一个包含所有后端路由的对象。现在，它看起来像这样:**

```
*--- backend/routes/index.js ---* **import userRoutes from './user';**// syntactic sugar for { userRoutes: userRoutes }
**export { userRoutes };**
```

**对于我们的用户路线，让我们这样做:**

```
*--- backend/routes/user.js ---* **import express from 'express';****const userRoutes = express.Router();****userRoutes.post("", (req, res) => {
  res.send(req.body);
});****export default userRoutes;**
```

**这里我们使用 **express 创建一个路由器。Router()** 我们在上面定义了一个“post”方法，对应一个 POST 请求。第一个参数是路径(但它是一个空字符串！一会儿再详细介绍)，第二个是接收请求和响应对象的回调。在这个回调函数中，我们定义了我们希望路由做什么。这里我们告诉响应“发送”回请求的“主体”。**

**最后，回到我们的 **server.js** :**

```
*--- backend/server.js ---* import express from 'express';
**import { userRoutes } from './routes/index';**
import { PORT, NODE_ENV } from './config';const app = express();app.disable('x-powered-by');app.use(express.urlencoded({ extended: true }));
app.use(express.json());**const apiRouter = express.Router();
app.use('/api', apiRouter);
apiRouter.use('/users', userRoutes);**app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
```

**在我们的 **server.js** 中，导入我们的 **userRoutes** ，然后做两件事:
1。我们像之前一样设置了另一个路由器，名为 **apiRouter** ，并告诉我们的**应用**对任何以“ **/api** 开头的路径使用这个路由器。
2。然后我们告诉我们的 **apiRouter** 使用我们的 **userRoutes** ，对于任何以“ **/users** 开头的路径。
这就是为什么我们在 **routes/user.js** 中使用空字符串作为路径——因为我们在 server.js 中提供了“/users”路径。**

**现在我们来测试一下！我将使用[邮递员](https://www.getpostman.com/)。在我写这篇文章的时候，Google Chrome 扩展已经过时了，所以我正在使用可以免费下载的原生应用。您可以随意使用任何选项来测试 HTTP 请求。**

**![](img/b1fbdf9dbc963ca09351f61fb226e0b8.png)**

**如果这是你第一次使用 Postman，它可能看起来像有一百万个按钮、旋钮和下拉框。我已经突出显示并标记了我们测试的一步一步的工作流程。**

**如果一切顺利，我们应该得到一个 200 级别的响应，从请求中发送回主体。我们进展顺利。让我们保持下去！🙌**

# **步骤 3:MongoDB/mongose 集成**

**该去实地考察了！让我们创建我们的 MongoDB Atlas 帐户。我记得第一次使用 Atlas 时，我感到非常失落…所以我们将一起做这件事。(如果你是地图集专家，请直接跳过)。**

**1.首先去 https://www.mongodb.com/[创建一个免费账户。一旦你的帐户被创建，你应该被引导到一个页面，在那里你可以创建你的第一个“集群”。选择您的首选项，我使用了所有预先选择的选项:AWS、my region、free-tier storage，并且没有费心更改集群名称。然后将开始集群创建过程(这可能需要几分钟时间)。
2。在“Clusters”页面中，单击页面中间附近的 **Security** 选项卡，然后单击“**Add new user”**按钮。创建用户名/密码，并使用所有默认设置。我们希望这个用户拥有“**读写权限**”。
3。接下来，仍然在**安全**选项卡中，单击 **IP 白名单**子选项卡。然后点击**添加 IP 地址**按钮和**添加当前 IP 地址**按钮。(我们快好了！)
4。现在，转到安全选项卡旁边的**概述**选项卡。在页面中间附近，单击**连接**，然后单击“**连接您的应用程序”**按钮(对我来说，这是中间的选项)。复制提供的连接字符串。我们将在我们的 **config.js** 中使用它:](https://www.mongodb.com/)**

```
*--- backend/config.js ---* export const {
  PORT = 5000,
  NODE_ENV = 'development', **MONGO_URI = 'mongodb+srv://*YOURUSERNAME*:*YOURPASSWORD*@cluster0bdxbr.mongodb.net/testretryWrites=true'**} = process.env;
```

**提醒:如果你正在使用 GitHub 或者你正在上传你的代码，在你的**中忽略这个配置文件。gitignore** 。
**不要分享我们放在 config.js 中的任何信息** 请确保填写您的真实用户名&密码。
现在，让我们更新我们的 **server.js** 并连接这个数据库。**

```
*--- backend/server.js ---* import express from 'express';
**import mongoose from 'mongoose';**
import { userRoutes } from './routes/index';
import { PORT, NODE_ENV, **MONGO_URI** } from './config';**(async () => {
**  **try {**
    **await mongoose.connect(MONGO_URI, { useNewUrlParser: true });
    console.log('MongoDB connected');** const app = express();
    app.disable('x-powered-by');
    app.use(express.urlencoded({ extended: true }));
    app.use(express.json()); const apiRouter = express.Router();
    app.use('/api', apiRouter);
    apiRouter.use('/users', userRoutes); app.listen(PORT, () => console.log(`Listening on port ${PORT}`));
  **} catch (err) {
    console.log(err)**
  **}
})();**
```

**哇…这是怎么回事？我们来分解一下。
首先我们导入 [**猫鼬**](https://mongoosejs.com/) ，一个对象数据建模(ODM)库。我们还将**蒙戈 _URI** 添加到我们的配置导入中。
然后，我们使用 mongoose 连接到我们的 MongoDB Atlas，使用我们导入的 URI 和一个包含`useNewUrlParser: true`的 options 对象(这允许我们的 URI 被解析)。
整个过程是*异步的*，我们不希望我们的服务器在数据库连接之前启动。
我们*可以*用**。然后是**，但是对于整个应用程序，我们将使用 [**异步/等待**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 。为了适应这种情况，我们将所有代码包装在一个异步的[life](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)(立即调用的函数表达式)中。
最后，我们使用一个 **try/catch** 块，如果出现问题，它将记录控制台中的任何错误。如果一切顺利，我们应该会看到两个 **console.log** 输出。**

**![](img/5258fcd0a3e495787de427d67506200d.png)**

# **步骤 4:模型和验证**

**这一步可能有点复杂，但是不要担心！我们在一起。
在我们的 **models/** 文件夹中，创建一个 **user.js** 文件。
我们将使用 [Bcrypt](https://www.npmjs.com/package/bcryptjs) 对用户的密码进行加盐和哈希处理，然后将它们保存到数据库中。**切勿保存纯文本密码**。
我使用 BCrypt 的 **sync** 版本函数(它使用 **async/await** )来保持我们的代码更干净。我把这个分成两部分:**

```
*--- backend/routes/user.js ---* **import mongoose from 'mongoose';
import { compareSync, hashSync } from 'bcryptjs';****const UserSchema = new mongoose.Schema({
  username: {
    type: String,
    validate: {
      validator: username => User.doesNotExist({ username }),
      message: "Username already exists"
    }
  },
  email: {
    type: String,
    validate: {
      validator: email => User.doesNotExist({ email }),
      message: "Email already exists"
    }
  },
  password: {
    type: String,
    required: true
  }
}, { timestamps: true });
...**
```

**我们来分析一下。我们从导入**mongose**和我们需要的 Bcrypt 函数开始。
接下来，我们定义一个**用户模式**。这基本上是如何将我们的用户对象保存在我们的数据库中的蓝图。我们的用户很简单，只需要用户名、电子邮件和密码([更多关于 mongoose 模式)](https://mongoosejs.com/docs/guide.html)。我们指定这些字段将是**字符串**，为用户名和电子邮件添加一个定制的**验证器**，为密码添加一个简单的**必需的**验证器，以及**时间戳**。我们需要在这里再补充一点:**

```
*--- backend/routes/user.js ---* **...
UserSchema.pre('save', function () {
  *if* (*this*.isModified('password')) {
    *this*.password = hashSync(*this*.password, 10);
  }
});****UserSchema.statics.doesNotExist = async function (field) {
  *return* *await* *this*.where(field).countDocuments() === 0;
};****UserSchema.methods.comparePasswords = function (password) {
  *return* compareSync(password, *this*.password);
};****const User = mongoose.model('User', UserSchema);
export default User;**
```

**接下来，我们需要为我们的 UserSchema 添加一个“pre-hook”**或“中间件”。这段代码将在我们的用户实例保存之前运行。由于词法范围的原因，我们不能对这三种方法使用箭头函数。
**这个**就是我们的 **pre** 方法所指的用户实例。我们仔细检查是否提供了一个密码，然后将该密码重置为它的 salted 和 hashed 版本。
然后我们定义我们的自定义验证器，**不存在，**，它是在我们的 **UserSchema** 上定义的*类方法*。使用两个 mongoose 方法和一个比较操作符，我们可以确定用户是否存在。这是一个异步步骤，所以我们使用 async/await。
我们还定义了一个*实例方法*、 **comparePasswords、**，它简单地将一个纯文本密码与一个用户实例的散列密码进行比较。
最后，我们明确定义或**用户**模型并导出。****

****下一站:验证。我们将使用 [Joi](https://www.npmjs.com/package/joi) 进行验证。但是你可能会问，“我们不是刚刚在模型中添加了验证吗？”是的……但是，这些本质上是我们的数据库级验证，我们希望在创建用户实例之前，在上面再检查一层验证。在 Joi 的帮助下，我们也可以变得更具体。在我们的 **validations/** 文件夹中创建一个 **user.js** 文件，如下所示:****

```
***--- backend/validations/user.js ---* **import Joi from 'joi';****const email = Joi.string().email().required();
const username = Joi.string().alphanum().min(3).max(30).required();****const message = 'must be between 6-16 characters, ' +
  'have at least one capital letter, ' +
  'one lowercase letter, one digit, ' +
  'and one special character';****const password = Joi.string()
  .regex(/*^*(?=.*[A-Za-z])(?=.*\d)(?=.*[!@#$%^&*])[a-zA-Z0-9!@#$%^&*]{6,16}*$*/)
  .options({
    language: {
      string: {
        regex: {
          base: message
        }
      }
    }
});****export const signUp = Joi.object().keys({
  email,
  username,
  password
});****export const signIn = Joi.object().keys({
  email,
  password
});****
```

****由于[文档](https://www.npmjs.com/package/joi)很精彩，我不会花太多时间解释这里发生了什么。我们看到我们定义了一个**电子邮件**、**用户名**和**密码**，每个都附加了验证。默认情况下，Joi 会在某些内容无效时提供一条消息，但是对于密码字段，它实际上会打印出用户猜测的密码。密码应该永远不可见。因此，我们创建了一个定制的**消息**并深入到**选项**对象中来传递它。然后，我们使用正则表达式来获取密码验证的具体细节。
接下来，我们使用我们的**电子邮件**、**用户名**和**密码**来创建和导出两个 **Joi 对象**。我们将使用这些来验证用户的输入。****

****让我们回到我们的 **routes/user.js** 文件:****

```
****import Joi from 'joi';**
import express from 'express';
**import User from '../models/user';
import { signUp } from '../validations/user';****const userRouter = express.Router();
userRouter.post("", async (req, res) => {
  *try* {
    const { username, email, password } = req.body
    *await* Joi.validate({ username, email, password }, signUp);** **const newUser = new User({ username, email, password });
    *await* newUser.save();
    res.send({ userId: newUser.id, username });
  } *catch* (err) {
    res.status(400).send(err);
  }
});**export default userRouter;**
```

****我们引入了 **Joi** ，我们的**用户**模型，以及我们的**注册**验证器。
然后我们修改我们的 **post** 方法，使用 **async** 和一个 **try/catch** 块。
接下来，我们析构 **req.body** 并调用 **Joi.validate** 方法，传递一个包含用户输入和我们的**注册**验证器的对象。如果通过，我们定义一个**新用户**实例并尝试保存该用户。如果通过了，我们就发送回我们希望我们的前端可以访问的内容。如果有任何失败，它将被捕获，我们通过将状态代码设置为 **400** 并发回错误来做出响应(我们将在以后改进我们的错误处理)。有人可能会问，“如果 req.body 只包含一个包含用户名、电子邮件和密码的 JSON 对象，为什么还要将其析构呢？为什么不直接将整个主体传递给我们的验证器和新的用户函数呢？”嗯……问题是我们不知道这是否是身体所包含的全部。我们已经看到使用 Postman，用户可以在请求的主体/参数中放入任何他们想要的东西。有了我们定义的验证器和模式，这里没有太大的风险，但是一个好的经验法则是永远不要相信来自用户的输入。****

****让我们用 Postman 来测试一下！****

****![](img/c53ec00da312795233619b17d2cb4420.png)****

****就服务器而言，这里的情况看起来很好。注意到我们得到的回应了吗？我们的 Redux 商店以后会喜欢的。
现在，让我们转到我们的 MongoDB Atlas 帐户，从主集群页面中，单击“**集合**”。****

****![](img/999057f015c328167de806c1b8434c6c.png)****

****我们的用户来了！您可以看到密码确实经过了哈希处理。****

****![](img/8c355e6ca087f03a53acecc0efc8be04.png)****

****如果我们试图输入任何无效的内容，我们会得到一个包含一条好消息的响应，稍后我们将在客户端使用这条消息向用户显示。****

****在进入前端之前，我们还有一个后端步骤要完成。
现在不能停！✊****

# ****第五步:会话****

****让我们进入正题。在我们的 **config.js** 中，我们需要再添加几行代码:****

```
***--- backend/config.js --* ...
  **SESS_NAME = 'sid',
  SESS_SECRET = 'secret!session',
  SESS_LIFETIME = 1000 * 60 * 60 * 2**
} = process.env;**
```

****这里我们析构/定义一个会话**名称**，**秘密**，和**寿命**(我只用了两天)。这个 **SESS_SECRET** 用于使用 HMAC 散列 sessionID。这个秘密通常应该是在您的环境中定义的深奥的东西(例如 Heroku 等)，但是对于开发来说，这是可以的。 **SESS_NAME** 稍后将用于引用我们的会话。****

****好了，让我们回到我们的 **server.js** 文件。我会把它一分为二:****

```
***--- backend/server.js ---*
import express from 'express';
import mongoose from 'mongoose';
**import session from "express-session";**
**import connectStore from "connect-mongo";**
import { userRoutes } from './routes/index';
import {
PORT, NODE_ENV, MONGO_URI, **SESS_NAME**, **SESS_SECRET**, **SESS_LIFETIME**
} from "./config";(async () => { 
...**
```

****首先，我们处理一些进口商品。
使用**快速会话**时，我们应该注意一些事情:****

> ********警告*** 默认的服务器端会话存储`*MemoryStore*`，是*特意*不为生产环境设计的。它在大多数情况下会泄漏内存，不会扩展到单个进程之外，并且只用于调试和开发。”
> *—* [*快速-会话文档*](https://github.com/expressjs/session)*****

*****啊哦。我们应该做点什么，对吗？
进入 [**连接-mongo**](https://github.com/jdesboeufs/connect-mongo) 。这将把我们的会话信息保存到数据库中，并允许在我们的服务器关闭或重启时重新建立连接。当我们*注销时销毁*我们的会话时，这个会话信息会自动从我们的数据库中删除。很酷，对吧？让我们继续前进:*****

```
****--- backend/server.js ---*
    ...
    const app = express();
    **const MongoStore = connectStore(session);** app.use(express.urlencoded({ extended: true }));
    app.use(express.json());
 **app.use(session({
      name: SESS_NAME,
      secret: SESS_SECRET,
      saveUninitialized: false,
      resave: false,
      store: new MongoStore({
        mongooseConnection: mongoose.connection,
        collection: 'session',
        ttl: parseInt(SESS_LIFETIME) / 1000
      }),
      cookie: {
        sameSite: true,
        secure: NODE_ENV === 'production',
        maxAge: parseInt(SESS_LIFETIME)
      }
    }));
    ...*****
```

*****我们调用 **connectStore** 传入我们的**会话**中间件，并将其保存到常量 **MongoStore** 中。
现在，我们告诉我们的**应用**使用**会话**中间件，并传入一个选项对象。我们已经覆盖了**名称**和**秘密**，但是我们又传递了两个选项:
`saveUninitialized: false`；这符合在设置 cookie 之前需要许可的法律。
T1；如果会话未被修改，这可以防止不必要的重新保存。
商店**选项是我们插入 **MongoStore** 的地方。我们将通过它的三个选项:
`mongooseConnection: mongoose.connection`；这使用了我们在上面建立的连接，这样我们就不必连接 Mongo 两次。
T3；这指定了我们的会话信息在数据库中的保存位置。
T4；(“生存时间”)我们希望这与我们的 **SESS_LIFETIME** 相匹配，除了 connect-mongo 使用秒而不是毫秒，所以我们除以 1000。我们在这里和下面也使用 **ParseInt()** ，因为一些环境变量在生产中经常被设置为字符串。我们想要一个数字。
接下来给我们的 **cookie** 一些选项:
`sameSite: true`；这有助于防止 CSRF 袭击。
`secure: NODE_ENV === ‘production’`；我们只在生产过程中将 cookie 设置为安全的。*成为* **真***需要一个支持 https 的网站。*
**maxAge** 简单地指定了我们的 cookie 的年龄。
我只是为这些选项提供一个高层次的解释，还有*许多默认选项正在设置*。[文档](https://github.com/expressjs/session)详细解释了所有这些。*******

***在完成后端之前，我们还有两个文件要处理。在创建会话路由之前，我们将在 **util/helpers.js** 中定义两个助手方法:***

```
**--- backend/util/helpers.js ---* **export const parseError = err => {
  if (err.isJoi) return err.details[0];
  return JSON.stringify(err, Object.getOwnPropertyNames(err));
};****export const sessionizeUser = user => {
  return { userId: user.id, username: user.username };
}***
```

***第一个函数将允许我们区分 **Joi** 错误和我们稍后抛出的自定义**错误**并提取消息，我们将消息发送给客户端并显示给我们的用户。
第二个函数返回一个对象，该对象包含我们希望保存到会话中并稍后发送回 Redux 存储的内容。
让我们将我们的错误解析器添加到 **routes/user.js** ，并将我们的用户保存到我们的**会话**:***

```
**--- backend/routes/user.js ---* import Joi from 'joi';
import express from 'express';
import User from '../models/user';
import { signUp } from '../validations/user';
**import { parseError, sessionizeUser } from "../util/helpers";**const userRouter = express.Router();
userRouter.post("", async (req, res) => {
  try {
    const { username, email, password } = req.body;
    await Joi.validate({ username, email, password }, signUp); const newUser = new User({ username, email, password });
    **const sessionUser = sessionizeUser(newUser);**
    await newUser.save(); **req.session.user = sessionUser;**
    res.send(**sessionUser**);
 } catch (err) {
   res.status(400).send(**parseError(err)**);
  }
});export default userRouter;*
```

***在设置了**快速会话**之后，我们可以访问会话对象*，该对象只保存在服务器端*(客户端不能访问该会话)。记住这一点，我们将用户的 id 和用户名保存到我们的会话中。***

***![](img/c7cd0e9dde13d8d8bb7f76a48add0972.png)***

***为了展示它的工作原理，我在我们存储用户的地方放了一个**console . log(req . session)**。然后我用 Postman 创建了另一个用户，他就在那里！***

***好了，现在让我们在 **routes/** 文件夹中创建一个 **session.js** :***

```
**--- backend/routes/session.js ---* **import express from "express";
import Joi from "joi";
import User from "../models/user";
import { signIn } from "../validations/user";
import { parseError, sessionizeUser } from "../util/helpers";
import { SESS_NAME } from "../config";****const sessionRouter = express.Router();** *...**
```

***首先，我们进口我们需要的一切。除了可能 **SESS_NAME** 之外，这里没有什么好惊讶的，你很快就会明白为什么我们需要它。然后，我们定义**会话路由器**，就像我们定义**用户路由器**一样。***

***现在，我们将从登录开始定义三个 API 端点:***

```
**--- backend/routes/session.js ---
...*
**sessionRouter.post("", async (req, res) => {
  try {
    const { email, password } = req.body
    await Joi.validate({ email, password }, signIn);** **const user = await User.findOne({ email });
    if (user && user.comparePasswords(password)) {
      const sessionUser = sessionizeUser(user);** **req.session.user = sessionUser
      res.send(sessionUser);
    } else {
      throw new Error('Invalid login credentials');
    }
  } catch (err) {
    res.status(401).send(parseError(err));
  }
});
...***
```

***我们来谈谈这个。我们在我们的 **sessionRouter** 上定义了一个 post 方法，我们添加了 **async** ，从请求体中提取了 **email** 和**密码**，并使用了我们的**sign in****Joi**validator。
如果通过，我们使用**findOne**mongose 方法查询我们的用户集合，确保使用 **await** 。然后我们检查我们是否得到了一个用户，以及提供的密码是否与散列密码匹配(这里的逻辑顺序很重要)。如果成功，我们将用户会话化，将他们的数据存储在会话对象中，并将他们的信息发送给客户端。否则，我们捕捉任何错误，无论是一个 **Joi** 错误还是我们说`new Error(‘Invalid login credentials’)`时抛出的错误，设置一个失败状态代码，并发回解析后的错误。***

***下一步:注销。***

```
**--- backend/routes/session.js ---
...* **sessionRouter.delete("", ({ session }, res) => {
  try {
    const user = session.user;
    if (user) {
      session.destroy(err => {
        if (err) throw (err);** **res.clearCookie(SESS_NAME);
        res.send(user);
      });
    } else {
      throw new Error('Something went wrong');
    }
  } catch (err) {
    res.status(422).send(parseError(err));
  }
});
...***
```

***这里我们从请求对象中析构出**会话**(因为我们不需要任何其他东西)并将用户从该会话中拉出。如果有，我们调用内置的**销毁**方法。
然后我们在我们的响应上调用**clear cookie(***session _ name****)***方法，将用户发送回客户端。***

***我们的最后一个端点只是检查用户是否登录。这个很简单:***

```
**--- backend/routes/session.js ---
...* **sessionRouter.get("", ({ session: { user }}, res) => {
  res.send({ user });
});****export default sessionRouter;***
```

***我们将请求分解给用户。这要么是我们的用户，要么是未定义的。在任何情况下，我们发送回一个**用户**的对象，该对象指向我们从会话中得到的任何内容。***

***我们只有两个任务要完成了！在 **routes/index.js** 中:***

```
**--- backend/routes/index.js ---* import userRoutes from './user';
**import sessionRoutes from './session';**export { userRoutes, **sessionRoutes** };*
```

***Annnnd in our **server.js** :***

```
**--- backend/server.js ---*
    ...
    const apiRouter = express.Router();
    app.use('/api', apiRouter);
    apiRouter.use('/users', userRoutes);
 **apiRouter.use('/session', sessionRoutes);
    ...***
```

***就是这样！我们的后端现在完成了！🎉给自己一个鼓励。你刚刚从头开始写了一个后端！👍***

# ***步骤 6:反应/还原设置***

***在继续之前，如果这是您第一次使用 React 和 Redux，这些概念可能听起来有点混乱，但是不要担心！它们非常重复，只是需要练习。和往常一样，我将从高层次上解释事情，但如果你感到头晕，我强烈推荐 [React](https://reactjs.org/) 和 [Redux](https://redux.js.org/) 文档。他们太棒了！最后，注意*隐式返回*。我们会用到很多。准备好了吗？***

***让我们将目录向上切换到我们的根项目目录， **SessionAuth** : `cd ..`
现在运行这个命令:`npx create-react-app frontend` 这将设置一个 React 应用程序，而不需要任何构建配置。这给我们制造了相当多的时间，所以我们需要做一些清理工作。
你可以随意保留任何你想要/需要的文件。我对这个应用程序进行了精简，以便它可以在未来的应用程序中重复使用，没有任何负担。我删除了`public/**favicon.ico**, **manifest.json**, src/**App.css**, **App.test.js**, **index.css**, **logo.svg**, **serviceWorker.js**`，并重新整理了我的文件结构，如下所示:***

```
*SessionAuth/
   |— backend/
   |— frontend/
      |— node_modules/
      |— public/
         |— index.html
      |— src/
         **|— actions/
         |— components/** |— App.js <= moved **|— reducers/**
 **|— errors/
            |— session/
         |— store/
         |— util/** |— index.js
 |— .gitignore
 |— package-lock.json
 |— package.json*
```

***我也给 index.html 修剪了一下:***

```
**--- frontend/public/index.html ---* <!DOCTYPE *html*>
<html *lang*="en">
  <head>
    <meta *charset*="utf-8" />
    <meta
      *name*="viewport"
      *content*="width=device-width, initial-scale=1, shrink-to-fit=no"
    />
    <title>**Session Auth**</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
  <div *id*="root"></div>
  </body>
</html>*
```

***下一站是我们的 **App.js** :***

```
**--- frontend/src/components/App.js ---* **import React from 'react';****export default () => (
  <>
    <h1>Hello world</h1>
  </>
);***
```

***在非常高的层面上， [**组件**](https://reactjs.org/docs/components-and-props.html) 只是返回[**【JSX】**](https://reactjs.org/docs/introducing-jsx.html)的函数，这些函数产生 **React 元素**，这些元素随后被渲染到 DOM 中。为了简单起见，我们不会使用任何有状态的组件。
我们将使用[片段](https://reactjs.org/docs/fragments.html)，它允许我们对元素进行分组，而无需向 DOM 添加额外的节点。这里我们用一个来安置我们的 **h1** 。我们过会儿会回到这里。***

***现在为我们的 **index.js** :***

```
**--- frontend/src/index.js ---* import React from 'react';
import ReactDOM from 'react-dom';
import App from '.**/components**/App';
// delete old imports we don't needReactDOM.render(
  <App />, 
  document.getElementById('root')
);*
```

***稍后我们会在这里添加更多的内容，但是现在，这已经很好了。我们正在渲染来自我们的**index.html**的`<div *id*=”root”></div>`内的 **App** 组件，它反过来渲染那个 **h1** 。

还记得我们为客户保存**端口 3000** 的时候吗？嗯，我们仍然需要一种方法让我们的客户端与我们的后端进行通信。为此，我们将使用一个 [**代理**](https://facebook.github.io/create-react-app/docs/proxying-api-requests-in-development) 。
在我们的*前端*底部 **package.json** 我们添加:***

```
**--- frontend/package.json ---* ...
     "browserslist": [
      ">0.2%",
      "not dead",
      "not ie <= 11",
      "not op_mini all"
    ],
  **"proxy": "http://localhost:5000"**
}*
```

***这告诉我们的服务器在我们向“/api/session”这样的东西发出请求时使用我们的后端资源。***

***现在，**在一个单独的终端标签或窗口中，**保持后端服务器运行，将目录更改到我们的 **frontend/** 文件夹:`cd frontend/`，并运行这个命令:`npm start`
现在转到 [http://localhost:3000/](http://localhost:3000/) ，您应该会看到“Hello world”。***

***现在在**另一个单独的**终端标签或窗口中(你现在应该至少打开三个)，确保你在**前端/** 目录中并安装这些依赖项:`npm i react-redux react-router-dom redux redux-thunk` 好的。现在一切就绪，我们可以开始集成 Redux 了！
让我们从在 **util/session.js** 中创建 HTTP 请求函数开始:***

```
**--- frontend/util/session.js ---* **export const signup = user => (
  fetch("api/users", {
    method: "POST",
    body: JSON.stringify(user),
    headers: {
      "Content-Type": "application/json"
    }
  })
);****export const login = user => (
  fetch("api/session", {
    method: "POST",
    body: JSON.stringify(user),
    headers: {
      "Content-Type": "application/json"
    }
  })
);****export const logout = () => (
  fetch("api/session", { method: "DELETE" })
);***
```

***这里我们使用 [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) 来访问我们的三个后端端点。还记得我们在使用 Postman 的时候选择了“**应用/json** ”吗？我们想在这里做同样的事情，但是我们通过使用 **json.stringify()** 来确保我们请求的**主体**是 JSON 格式的。***

***在我们继续前进之前，让我们来讨论一下我们希望如何塑造我们的前端状态——指的是我们的 Redux 存储。***

***当考虑状态形状时，问这两个问题是有好处的。在任何给定时刻，我们的应用程序需要访问哪些数据？
2。我们如何以一种[规范化](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape)的方式组织这些数据？
由于这个应用程序将只包括跟踪会话和注册/登录错误，以下是我现在的想法:***

```
*{
  session: { id, username },
  errors: ""
}*
```

***我们把它保持得非常简单。
现在让我们设置我们的**行动**。在**动作/** 中创建一个 **session.js** :***

```
**--- frontend/actions/session.js ---* **import * as apiUtil from ‘../util/session’;****export const RECEIVE_CURRENT_USER = ‘RECEIVE_CURRENT_USER’;
export const LOGOUT_CURRENT_USER = ‘LOGOUT_CURRENT_USER’;****const receiveCurrentUser = user => ({
  type: RECEIVE_CURRENT_USER,
  user
});****const logoutCurrentUser = () => ({
  type: LOGOUT_CURRENT_USER
});
...***
```

***在这里，我们将获取函数作为 **apiUtil** 导入，因此在使用**注册/登录/注销**时，我们不会有任何名称空间冲突。
接下来，我们定义两种**动作类型**，这将帮助我们的 reducers 决定如何处理我们的动作。使用字符串常量并导出它们可以确保我们的 reducers 不会让一个打字错误悄悄溜走。此外，当处理较大的应用程序时，所有这些都可以抽象到一个 **actionTypes.js** 文件中。之后，我们定义了两个**动作**。动作是 JavaScript 对象，为我们的 Redux 存储提供数据负载(在本例中来自我们的后端)。***

***稍后，我们将使用中间件，它根据对象是*函数*还是 *POJO* (普通的 ol' JavaScript 对象)这一前提来决定是进行更多的 API 调用还是进入下一步——reducers。
为此，我们将定义**动作创建者，**我们的组件将**分派**哪些动作创建者从后端收集资源。
在这些请求发出后，这些**动作创建者**将会**分派**我们的**动作**。然后**reducer**将过滤数据，并在我们的 Redux 存储中分配它的位置。***

***好吧，那么…让我们来定义这些动作创建者:***

```
**--- frontend/actions/session.js ---* **...
export const login = user => async dispatch => {
  const response = await apiUtil.login(user);
  const data = await response.json();****if (response.ok) {
    return dispatch(receiveCurrentUser(data));
  }
  // todo: handle errors
};****export const signup = user => async dispatch => {
  const response = await apiUtil.signup(user);
  const data = await response.json();

  if (response.ok) {
    return dispatch(receiveCurrentUser(data));
  }
  // todo: handle errors
};****export const logout = () => async dispatch => {
  const response = await apiUtil.logout();
  const data = await response.json();****if (response.ok) {
    return dispatch(logoutCurrentUser());
  }
  // todo: handle errors
};***
```

***这三个函数遵循相同的逻辑:返回一个异步函数，该函数将 **dispatch** 作为参数，发出一个 HTTP 请求，并从响应中提取数据。
如果响应没问题(200 级)，调度一个**动作**我们上面定义的有效载荷。如果响应不正常，那么我们处理错误。这样想象会有所帮助:***

```
*export const login = function(user) {
  return async function(dispatch) {
    ...
  };
};*
```

***但是因为我们是 JavaScript 专家，我们将保持它的 ESNext。
让我们迅速找到处理这些错误的方法。在我们的 **actions/** 文件夹中创建一个 **error.js** :***

```
**--- frontend/actions/error.js ---* **export const RECEIVE_ERRORS = 'RECEIVE_ERRORS';
export const CLEAR_ERRORS = 'CLEAR_ERRORS';****export const receiveErrors = ({ message }) => ({
  type: RECEIVE_ERRORS,
  message
});****export const clearErrors = () => ({
  type: CLEAR_ERRORS
});***
```

***同样的模式。现在，回到我们的**会话。js** :***

```
**--- frontend/actions/session.js ---* import * as apiUtil from ‘../util/session’; **import { receiveErrors } from "./error";
...** if (response.ok) {
    return dispatch(receiveCurrentUser(data));
  } **return dispatch(receiveErrors(data));**
};
...
if (response.ok) {
    return dispatch(receiveCurrentUser(data));
  } **return dispatch(receiveErrors(data));**
};
...
if (response.ok) {
    return dispatch(logoutCurrentUser());
  } **return dispatch(receiveErrors(data));** };*
```

***我们的行动结束了！💪***

***让我们开始我们的减速器！在 **reducers/root.js** 中:***

```
**--- frontend/reducers/root.js ---* **import { combineReducers } from 'redux';
import errors from './errors/errors';
import session from './session/session';****export default combineReducers({
  session,
  errors
});***
```

***我们还没有创建我们的**错误**和**会话**减少器，但是**组合减少器**做了我们在这里所期望的事情。它结合了我们的减速器，将它们放在我们前端状态的*相同深度水平*。这些可以嵌套得更深，尽管嵌套太深的状态会产生代码味道。你经常会在这里看到一个**实体**缩减器，实体缩减器将会合并更多的缩减器，比如用户、帖子等等…***

***现在让我们创建这两个减速器。让我们在 **reducers/** 中创建另一个 **session.js** 文件:***

```
**--- frontend/reducers/session/session.js ---* **import {
  RECEIVE_CURRENT_USER,
  LOGOUT_CURRENT_USER
} from "../../actions/session";****const _nullSession = { userId: null, username: null }**
**export default (state = _nullSession, { type, user }) => {
  Object.freeze(state);
  switch (type) {
    case RECEIVE_CURRENT_USER:
      return user;
    case LOGOUT_CURRENT_USER:
      return _nullSession;
    default:
      return state;
  }
};***
```

***我们稍微讨论一下这个。**缩减器**是一个接受两个参数的函数——当前状态和动作。关于减速器我想指出三点:
1。它们进入*的状态并不是整个状态*。例如，*这个* **会话**减速器*不能*影响我们**错误**的状态形状或状态的任何其他部分。只有**会话**。
2。*每一个动作都击中每一个减速器*。这就是为什么所有的动作都有一个**类型**属性，以及为什么 reducers 使用 switch 语句。
3。**千万不要突变状态。** *永不突变的状态。*让 Redux 做它的事。***

***这样一来，这里的代码就不会太差了。首先，我们导入动作类型。然后我们定义一个 **_nullSession** ，这样我们就可以一直调用**。用户名**或**。如果没有用户登录，就不会出错。
接下来，我们给传入状态一个默认值——我们的空会话——并析构 action 对象。我们还冻结了状态，这防止了任何意外的突变，并让其他人知道他们正在使用我们的代码，“后退！”
开个玩笑。但它清楚地表明。
最后，根据动作**类型**，我们返回的代码可能会也可能不会改变我们的 Redux 存储。*****

***我们的误差减少器将非常相似。**reducers/errors/errors . js**:***

```
**--- frontend/reducers/errors/errors.js ---* **import { RECEIVE_CURRENT_USER } from "../../actions/session";
import { CLEAR_ERRORS, RECEIVE_ERRORS } from "../../actions/error";****export default (state = "", { message, type }) => {
  Object.freeze(state);
  switch (type) {
    case RECEIVE_ERRORS:
      return message;
    case RECEIVE_CURRENT_USER:
    case CLEAR_ERRORS:
      return "";
    default:
      return state;
  }
};***
```

***这里唯一的区别是，当我们接收当前用户时，我们也会清除错误。这就是你经常会看到不同的动作触发状态的不同部分。***

***唉，我们快满足 Redux 了…
让我们创建我们的 **store/store.js** :***

```
**--- frontend/store/store.js ---* **import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import reducer from "../reducers/root";****export default preloadedState => (
  createStore(
    reducer,
    preloadedState,
    applyMiddleware(thunk)
  )
);***
```

***这是将所有东西结合在一起的东西。 **createStore** ，嗯，创建我们的 Redux store，它包含我们精心制作的前端状态。Redux Thunk 是帮助我们简化异步操作的中间件。我们导出一个函数，它接受一个预加载状态作为参数，并返回我们的存储。这个**预加载状态**是我们保持用户登录的方式。***

***现在让我们通过前往我们的 **index.js** 将 Redux 插入我们的应用程序:***

```
**--- frontend/src/index.js ---* import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
**import configureStore from './store/store';
import { Provider } from "react-redux";****let preloadedState = {};
const store = configureStore(preloadedState);**ReactDOM.render(
  **<Provider *store*={store}>**
    <App />
 **</Provider>,**
  document.getElementById("root")
);
***// FOR TESTING, remove before production* window.getState = store.getState;***
```

***我们导入我们在 **store.js** 和 **Provider** 中创建的函数。我们让预加载状态现在等于一个空对象，并定义我们的**存储**。通过将我们的**应用**包装在**提供者**中，并将**商店**作为道具传递，我们可以在应用的任何地方访问我们的 Redux 商店。 **Props** 可以被认为是提供给组件的参数，因为它们是通过 **prop** 对象访问的。
最后，我们将 **getState** 方法保存到窗口中，这样我们可以在控制台中看到我们的状态，然而对于更大的应用程序，我推荐 [redux-logger](https://github.com/LogRocket/redux-logger) 。***

***让我们前往 [http://localhost:3000/](http://localhost:3000/) 以确保一切仍然正常，并尝试我们的 **getState** 函数。***

***![](img/7bc437e8cbdc8c4918e8ed41568f92f1.png)***

***使用谷歌 Chrome 开发者工具，
Windows: `CTRL-SHIFT-J`或`F12`
Mac: `⌥-⌘-J`，
我们可以看到我们的状态！***

***Redux 现在准备好摇滚了！第 6 步到此结束！😅***

# ***第七步:组件***

*****这是最后一步。我们的旅程即将结束。我们所要做的就是创建一些组件，为其中一些组件添加保护，并实现一种方法，让用户在刷新页面时保持登录。
让我们从组件开始——总共有五个组件，包括 **App.js** 。
创建一个**组件/Welcome.js** :*****

```
**--- frontend/src/components/Welcome.js ---* **import React from 'react';
import { Link } from 'react-router-dom';****export default () => (
  <>
    <h1>Welcome!</h1>
    <Link to='/login'>Login</Link>
    <Link to='/signup'>Signup</Link>
    <Link to='/dashboard'>Dashboard</Link>
  </>
);***
```

***一如既往，我们从进口开始。 **Link** 除了其路径是绝对的之外，其行为就像一个锚标签。除此之外，我们使用另一个片段，添加一条欢迎消息和三个链接。很简单。
接下来是 **components/Signup.js** 。我把它分成三份:***

```
**--- frontend/src/components/Signup.js ---* **import React from "react";
import { connect } from "react-redux";
import { Link } from "react-router-dom";
import { signup } from "../actions/session";****const mapStateToProps = ({ errors }) => ({
  errors
});****const mapDispatchToProps = dispatch => ({
  signup: user => dispatch(signup(user))**
**});
.**..*
```

***我们进口中唯一的新来者是**通。这个函数将允许我们的组件访问我们的 Redux 存储。我们接着定义两个函数:
**mapStateToProps**；这将**状态**作为一个参数，并让我们的组件访问 Redux 存储的特定部分(在本例中是我们的错误)。
**mapdispatctoprops**；这将**分派**作为一个参数，并赋予我们的组件分派我们之前定义的**动作**的能力。
我们继续:*****

```
**--- frontend/src/components/Signup.js ---* ...
**const Signup = ({ errors, signup }) => {
  const handleSubmit = e => {
    e.preventDefault();
    const user = {
      username: e.target[0].value,
      email: e.target[1].value,
      password: e.target[2].value
    };
    signup(user);
  };** **return (
    <>
      <h1>Signup</h1>
      <p>{errors}</p>
      <form onSubmit={handleSubmit}>
        <label>
          Username:
          <input type="text" name="username" />
        </label>
        <label>
          Email:
          <input type="email" name="email" />
        </label>
        <label>
          Password:
          <input type="password" name="password" />
        </label>
        <input type="submit" value="Submit" />
      </form>
      <Link to="/login">Login</Link>
    </>
    );
  };**
...*
```

***这看起来有点可怕，但这里并没有发生太多的事情。这是我们在这个文件中实际的**组件**。首先，我们接受**道具**作为参数，并析构我们需要的东西。这些来自于**mapStateToProps**/**mapDispatchToProps。**
接下来，我们创建一个提交处理函数，其中“**e**”**是事件对象。我们阻止默认行为(刷新页面)，通过从表单中提取输入值来创建一个**用户**对象，并调用我们的**登录**函数。
通常，你会想要一个[受控组件](https://reactjs.org/docs/forms.html#controlled-components)在这里，但是我们保持事情简单。
最后，我们归还我们的 JSX。在这种情况下，它只是一个标题、我们的错误*、一个表单和另一个链接。*这些是来自我们 Redux 商店的错误。如果没有，React 不会渲染任何东西。花括号是我们在组件返回语句中添加 JavaScript 代码的方式。
注意到`<form onSubmit={handleSubmit}>`？这将在我们的表单上放置一个 **onsubmit** 事件，传递我们在上面定义的函数作为回调。
最后一点:*****

```
****--- frontend/src/components/Signup.js ---* ...
**export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Signup);*****
```

*****这就是连接是如何发生的。这里我们将我们的**注册**组件“连接”到 Redux 商店。*****

*****现在，让我们创建我们的**组件/Login.js** 文件:*****

```
****--- frontend/src/components/Login.js ---* **import React from "react";
import { connect } from "react-redux";
import { Link } from "react-router-dom";
import { login } from "../actions/session";****const mapStateToProps = ({ errors }) => ({
  errors
});****const mapDispatchToProps = dispatch => ({
  login: user => dispatch(login(user))
});****const Login = ({ errors, login }) => {
  const handleSubmit = e => {
    e.preventDefault();
    const user = {
      email: e.target[0].value,
      password: e.target[1].value,
    };
    login(user);
  }** **return (
    <>
      <h1>Login</h1>
      <p>{errors}</p>
      <form onSubmit={handleSubmit}>
        <label>
          Email:
          <input type="email" name="email" />
        </label>
        <label>
          Password:
          <input type="password" name="password" />
        </label>
        <input type="submit" value="Submit" />
      </form>
      <Link to="/signup">Signup</Link>
    </>
  );
};****export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Login);*****
```

*****这里唯一的不同是我们导入了**登录**而不是注册，并且少了一个表单字段。*****

*****干得好！
让我们继续我们的**组件/Dashboard.js** :*****

```
****--- frontend/src/components/Dashboard.js ---* **import React from "react";
import { connect } from "react-redux";
import { logout } from "../actions/session";****const mapStateToProps = ({ session }) => ({
  session
});****const mapDispatchToProps = dispatch => ({
  logout: () => dispatch(logout())
});****const Dashboard = ({ logout, session }) => (
  <>
    <h1>Hi {session.username}</h1>
    <p>You are now logged in!</p>
    <button onClick={logout}>Logout</button>
  </>
);****export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Dashboard);*****
```

*****注意到模式了吗？我们做同样的事情映射状态和分派到道具和连接我们的组件在底部。我们使用会话信息通过用户名问候用户，并添加一个按钮让用户注销。*****

*****最后，我们更新我们的 **App.js** :*****

```
****--- frontend/src/components/App.js ---* import React from "react"; **import { Route } from "react-router-dom";
import Welcome from "./Welcome";
import Login from "./Login";
import Signup from "./Signup";
import Dashboard from "./Dashboard";**export default () => (
  <> **<Route exact path="/" component={Welcome} />
    <Route path="/login" component={Login} />
    <Route path="/signup" component={Signup} />
    <Route path="/dashboard" component={Dashboard} />**  </>
);***
```

*****🤔嗯，我们进口我们的组件，但这是什么**路线**的事情？当 URL 与**路径**匹配时，**路由**组件接受一个**组件**属性。“ **exact** ”标志指定路径必须完全是“/”。如果没有这个，我们的 **Welcome** 组件将在每个页面上呈现，因为所有路径都以“/”开头。(这可能对**导航**组件有用！)*****

*****然而，为了使用**路径**，我们必须将**应用**包装在其中，就像我们对**提供者**所做的那样:*****

```
****--- frontend/src/index.js ---
...* **import { BrowserRouter } from "react-router-dom";**
*...*
ReactDOM.render(
  <Provider *store*={store}>
    **<BrowserRouter>**
      <App />
    **</BrowserRouter>**
</Provider>,
  document.getElementById("root")
);
***...******
```

*****![](img/31d7606d1cc0ae2eb07b2b8036d1eeee.png)*****

*****在浏览器中测试，我们可以看到我们的路线和网页的工作。让我们测试一下我们的错误:*****

*****![](img/243d0b3f7a1a50547e30213d33d3756b.png)*****

*****不错！好吧，如果我试着注册呢？似乎什么都没发生…还是真的发生了？*****

*****![](img/82dfa6a2bb0191599970b9097f2a8c96.png)*****

*****如果我们在 Chrome 控制台中再次查看我们的状态，我们可以看到它确实让用户登录了。如果我们看看我们的饼干:*****

*****![](img/692fa2d30d4596a7fdae5bec720cbf7e.png)*****

*****我使用的是 [Edit This Cookie](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en) Chrome 扩展，但这也可以使用 Chrome 开发者工具查看。
我们看到我们的饼干在那里！*****

*****如果我导航到仪表板(通过手动键入 URL)并注销，cookie 就会消失。我们的后端工作！
但是我们的网站有点问题……
让我们来解决这个问题。*****

*****我们的下一步是创建**保护路由**和**授权路由**。受保护的路由将不允许已注销的用户访问它们，并将它们重定向到登录页面。授权路由将不允许已登录的用户访问登录/注册页面。如果有，我们会将它们重定向到仪表板。从概念上讲，这听起来很简单，但是这里的代码有点复杂。别担心，我们已经到了最后关头，现在我们不能放弃！让我们开始吧。在您的 **util/** 文件夹中创建一个 **route.js** 文件:*****

```
****--- frontend/src/util/route.js ---* **import React from "react";
import { connect } from "react-redux";
import { Redirect, Route, withRouter } from "react-router-dom";****const mapStateToProps = ({ session: { userId} }) => ({
  loggedIn: Boolean(userId)
});
...*****
```

*****还没有太疯狂的事。我们的导入中的两个新成员是**重定向**和**带路由器**。第一个简单地重定向到我们选择的 URL。第二个就像 **connect** 一样使用，除了它为我们的组件提供了与路线/参数/位置等相关的有用属性。我们不会使用这些属性中的任何一个，但是我在这里添加了 **withRouter** ,因为理想情况下，我们希望在我们的应用程序中使用该功能。接下来，我们去拜访我们的好朋友。我们深度地析构了我们的状态对象，并提供了一个简单地回答这个问题的道具，“我们登录了吗？”让我们继续:*****

```
****--- frontend/src/util/route.js ---
...* **const Auth = ({ loggedIn, path, component: Component }) => (
  <Route
    path={path}
    render={props => (
      loggedIn ?
      <Redirect to='/dashboard' /> :
      <Component {...props} />
    )}
  />
);****const Protected = ({ loggedIn, path, component: Component }) => (
  <Route
    path={path}
    render={props => (
      loggedIn ?
      <Component {...props} /> :
      <Redirect to='/login' />
    )}
  />
);**
*...****
```

*****从语法上来说，我已经尽可能地让它可读性更好。这两个功能几乎相同——唯一的区别是**重定向**。
我们来分解一下。在高层次上，这些只是隐式返回一个 **Route** 组件的组件，该组件可能呈现另一个组件或重定向。我们从破坏我们的道具开始。需要`component: Component`以便 React 知道这是作为道具出现的整个组件。
接下来，我们通过使用我们提供的路径建立我们的**路线**。
然后，我们不使用`component={SomeComponent}`，而是使用 **render** prop，它允许我们提供一个匿名函数，该函数使用三元组来显示我们的组件或重定向。
我们还利用[扩展操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)将我们的道具传递给渲染组件。还有一步:*****

```
****--- frontend/src/util/route.js ---
...* **export const AuthRoute = withRouter(
  connect(mapStateToProps)(Auth)
);****export const ProtectedRoute = withRouter(
  connect(mapStateToProps)(Protected)
);*****
```

*****我们用路由器和**连接**这些组件来包装**中的所有东西。
现在回到我们的 **App.js** :*******

```
****--- frontend/src/util/routes.js ---
...* **import { AuthRoute, ProtectedRoute } from "../util/route";**export default () => (
  <>
    <Route exact path="/" component={Welcome} /> **<AuthRoute path="/login" component={Login} />
    <AuthRoute path="/signup" component={Signup} />
    <ProtectedRoute path="/dashboard" component={Dashboard} />**
  </>
);***
```

***注意到它们有多容易使用了吗？我们可以像使用普通路线一样使用它们！此外，现在我们总是有这些在我们的处置。***

***现在去 [http://localhost:3000/](http://localhost:3000/) 测试一下。如果一切顺利，你应该不能访问仪表板。
甜！还有一件事要做——而且很简单！***

***在我们的 **util/session.js** 中，让我们再添加一个函数:***

```
****--- frontend/src/util/session.js ---
...* **export const checkLoggedIn = async preloadedState => {
  const response = await fetch('/api/session');
  const { user } = await response.json();
  let preloadedState = {};**
  **if (user) {
    preloadedState = {
      session: user
    };
  }
  return preloadedState;
};*****
```

***这里，我们点击最后一个 API 端点来检查会话是否存在。
现在在我们的 **src/index.js** :***

```
****--- frontend/src/index.js ---
...* **import { checkLoggedIn } from "./util/session";****const renderApp = preloadedState => {
  const store = configureStore(preloadedState);
  ReactDOM.render(
    <Provider store={store}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </Provider>,
    document.getElementById("root")
  );
};****(async () => renderApp(*await* checkLoggedIn()))();*****
```

***首先，我们将与渲染我们的**应用**相关的一切封装到一个函数中。然后我们使用另一个生命，就像我们在我们的服务器中所做的一样。使用 **checkLoggedIn** 的返回值，我们的 **renderApp** 函数拥有启动我们的应用程序所需的一切。
不过说够了，我们来测试一下吧！***

***![](img/788ccfaf7235d3dd7713c1260dbcbf1d.png)***

***登录后，我们被自动重定向到仪表板(因为登录/注册页面是 **AuthRoute** s)。***

***我们可以刷新并保持登录状态！***

***注销后，我们会被自动重定向到登录页面(因为仪表板页面是一个**受保护的路由**)。我们的控制台没有错误、红灯或故障，一切都运行顺利。***

# ***🏆任务已完成🏆***

***我知道这篇文章很长，而且这是我第一次在网上写东西，但是我真的很享受这种体验。当然，最重要的是，我希望你喜欢这个！***

***下次见！***

***[在 GitHub 上查看代码库。](https://github.com/christiancashiola/SessionAuth)
[更多关于我。](http://christiancashiola.me/)***