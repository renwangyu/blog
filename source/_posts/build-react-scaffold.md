---
title: 2021了，来套react开发环境：搭建篇
date: 2021-07-14 14:13:32
tags:
- javascript
categories:
- 前端
keywords:
- react
- 脚手架
top_img: /img/post/computer.png
cover: /img/post/computer.png
---

### 前言
在写了2年的vue后，终于又能愉快的用react了~2021了，得益于那些伟大的项目，前端的环境搭建也早已不像从前那样风格迥异，艰难异常。我们就来已创建一个lib工程为例，看看如何用`create-react-app`(cra)配合一系列其他工具，快速搭建一个用于开发组件lib库的react开发环境。

### 我们的目标
+ 用cra创建，但不用`eject`来反编译生成构建源码。
+ 使用`react-app-rewired`来扩展webpack的构建功能
+ 配置代码校验和提交message校验工具，从而保证代码规范
+ 迅速生成开发组件的示例以及文档
+ 根据生产环境配置build出相应的lib包

### 用cra创建项目
首先，创建项目目录。不同以往，现在我们用cra初始化react项目非常简单，只需如下简单一步。
```bash
npx create-react-app my-app
```
生成的代码目录一目了然，也比较简单，这里说下public，这个目录存放的是静态资源，编译之后都会被copy到build目录
```bash
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    └── serviceWorker.js
```
现在在看package.json中的script，自动写入了4个脚本，我们运行start即可成功启动项目。
```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject"
}
```
> 注：eject脚本会把cra的配置完全暴露病托付给开发者，而且是一个不可逆转的操作，因此，在非必要的情况下，尽可能不要去用到这个脚本。

