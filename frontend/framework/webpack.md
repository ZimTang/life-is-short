# webpack 入门

## 什么是 webpack

先看看 `webpack` 官方文档的解释：

> **webpack** is a *static module bundler* for modern JavaScript applications.

翻译成中文的意思就是：`webpack` 是一个为现代 `JavaScript ` 应用的静态打包工具。

这里面有几个重要的名词：`static` 、`module` 、`bundler` 、`modern`

* `static`：表示静态资源，最终将它们部署到服务器上
* `module`：表示默认支持模块化开发，如：`CommonJS` 、`ES Module` 等
* `bundler`：表示这是一个打包工具
* `modern`：表示现代前端的发展存在很多问题，正是因为有这些问题，才催生了 `webpack` 的出现

### webpack 出现前存在哪些问题

如今的前端开发已经变得越来越复杂，需要考虑很多东西，如：模块化开发、使用高级特性来提高开发效率、对代码进行压缩、实施监听文件变化提高开发效率。

这个时候 `webpack` 出现了。

如今的前端三大框架 `Vue` 、`React` 、`Angular` 的创建使用都是基于脚手架（CLI），但其实，CLI都是 **基于webpack** 来帮助我们支持上述功能。

## webpack 的安装

`webpack` 的安装分为两个：`webpack`、`webpack-cli`

**它们之间的关系是这样的：**

* 在执行webpack命令的时候，会执行 `node_modules` 下的 `.bin` 目录下的 `webpack` 
* `webpack` 在执行的时候，依赖 `webpack-cli` ，如果没有安装则会报错
* 而 `webpack-cli` 中的代码在执行的时候，才是真正利用了 `webpack` 的编译和打包

## webpack 的默认打包

在项目目录下，我们可以直接在终端中，执行 `webpack` 命令。

此时会生成一个 `dist` 文件夹，里面会存放 `main.js` 文件。这就是 `webpack` 的出口文件。

### 入口与出口

当我们运行 `webpack` 命令的时候，它会查找当前目录下的 `src/index.js` 文件作为入口文件。

并且会将打包生成 `dist/main.js` 文件，作为出口文件。

在这个 `main.js` 文件中，它默认会对代码进行压缩和丑化。

## 局部使用 webpack

可以通过 `npx webpack` 命令执行，它会执行 `node_modules` 下的 `.bin` 目录下的 `webpack`。

也可以通过在 `package.json` 文件中，创建 `scripts` 脚本，执行打包。

```json
  "scripts": {
    "build": "webpack"
  },
```

## webpack 的配置文件

webpack需要打包的项目是非常复杂的，并且我们需要一系列的配置来满足要求。

我们可以在根目录下创建一个 `webpack.config.js` 文件，来作为 `webpack` 的配置文件： 

```js
const path = require('path')

module.exports = {
  // 配置入口文件
  entry: './src/index.js',
  // 配置出口文件
  output: {
    // 绝对路径   dirname获取当前文件所在的路径
    path: path.resolve(__dirname, './dist'),
    // 打包的文件名
    filename: 'main.js',
  },
}
```

## webpack 的依赖

`webpack` 在处理应用程序时，它会根据命令或者配置文件找到入口文件。

从入口开始，会生成一个 依赖关系图，这个依赖关系图会包含应用程序中所需的所有模块（比如.js文件、css文件、图片、字体等）。

然后遍历图结构，打包一个个模块（根据文件的不同使用不同的 `loader` 来解析）。

**`webpack` 的官方图：**

