---
title: 简单搭建React应用
description: 搭建一个简单的React全家桶应用，串联工具/技术知识，方便日后的持续学习使用。
date: 2022-05-03 00:00
---

搭建一个简单的React全家桶应用，串联工具/技术知识，方便日后的持续学习使用。

本篇为第一篇，完成以下内容：
- 基于 webpack5 搭建了一个支持 ES6+ 的基础 React 应用
- 接入了 css-modules/less/postcss 样式处理
- 引入了UI组件库 shineout，添加了 ESlint 规则
- **不包含**全局状态管理和路由切换

本篇源码 nostore-v1.0.0：https://github.com/zzyxka/react-zzy-prj/releases/tag/nostore

<!-- more -->

# 基础的 react 应用

目标：
- [x] webpack5
- [x] 支持 ES6+
- [x] 支持 React(JSX)
- [x] 开发/生产环境分离

```bash
mkdir react-zzy-prj
cd react-zzy-prj
npm init -y

npm install webpack webpack-cli --save-dev
touch webpack.config.dev.js
touch webpack.config.prod.js
```

```json
// package.json
{
  // ...
   "scripts": {
    "start": "webpack --config webpack.config.dev.js",
    "build": "webpack --config webpack.config.prod.js"
  },
  // ...
}
```

## HtmlWebpackPlugin

前置工作：自动生成 HTML，并引入打包后的入口 js 文件。

```bash
npm i -D html-webpack-plugin
touch index.js
```

```js
// index.js
const h1 = document.createElement('h1');
h1.innerText = 'HELLO';
document.body.appendChild(h1);
```

```js
// webpack.config.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './index.js',
  plugins: [new HtmlWebpackPlugin({ title: 'HELLO' })],
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

```bash
npm run build # 执行后生成 dist 目录，访问目录下的 index.html 展示出 HELLO
```

## ES6 支持

使用 babel + [babel-loader](https://github.com/babel/babel-loader) + [@babel/preset-env](https://babel.docschina.org/docs/en/babel-preset-env) 编译 ES6+ 语法。

```js
// index.js
const h1 = document.createElement('h1');
h1.innerText = 'HELLO';
document.body.appendChild(h1);

const print = () => {
  const a = {};
  console.log(a?.b);
};

window.onload = () => {
  print();
};
```

```js
// 执行 npm run build
// bundle.js
(() => {
  const e = document.createElement('h1');
  (e.innerText = 'HELLO'),
    document.body.appendChild(e),
    (window.onload = () => {
      console.log({}?.b);
    });
})();
```

index.js 增加了一些 ES6+ 的箭头函数和可选链`?.`语法，可以看到打印结果中，也是 ES6+ 语法。

```bash
npm install -D babel-loader @babel/core @babel/preset-env
```

```js
// webpack.config.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './index.js',
  plugins: [new HtmlWebpackPlugin({ title: 'HELLO' })],
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: { // 新增 babel-loader 处理 js 文件
    rules: [
      {
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [['@babel/preset-env', { targets: 'defaults' }]],
          },
        },
      },
    ],
  },
};
```

执行 `npm run build` 后，再次打包出的 bundle.js 已经转换为兼容性语法。

```js
// bundle.js
(() => { // 最外层的箭头函数，见下文
  var n = document.createElement('h1');
  (n.innerText = 'HELLO'),
    document.body.appendChild(n),
    (window.onload = function () {
      var n;
      (n = {}), console.log(null == n ? void 0 : n.b);
    });
})();
```

奇怪的是，最外层的 IIFE 是个箭头函数，仔细看 babel-loader 的文档可以找到解决方案。

![](https://raw.githubusercontent.com/zzyxka/images/main/20220510172436.png)

## React/jsx 支持

与支持 ES6 类似，使用 [@babel/preset-react](https://babel.docschina.org/docs/en/babel-preset-react) 转换 React 的 jsx 语法。

```jsx
// App.jsx
import React from 'react';
import ReactDOM from 'react-dom';

const App = () => (
  <h1>Hello React !</h1>
);

export default () => {
  ReactDOM.render(<App />, document.body);
};
```

```js
// index.js
import renderApp from './App.jsx';

renderApp();
```

```bash
npm install -D react react-dom
npm install --save-dev @babel/preset-react
```

```js
// webpack.config.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './index.js',
  plugins: [new HtmlWebpackPlugin({ title: 'HELLO' })],
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    environment: { arrowFunction: false }, // 解决最外层 IIFE 箭头函数
  },
  module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [['@babel/preset-env', { targets: 'defaults' }]],
          },
        },
      },
      {
        test: /\.jsx$/, // 新增处理 jsx 文件
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-react',
              ['@babel/preset-env', { targets: 'defaults' }],
            ],
          },
        },
      },
    ],
  },
};

```

执行 `npm run build` 打开 index.html 会看到 Hello React!

> 注：babel 可以创建一个独立的配置文件，替代在 webpack 中配置所有内容。这一部分，以及 polyfill 方案，打算另起一篇笔记来写。

## 开发环境打包


```bash
npm install --save-dev webpack-dev-server
```

CV 一份 webpack.config.prod.js 到 webpack.config.dev.js，然后做如下调整。

> webpack.config.prod.js 下也增加了 mode 参数，比较简单。

```js
// webpack.config.dev.jsx
// webpack.config.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // ...
  mode: 'development', // 配置打包模式
  devtool: 'inline-source-map', // source-map
  devServer: {
    // devServer
    static: './dist',
  },
  // ...
};
```

```json
{
  // ...
  "scripts": {
    "start": "webpack serve --open --config webpack.config.dev.js",
    // ...
  },
}
```


# 样式处理

目标：
- [x] css & css module
- [x] 支持 less & 组件库 shineout
- [x] 支持 postcss(浏览器前缀)

## css

```bash
npm install --save-dev style-loader css-loader
```

```js
// webpack.config.xxx.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
        exclude: /node_modules/,
      },
      // ...
    ],
  },
};
```

```css
/* App.css */
h1 {
  color: yellowgreen;
}
```

```jsx
// App.jsx
import './App.css';
// ...
```



## css modules
```js
// webpack.config.xxx.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
            },
          },
        ],
        exclude: /node_modules/,
      },
      // ...
    ],
  },
};
```
## less & 组件库 shineout

```bash
npm install less less-loader --save-dev
npm install shineout
```

```js
// webpack.config.xxx.js
const webpack = require('webpack');

module.exports = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env': {
        CSS_MODULE: true,
      },
    }),
  ],
  module: {
    rules: [
      {
        test: /\.less$/i,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
            },
          },
          'less-loader',
        ],
      },
      // ...
    ],
  },
};
```


## postcss

```bash
npm i postcss-loader astroturf
npm i autoprefixer
```

postcss 不再赘述。整理项目结构，修改相关 loader，备份分支 nostore，后续改分支不加入任何状态管理内容，用于某些场景使用。

https://github.com/zzyxka/react-zzy-prj/releases/tag/nostore


# 添加 ESlint

```bash
npm install eslint --save-dev
./node_modules/.bin/eslint --init # 按命令行指引选所需内容
```

使用 `.eslintignore` 来忽略不需要进行规则检查的文件/目录。

整理项目结构，将非配置类的代码移动到 src 目录下，增加别名、后缀省略等相关人性化配置：

```js
exports.baseConf = {
  resolve: {
    modules: ['node_modules', 'web_modules'],
    extensions: ['.js', '.jsx', '.css', '.json'],
    alias: {
      src: path.resolve(__dirname, './src/'),
    },
  },
};
```