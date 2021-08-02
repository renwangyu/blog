---
title: 2021了，这样来搞react开发环境：lint
date: 2021-08-02 17:32:47
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
书接前文，我们已经搭建好了环境，为了保证项目的工程化和规范化，我们可以通过增加lint规则来增强项目的健壮性。让我们一步步来看那些把项目武装起来的神器吧~本文不做详细的知识讲述，只是把最常用的lint扩展放到项目中去。
> 注：全文都以上文用create-react-app生成的项目举例子。

### eslint
eslint是提供一个插件化的javascript代码检测工具，是代码质量的强力保证。初始化的时候可以通过全局安装，代码如下：
```bash
# 全局安装 ESLint
npm install -g eslint
# 初始化 ESLint 配置
eslint --init
```
在经过一系列提示问答之后，就会在项目根目录生成`.eslintrc.js`文件(当然，你也可以在package.json文件中添加eslintConfig字段进行配置)，在这个文件中配置规则和插件就行了。
如果是cra生成的项目默认已经安装了eslint，所以也不用全局安装。除了eslint，还会默认安装如下工具：
```bash
"babel-eslint": "7.2.3",
"eslint": "4.10.0",
"eslint-config-react-app": "^2.1.0",
"eslint-loader": "1.9.0",
"eslint-plugin-flowtype": "2.39.1",
"eslint-plugin-import": "2.8.0",
"eslint-plugin-jsx-a11y": "5.1.1",
"eslint-plugin-react": "7.4.0",
```
具体配置字段可以参考[官方配置文档](https://eslint.bootcss.com/docs/user-guide/configuring)。这里单独拎出几个常用的配置字段：
+ parser：配置成`babel-eslint`，覆盖默认的esprima。
+ extends：如果是cra创建的react项目，一般会用`plugin:react/recommended`(eslint-config-react-app里包含eslint-plugin-react)。此外，有两个比较著名的最简实践，airbnb和standard。这个可以看开发者的风格来选择，安装相应的eslint-config-airbnb或eslint-config-standard。
+ parserOptions：配置想要支持的js语言选项。一般react都要支持jsx，编译到es5等。
  ```javascript
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 2015,
    sourceType: 'module',
  }
  ```
+ rules：规则，具体的可以根据实际情况增加或覆盖extends。常见配置如下：
  ```javascript
  rules: {
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    'no-console': ['off'],
    'no-debugger': ['off'],
    'max-lines': ['error', { 'max': 600, 'skipBlankLines': true }], // 每个文件不算空行，最多600行
    'max-len': ['error', 120, 2], // 单行最多120
    'max-statements': ['error', 80], // 单个函数最多80行
    'no-multiple-empty-lines': ['error', { max: 2, maxEOF: 1 }], // 空行最多不能超过2行
    'max-depth': ['error', 4], // 代码最多允许4层嵌套
    'no-unneeded-ternary': ['error'], // 禁止不必要的嵌套 var isYes = answer === 1 ? true : false;简单的判断用三元表达式代替
    'react/react-in-jsx-scope': ['off'], // 有jsx的文件要有import React from 'react'
    'react/jsx-filename-extension': ['off'] // jsx要在jsx的扩展名文件中
  }
  ```
上述操作后，基本上就已经配置好了我们的eslint，这里在提一下如果我们不需要对某些文件做校验，可以编写`.eslintignore`文件，加上我们的忽略的文件名，如：
```bash
build/*.js
src/assets
public
.eslintrc.js
```

### commitlint