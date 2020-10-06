---
title: 重新认识Selection和Range
date: 2020-10-06 15:23:19
tags:
- javascript
categories:
- 前端
top_img: https://i.loli.net/2020/10/06/hWq5VwlHQLZTStG.png
cover: https://i.loli.net/2020/10/06/hWq5VwlHQLZTStG.png
---

### 前言
前段时间，产品来了个需求：输入框中划词选中的内容匹配词槽。所以只能苦逼地在国庆加班搞一下（嗯，三倍还是挺香的），之前也模模糊糊地了解过浏览器有鼠标拖动选中内容的api，这次索性跟着需求彻底捋一遍Selection和Range这对双生子，顺便把踩过的坑记录一下。先来看下实现后的需求效果，好有个感官上的认识。
![划词效果gif](https://i.loli.net/2020/10/06/EKfbV827MGlSUxc.gif)

### 特殊的前提
因为是在输入框内选中，并且选中的内容加上对应颜色和下面的列表呼应，所以这里只能采用富文本的方法，没错，就是在容器上增加```contenteditable="true"```属性。不清楚的同学可以[点击这里](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/contenteditable)来了解更多，因为不是重点，咱们这就不过多展开了。当然，如果只是划词一般页面上的内容，那么就可以忽略这个前提。

### 先来看Range
Range本质上是页面上的一个```起始边界点间的区域```：包含一个范围起点和范围终点。
> 不仅仅可以用于鼠标划词，页面上任何元素、文本都可以创建Range。

#### 创建一个Range实例
构造函数```Range()```返回一个新创建的```Range对象```，新创建的对象属于**全局Document对象**。
```javascript
const range = new Range();
```

#### 设置起始点
光有一个空Range对象实例并没有什么用，要设置起始位置才能确定一个范围区域。这里就用到了Range中的两个重要方法：
+ **setStart(startNode, startOffset)**：设置『起点』
+ **setEnd(endNode, endOffset)**：设置『终点』

这两个方法，都需要传入2个参数，第一个是文档中的某个节点，第二个是相对的偏移量，按照MDN上对这两个方法的的描述，具体又可以分为以下两种情况：
+ startNode/endNode的节点类型是```Text```，```Comment```或```CDATASection```之一，那么startOffset/endOffset指的是从起始节点算起```字符```的偏移量。
+ 其他```Node```类型节点，startOffset/endOffset是指从起始节点开始算起```子节点```的偏移量。

> 要知道节点的类型，可以参考[nodeType](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType)加以判断，比如1为元素节点、3为文本节点、4为CDATASection、8为注释节点等。

从代码来举两个例子：
```javascript
const text = '北京招商银行信用卡今年冬天会有活动吗？';
// 等text挂到div下，页面文档假设如下:
// <div id="box">北京招商银行信用卡今年冬天会有活动吗？<div>
const box = document.querySelector('#box');
const range = new Range();
range.setStart(box, 2); // 『招』
range.setEnd(box, 6); // 『信』
console.log(range.toString()); // 『招商银行』
```
因为box的firstChild是一个Text，所以设置完起点和终点的偏移量之后，range范围的内容就是『招商银行』这四个字符。这就是上面说的节点类型是```Text```，```Comment```或```CDATASection```之一的情况。再来看第二种情况：
```javascript
const richtext = '北京<span huaci-id="bymvz9" style="background-color: rgb(197, 165, 171);">招商银行</span><span huaci-id="12mdc0" style="background-color: rgb(161, 226, 188);">信用卡</span>今年冬天会有<span huaci-id="cd2w41" style="background-color: rgb(239, 132, 142);">活动</span>吗？';
// 等richtext挂到div下，页面文档假设如下:
// <div id="box">
//   北京
//   <span huaci-id="bymvz9" style="background-color: rgb(197, 165, 171);">招商银行</span>
//   <span huaci-id="12mdc0" style="background-color: rgb(161, 226, 188);">信用卡</span>
//   今年冬天会有
//   <span huaci-id="cd2w41" style="background-color: rgb(239, 132, 142);">活动</span>
//   吗？
// </div>
const box = document.querySelector('#box');
const range = new Range();
range.setStart(box, 1); // <span>招商银行</span>
range.setEnd(box, 2); // <span>信用卡</span>
console.log(range.toString()); // 『招商银行』
```
虽然打出来的内容是一样，但是此时的range是通过子元素的偏移获取到的~
> 总结，node既可以是文本节点，也可以是元素节点：对于文本节点，offset偏移的是字符数，而对于元素节点则是子节点数。注意，偏移量是**左闭右开**。

此外，还有一些其他的方法获取起点和重点，相对而言比较简单，这里就仅作罗列，不再详述。
+ setStartBefore(node)：将起点设置在node前面。
+ setStartAfter(node)：将起点设置在node后面。
+ setEndBefore(node)：将终点设置为node前面。
+ setEndAfter(node)：将终点设置为node后面。

#### Range的属性
Range生成后的属性非常有用，往往可以作为后续操作的依据。以下图为例：
![](https://i.loli.net/2020/10/06/vUCZS7jc54RyqWi.jpg)
+ startContainer，startOffset：起始节点和偏移量，

+ endContainer，endOffset —— 结束节点和偏移量，
在上例中：分别是 <b> 中的第一个文本节点和 3。

+ collapsed —— 布尔值，如果范围在同一点上开始和结束（所以范围内没有内容）则为 true，
在上例中：false

+ commonAncestorContainer —— 在范围内的所有节点中最近的共同祖先节点，
在上例中：```<p>```

#### Range的其他方法
+ selectNode(node)：设置范围以选择整个node。
+ selectNodeContents(node)：设置范围以选择整个node的内容。
+ collapse(toStart)：如果toStart=true则设置end=start，否则设置 start=end，从而折叠范围。
+ cloneRange()：创建一个具有相同起点/终点的新范围。
+ deleteContents()：从文档中删除范围内容。
+ extractContents()：从文档中删除范围内容，并将删除的内容作为```DocumentFragment```返回。
+ cloneContents()：复制范围内容，并将复制的内容作为```DocumentFragment```返回
+ insertNode(node)：在范围的起始处将node插入文档
+ surroundContents(node)：使用node将所选范围内容包裹起来。注意，所选范围必须包含其中所有元素的开始和结束标签，不完整的标签会导致方法失败并抛出error。

其中surroundContents方法，在这次需求中作为操作划词内容替换为span节点的主要api，必须在每次使用后将range清除出Selection，不然会不断触发```document.onselectionchange```事件，非常尴尬。
基本上有了上述方法，我们就能随心所欲创建我们想要的任何范围的Range对象了。相比较起点终点，这些个方法都比较容易理解，使用的时候只要参照[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Range/Range)就可以了，要注意实验特性的兼容性哈~

### Selection登场
说完Range，接下来我们看看双生子中的另一位：[Selection](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection)。
如果说Range是用于选择范围的对象，通过创建Range对象，来获取页面文档上的一个范围，那么Selection就是用来表示文档选择的。通过Firefox的一张图来看下(**一个选择可以包括零个或多个范围，不过实际上只有firefox是支持Selection里有多个Range，其余的高级浏览器只支持一个Range，一般我们也只用一个就足够，就忽略firefox的这种多个情况吧**)：
![firefox-selection.png](https://i.loli.net/2020/10/06/pcT2ax1CWr4FfVs.png)
如上图所示，蓝色就是Selection，也就是在文档中的选择范围。

#### 获取全局Selection对象
可以通过api获取全局Selection对象：
```javascript
window.getSelection();
// or
document.getSelection();
window.getSelection() === document.getSelection(); // true
```
通常，如果鼠标滑动选中了内容，那么这个全局Selection对象的range就是我们当前划词的内容，当然，也是可以通过```addRange```把创建的Range添加进去。相应的，通过```getRangeAt```获取Selection的Range，因为一般都是一个，所以参数就是0(firefox除外)。
```javascript
const range = new Range();
const sel = window.getSelection();
sel.addRange(range);
const r = sel.getRangeAt(0);
```
> 注：Selection的toString和Range的toString，都会返回被选中区域中的**纯文本**，要求变量为字符串的函数会自动对对象进行该处理。

#### 关于光标
如果我们在应用中需要获取光标的位置，就可以用Selection对象的```isCollapsed```属性，这个属性本来是标识选择是不是在同一位置(光标起始可以理解为类似的意思)。
```javascript
const sel = window.getSelection();
if (sel.isCollapsed) {
  // todo 光标信息可以通过sel进一步获取
}
```

#### 非常重要的选择事件
除了代码控制Selection和Range，最重要的还是要跟踪选择。我们可以通过下面两个事件跟踪选择：
+ ```element.onselectstart```：当选择从element上开始时触发，例如，用户按下鼠标键并开始移动鼠标。
> 阻止默认行为会使选择无法开始。

+ ```document.onselectionchange```：当选择发生变化时触发，例如，用户在页面文档里从一个划词到另一个划词。
> **重要：此事件只能绑定在document上。**

在实际使用时，最常用的是```document.onselectionchange```事件，在使用时候要像事件委托一样处理，所以要配合```window.getSelection```获取到的对象，根据进一步的信息判断是否要执行下一步代码。那么，什么是进一步的信息呢？当然就是Selection的属性啦~

#### Selection深入——属性探索