用cra创建的项目，默认是`src/index.js`的单入口的SPA，也只有一个dev和build的环境，但在现实开发中我们的配置是多样化的，比如多入口打包，多环境打包甚至其他一些意想不到的骚操作等。就好像我们正在搭建的lib项目，需要在本地开发时运行dev能看到效果，但在build时只需要组件部分的打包。
cra虽然便捷的帮我们创建了项目，但并不能让我们自由地进行配置，若我们使用eject来反编译出配置，除了增加了项目的理解负责度和学习成本，也不利于将来的维护。因此我们应该使用[react-app-rewired](https://github.com/timarney/react-app-rewired)来配置cra生成的项目。

### react-app-rewired自定义配置
首先肯定是安装依赖：
+ 对于使用 Webpack 4 的 create-react-app 2.x
  ```bash
  npm install react-app-rewired --save-dev
  // or
  yarn add react-app-rewired --dev
  ```
+ 对于 create-react-app 1.x 或 react-scripts-ts 与 Webpack 3
  ```bash
  npm install react-app-rewired@1.6.2 --save-dev
  // or
    yarn add react-app-rewired@1.6.2 --dev
  ```
再替换package.json中的scripts，来接管react-scripts：
```json
"scripts": {
-   "start": "react-scripts start",
+   "start": "react-app-rewired start",
-   "build": "react-scripts build",
+   "build": "react-app-rewired build",
-   "test": "react-scripts test --env=jsdom",
+   "test": "react-app-rewired test --env=jsdom",
    "eject": "react-scripts eject"
}
```

然后，为了变更create-react-app的配置，我们可以采取两种方式：
+ 在package.json中配置第三方的`config-overrides.js`：
  ```json
  "config-overrides-path": "node_modules/some-preconfigured-rewire"
  ```
+ 在根目录中创建一个`config-overrides.js`文件：
  - 默认情况下，该文件导出`单个函数`，以便在开发或生产模式下自定义webpack配置
  ```javascript
  module.exports = function override(config, env) {
    //do stuff with the webpack config...
    return config;
  }
  ```
  - 也可以使文件中导出一个对象，每个属性都是一个函数。这种增强型的方式，就可以另外配置Webpack Dev Server或Jest。
  ```javascript
  module.exports = {
    // The Webpack config to use when compiling your react app for development or production.
    webpack: function(config, env) {
      // ...add your webpack config
      return config;
    },
    // The Jest config to use when running your jest tests - note that the normal rewires do not
    // work here.
    jest: function(config) {
      // ...add your jest config customisation...
      return config;
    },
    // The function to use to create a webpack dev server configuration when running the development
    // server with 'npm run start' or 'yarn start'.
    // Example: set the dev server to use a specific certificate in https.
    devServer: function(configFunction) {
      // Return the replacement function for create-react-app to use to generate the Webpack
      // Development Server config. "configFunction" is the function that would normally have
      // been used to generate the Webpack Development server config - you can use it to create
      // a starting configuration to then modify instead of having to create a config from scratch.
      return function(proxy, allowedHost) {
        // Create the default config by calling configFunction with the proxy/allowedHost parameters
        const config = configFunction(proxy, allowedHost);
        // Return your customised Webpack Development Server config.
        return config;
      };
    },
    // The paths config to use when compiling your react app for development or production.
    paths: function(paths, env) {
      // ...add your paths config
      return paths;
    },
  }
  ```
在我们项目中，需要根据编译环境调整文件入口，此时我们需要多环境配置。
  
### 用env-cmd设置环境
安装`env-cmd`后在项目根目录新建`.env`文件，我们项目里定义为library，项目结构如下：
```bash
  my-app
    ├── README.md
    ├── node_modules
    ├── package.json
    ├── .gitignore
    ├── src
+   └── .env.library
```
然后我们在`.env.library`中写入环境变量
```javascript
REACT_APP_NODE_ENV = "library"
```
然后在package.json的scripts中改写build脚本，让我们的编译带上指定的环境变量：
```json
"scripts": {
-   "build": "react-app-rewired build",
+   "build": "env-cmd -f .env.library react-app-rewired build",
}
```

### 重写config-overrides.js
安装了`react-app-rewires`和`env-cmd`后，我们就可以根据环境变量配置相应的webpack配置了，这里我们只是简单的把打包方式和css做下处理。
```javascript
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = function(config, env, options) {
  // 当值为library的时候，修改配置
  if (env === 'library') {
    const srcFile = process.env.npm_package_module || options.module;
    const libName = process.env.npm_package_name || options.name;
    config.entry = path.resolve('./', srcFile);
    // 构件库信息
    config.output = {
      path: path.resolve('./', 'build'),
      filename: libName + '.js',
      library: libName,
      libraryTarget: 'umd'
    };
    // 修改webpack optimization属性，删除代码分割逻辑
    delete config.optimization.splitChunks;
    delete config.optimization.runtimeChunk;
    // 清空plugin只保留构建CSS命名
    config.plugins = [];
    config.plugins.push(
      new MiniCssExtractPlugin({
        filename: libName + '.css'
      })
    );
    // 代码来自 react-app-rewire-create-react-library
    // 生成externals属性值，排除外部扩展，比如React
    let externals = {};
    Object.keys(process.env).forEach(key => {
      if (key.includes('npm_package_dependencies_')) {
        let pkgName = key.replace('npm_package_dependencies_', '');
        pkgName = pkgName.replace(/_/g, '-');
        // below if condition addresses scoped packages : eg: @storybook/react
        if (pkgName.startsWith('-')) {
          const scopeName = pkgName.substr(1, pkgName.indexOf('-', 1) - 1);
          const remainingPackageName = pkgName.substr(
            pkgName.indexOf('-', 1) + 1,
            pkgName.length
          );
          pkgName = `@${scopeName}/${remainingPackageName}`;
        }
        externals[pkgName] = `${pkgName}`;
      }
    });
    config.externals = externals;
  }
  return config;
};
```
上面代码中的externals，主要是用来在`process.env`中筛选出`react`，`react-dom`等作为外部依赖，写法比较独特。
至此，我们通过cra以及其他工具，就已经搭建了好了一个能用来打包lib的开发环境，接下来就可以开始组件的开发了。

### 在src下创建components目录
我们的组件放在如下目录，然后按照如下格式依次创建文件：
```bash
my-app
└── src
    ├── components
        ├── Demo
            ├── index.js
            ├── Demo.jsx
            ├── Demo.scss
            ├── Readme.md
```