![image-20220102112311888](https://codertzm.oss-cn-chengdu.aliyuncs.com/image-20220102112311888.png)

## 什么是loader

`loader` 可以用于对模块的源代码进行转换，

我们可以将 `css` 文件也看成是一个模块，如果我们现在在一个 `js` 文件中，写如下的代码用于引入 `css` 文件：

```js
import './css/style.css'
```

此时我们进行打包的时候，是会报错的。

因为在加载这个模块的时候，`webpack` 是不知道如何对其进行加载的。我们需要使用对应的 `loader` 来完成这个加载的功能。

## loader的配置

如果需要加载 `css` 文件模块，需要使用 `css-loader`。

在使用 `npm` 安装完成后，我们需要使用它，有以下两种方案：

1. 内联方式（不推荐）

   在引入的文件前，加上使用的 `loader` ，并使用 ! 分割

   ```js
   import 'css-loader!./css/style.css'
   ```

2. 配置方式**（推荐）**

   `module.rules`  中允许配置多个 `loader` 

   `rules` 对应的值是一个数组：数组中是一个个的对象，对象中可以设置多个属性：

   * `test` 属性：用于对 resource（资源）进行匹配的，通常会设置成正则表达式
   * `use` 属性：对应的是一个数组，数组中的每个元素是对象，对象的属性包括
     * `loader`：必须有一个 loader属性，对应的值是一个字符串
     *  `options`：可选的属性，值是一个字符串或者对象，值会被传入到loader中
   * `loader`属性： `Rule.use: [ { loader } ] ` 的简写

   ```js
   module：{
     rules:[
       {
         {
           // 正则表达式
           test: /\.css$/,
           // 第一种写法 语法糖
           // loader: 'css-loader',
   
           // 第二种写法 完整写法
           use: [
             {
               loader: 'css-loader',
             },
           ],
         },
       }
     ]
   }
   ```

配置完成，重新打包后，我们会发现样式并没有生效，这是因为 `css-loader` 只是**负责将.css文件进行解析**，并不会将解析之后的css插入到页面中。

如果我们需要完成插入`style` 的操作，需要使用 `style-loader` 。

## loader 的配置顺序

`loader` 的配置是需要考虑顺序的，比如 `style-loader` 的使用是需要建立在有 `css-loader`  的基础上的。

因为loader的执行顺序是从后到前，所以我们需要将 `style-loader` 写到 `css-loader` 的前面；

```js
use: [
  'style-loader',
	'css-loader'
],
```

## less 的处理

在实际开发中，我们可能会使用 `less`、`sass` 这样的预处理器，（这里以 `less` 为例）因此我们需要一个将`less` 转换为 `css` 文件的编译工具。

我们可以使用 `less` 工具完成编译：

安装：

`npm install less -D`

执行如下命令，进行编译转换：

`npx lessc 'less文件' 'css文件'`

### less-loader

但是，在实际开发中，我们需要编写很多的 `less` 文件，如果全部进行手动编译，是不现实的。因此，我们需要 `less-loader` 来帮助我们完成自动编译。

```js
{
  test:/\.less$/,
  use: [
    'less-loader',
    'style-loader',
    'css-loader'
  ],
}
```

## 静态资源的打包

如果我们需要在项目中使用静态资源（如：图片，字体等），也需要对它进行打包的处理。

在 `webpack5` 之前，加载这些资源我们需要使用一些 `loader`，比如 `raw-loader` 、`url-loader`、`file-loader` 。

从 `webpack5` 开始，我们可以直接使用资源模块（asset module type），来代替上面的这些 `loader` 。

**资源模块类型(asset module type)**，通过添加 4 种新的模块类型，来替换所有这些 `loader`： 

* **asset/resource** 发送一个单独的文件并导出 URL。之前通过使用 `file-loader` 实现；

* **asset/inline** 导出一个资源的 data URI。之前通过使用 `url-loader` 实现； 

* **asset/source** 导出资源的源代码。之前通过使用 `raw-loader` 实现；

* **asset** 在导出一个 data URI 和发送一个单独的文件之间自动选择。之前通过使用 url-loader，并且配置资源体积限制实现；

```json
{
  test: /\.(jpe?g|png|gif|svg)$/,
  type: "asset",
  generator: {
    // 设置生成的文件名和在哪个文件夹下面
    // 其中img/为文件夹名称 
    // [name] 原文件名
    // [hash:xxx] 设置的hash值
    // [ext] 文件扩展名
    filename: "img/[name]_[hash:6][ext]"
  },
  parser: {
    dataUrlCondition: {
      // 设置需要转换base64的文件大小最大值
      maxSize: 100 * 1024
    }
  }
},
```

## Plugin

`webpack` 的另一个核心概念是 `Plugin` 。

`Loader` 仅仅是用于特定的模块类型进行转换。

而 `Plugin` 可以用于执行更加广泛的任务，如打包优化，资源管理等。

### CleanWebpackPlugin

此插件可以帮助我们打包的时候，自动删除原打包文件夹。

#### 使用

首先需要使用 `npm` 安装：

`npm install clean-webpack-plugin -D`

接着引入，并在 `webpack.config.js` 文件中使用：

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
plugins:[
  new CleanWebpackPlugin()
]
```

### **HtmlWebpackPlugin**

此插件是对 `index.html` 文件进行打包处理。

```js
new HtmlWebpackPlugin()
```

#### 自定义HTML模板

如果我们想要在自己的模块中加入一些特别的内容，这时候我们需要一个自己的模板

```js
new HtmlWebpackPlugin({
  template: "./public/index.html",
  title: "html文件的标题名"
})
```

### DefinePlugin

此插件可以在编译 `template` 中的常量，对常量进行解析。

```js
new DefinePlugin({
  // 配置全局常量 BASE_URL
  BASE_URL: "'./'"
}),
```

### **CopyWebpackPlugin**

在打包过程中，如果我们想要将一个目录下的文件被复制到 `dist` 文件夹中，我们可以使用 `CopyWebpackPlugin` 来完成。

```js
new CopyWebpackPlugin({
  // 复制的规则在patterns中设置
  patterns: [
    {
      // from是原目录
      from: "public",
      // to是复制到的目录 默认值是打包的目录
      to: "./",
      // globOptions用于设置一些额外的选项，其中可以编写需要忽略的文件：
      globOptions: {
        ignore: [
          "**/index.html"
        ]
      }
    }
  ]
})
```

## mode配置

mode配置选项，可以告知 `webpack` 使用响应模式的内置优化。

默认值是  `production` ，可选值有 `none` | `development` | `production`

其中在开发阶段，我们一般设置为 `development` ，生产阶段设置为 `production` 。

> 注意：mode的配置一旦配置，webpack其实帮我们配置了其他很多东西，如果出现与你的配置重复，则会选择合并，不会覆盖。
