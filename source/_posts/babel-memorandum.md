---
title: babel知识备忘录
date: 2020-06-05 20:52:30
tags:
- babel
- javascript
categories:
- 前端
keywords:
- babel
top_img: https://www.ruanyifeng.com/blogimg/asset/2016/bg2016012501.png
cover: https://www.ruanyifeng.com/blogimg/asset/2016/bg2016012501.png
---

### 前言
babel已经是现代前端开发不可或缺的神器，感谢babel，让我们提前享受es6带来的爽快，也许有朝一日当浏览器对es6的普遍支持，使得我们不再依赖babel，但作为一名前端开发，babel是神一般的存在。长期记录零碎的知识点，温故知新。babel是什么，能做什么，基本概念，不再赘述，详情可见[babel文档](https://www.babeljs.cn/docs/)。

### plugin
babel为了能完成从es6到es5`(其实可以更低，但es5一般已经足够)`的转换`(解析，转换，生成)`，就要对其进行适当的配置。babel本身如同`code => code`，并不做任何操作，所以babel的插件`plugin`是非常重要的。插件按功能分为两种：
+ **语法插件**：在`解析`时候，使babel能够解析更多的语法 *(babel内部使用的解析类库叫做@babel/parser，原名babylon，作用是把代码解析为AST)*。
+ **转译插件**：在`转换`时，转换源码并生成目标类型代码 *(这是babel最重要最本质的作用，也是我们自定义插件的主要功能)*。
> 注：同一类语法可能同时存在语法插件版本和转译插件版本。如果我们使用了转译插件，就不用再使用语法插件了。

### preset
可以理解为一系列插件的集合。分类如下：
+ 官方内容：目前包括`env`，`react`, flow, minify等。
+ stage-x：这里面包含的都是当年最新规范的草案，每年更新。
  - stage-0  稻草人: 只是一个想法，经过 TC39 成员提出即可。
  - stage-1  提案: 初步尝试。
  - stage-2  初稿: 完成初步规范。
  - stage-3  候选: 完成规范和浏览器初步实现。
  - stage-4  完成: 将被添加到下一年度发布。
  > 注：低一级的 stage 会包含所有高级stage的内容，stage-0包含2、3所有内容。stage-4 在下一年更新会直接放到env中，所以没有单独的stage-4可供使用。
+ es201x, latest：这些是已经纳入到标准规范的语法 *(已废弃)*。
> 注：因为env的出现，使得es2016和es2017都已经废弃。所以我们经常可以看到es2015被单独列出来，但极少看到其他两个。latest是env的雏形，它是一个每年更新的preset，目的是包含所有 es201x。但也是因为更加灵活的env的出现，已经废弃。
使用时尽可能少用es201x和stage-x，今后的趋势是env。

### plugin和preset的顺序
+ plugin先于preset执行。
+ plugin顺序从左至右执行。
+ preset倒序从右到左执行。
> 注：preset的倒序主要是为了保证向后兼容

归根结底，很简单：plugin按需依次，preset**按照规范的时间顺序**列出就行了(最近出的在最左)。

### 常用重点细品
总体上就是这些，再来看看细节上的一些常用的重点。

#### @babel/cli
@babel/cli就是babel的命令行工具，能够在命令行中使用babel命令来编译文件。

#### @babel/node
@babel/node能让我们在node环境直接运行es6的语法，而不用进行转换。
> 注：@babel/node有点类似@babel/polyfill和@babel/register的结合。

#### @babel/register
改写commonJS的require，加上一个钩子，当使用require加载`.js`、`.jsx`、`.es`和`.es6`后缀名的文件，就会先用babel进行转码。实时转码，一般只用在开发环境。
> 注：只会对require命令加载的文件转码，一般写在入口文件(还是要用es5写)的第一行。

#### @babel/polyfill
这个名气太大了~正因为babel转换的只是语法，而对于新的API，比如Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise等全局对象，以及一些定义在全局对象上的方法(比如 Object.assign)则都不会进行转换。所以才需要这个插件去做API的转换。需要注意的点：
+ es6语法是语法，API是API，两个东西。
+ babel-polyfill必须运行在最前面，所以一般都是在**入口文件引入**，或者做为webpack的第一个entry。
+ babel-polyfill是一个整体，所以使用后打出的包会大很多(350k左右)，因为babel-polyfill所有方法都加到原型链上，即使我们只有了某一个Object.assgin，但也必须加上其他的。**解决方法是单独使用`core-js`的某个类库，因为core-js是分开的。**
+ babel-polyfill会**污染全局变量**，给很多类的原型链上都作了修改，如果我们开发的也是一个类库供其他开发者使用，这种情况就会变得非常不可控。

#### @babel/preset-env`(env)`
根据官网描述，@babel/preset-env是一个方便我们使用最新JavaScript的智能集合，避免目标环境下的语法转换(浏览器polyfills)配置管理。
env的核心目的是通过配置得知目标环境的特点，然后只做必要的转换。例如目标浏览器支持es2015，那么es2015这个preset其实是不需要的，于是代码就可以小一点,构建时间也可以缩短一些。
> 注：env不支持stage-x的插件，这点很重要！

下面配置表示，根据所有浏览器最近的2个版本(且safari大于7)的特性，将必要的代码进行转换，而这些版本已有的功能就不进行转化了。详见[browserslist](https://github.com/browserslist/browserslist)。
```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": ["last 2 versions", "safari >= 7"]
        }
      }
    ]
  ]
}
```
env还有很多配置的options([env文档](https://www.babeljs.cn/docs/babel-preset-env#options))，关键的有：
+ targets：目标环境。*也可以在package.json的browserslist属性中获取。*

#### @babel/runtime和@babel/plugin-transform-runtime
这对双生子，我们在日常经常成对使用，那么使用runtime的意义合在？
我们来看下官网的例子：
```javascript
class Circle {} // 我们定义一个class

// 可能被babel转译后编程如下代码

function _classCallCheck(instance, Constructor) {
  //... 可能很长的代码
}

var Circle = function Circle() {
  _classCallCheck(this, Circle);
};
```
在项目中，定义class是件非常频繁的操作，那么就意味着_classCallCheck可能在很多地方被重复加入，那么对于代码的冗余是显而易见的。
针对这个问题，所以配合`@babel/plugin-transform-runtime`这个转换插件，就可以把代码转换成如下形式：
```javascript
var _classCallCheck = require("@babel/runtime/helpers/classCallCheck");

var Circle = function Circle() {
  _classCallCheck(this, Circle);
};
```
而对于@babel/runtime/helpers/classCallCheck的实现恰恰就在`@babel/runtime`里，这样一来关系就很清楚了：
+ @babel/runtime是一个内部实现的集合，内部集成了：
  - `core-js`: 转换一些内置类(Promise, Symbols等等)和静态方法 (Array.from等)。绝大部分转换是这里做的。自动引入。
  - `regenerator`: 作为 core-js 的拾遗补漏，主要是`generator/yield`和`async/await`两组的支持。当代码中有使用`generators/async` 时自动引入。
  - `helpers`, 如jsx, classCallCheck等等，可以查看[babel-helpers](https://github.com/babel/babel/blob/6.x/packages/babel-helpers/src/helpers.js)。在代码中有内置的helpers使用时移除实现，并插入引用。
    > 把helpers抽离并统一起来，避免重复代码的工作还有一个叫babel-plugin-external-helpers的插件也能做。但因为我们使用的 transform-runtime已经包含了这个功能，所以不必重复使用，未来可能也会废弃babel-plugin-external-helpers。
+ @babel/plugin-transform-runtime在`转换`阶段把`代码实现`变为`代码引用`，减少代码冗余。
+ 在使用@babel/plugin-transform-runtime的时候必须把@babel/runtime当做依赖。
> 注：**@babel/plugin-transform-runtime不支持实例方法(例如：Array.prototype.includes)**，此时则只能引入@babel/polyfill。


### 参考文章
[https://www.babeljs.cn/docs/](https://www.babeljs.cn/docs/)
[https://juejin.im/post/5c19c5e0e51d4502a232c1c6](https://juejin.im/post/5c19c5e0e51d4502a232c1c6)
