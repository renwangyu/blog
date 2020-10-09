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
前段时间，产品来了个需求：输入框中划词选中的内容匹配词槽。所以只能苦逼地在国庆加班搞一下（嗯，三倍还是挺香的），之前也模模糊糊地了解过浏览器有鼠标拖动选中内容的api，这次索性跟着需求彻底捋一遍Selection和Range这对双生子，顺便把踩过的坑记录一下。文章有点长，阅读可能需要花点时间~
先来看下实现后的需求效果，好有个感官上的认识。
![划词效果gif](https://i.loli.net/2020/10/06/EKfbV827MGlSUxc.gif)

### 特殊的前提
因为是在输入框内选中，并且选中的内容加上对应颜色和下面的列表呼应，所以这里只能采用富文本的方法，没错，就是在容器上增加```contenteditable="true"```属性。不清楚的同学可以[点击这里](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/contenteditable)来了解更多，因为不是重点，咱们这就不过多展开了。当然，如果只是划词一般页面上的内容，那么就可以忽略这个前提。

### 先来看Range
![Range](https://developer.mozilla.org/zh-CN/docs/Web/API/Range)本质上是页面上的一个```起始边界点间的区域```：包含一个范围起点和范围终点。
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
Range生成后的属性非常有用，往往可以作为后续操作的依据。通过一张图，我们来了解下Range的属性概念：
![](https://i.loli.net/2020/10/06/vUCZS7jc54RyqWi.jpg)
+ startContainer：起始节点，即上图中的&lt;p&gt;的第一个文本节点。
+ startOffset：起始节点偏移量，为2。
+ endContainer：结束节点，即上图中的&lt;b&gt;的第一个文本节点。
+ endOffset：结束节点偏移量，为3。
+ collapsed：范围的开始和结束是否为同一点，上图为false。
+ commonAncestorContainer：在范围内的所有节点中最近的共同祖先节点，即上图中的&lt;p&gt;。

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
如果说Range是用于选择范围的对象，通过创建Range对象，来获取页面文档上的一个范围，那么Selection就是用来表示文档选择的。通过Firefox的一张图来看下(**一个选择可以包括零个或多个范围，不过实际上只有Firefox是支持Selection里有多个Range，其余的高级浏览器只支持一个Range，一般我们也只用一个就足够，就忽略Firefox的这种多个情况吧**)：
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
通常，如果鼠标滑动选中了内容，那么这个全局Selection对象的range就是我们当前划词的内容，当然，也是可以通过```addRange```把创建的Range添加进去。相应的，通过```getRangeAt```获取Selection的Range，因为一般都是一个，所以参数就是0(Firefox除外)。
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

#### Selection的属性
首先要清楚，选择的起点称为```锚点(anchor)```，终点称为```焦点(focus)```。
+ anchorNode：选择的起始节点。
+ anchorOffset：选择开始的anchorNode中的偏移量。
+ focusNode：选择的结束节点。
+ focusOffset：选择开始处focusNode的偏移量。
+ isCollapsed：如果未选择任何内容（空范围）或不存在，则为true。
+ rangeCount：选择中的范围数，之前说过，除Firefox外，其他浏览器最多为1。

起始细细品味，Selection和Range的属性有异曲同工之妙，都可以作为下一步操作的依据。但这里要提的是Range的起点不能在终点之后，但Selection的锚点和焦点则可以随意前后，因为我们可以向前选择，也可以向后拖动~

#### Selection的方法
选择的方法不少，其中和Range相关的有：
+ getRangeAt(i)：获取第i个Range。
+ addRange(range)：将Range添加到选择中。如果选择已有关联的范围，则除Firefox外的所有浏览器都将忽略该调用。
+ removeRange(range)：从选择中删除Range。
+ removeAllRanges()：删除所有范Range。
+ empty()：removeAllRanges的别名。

从上述方法就可知我们可以随意操作Range和Selection的转换。
还有一些和Range无关的方法：
+ collapse(node, offset)：用一个新的范围替换选定的范围，该新范围从给定的node处开始，到偏移offset处结束。
+ setPosition(node, offset)：collapse的别名。
+ collapseToStart()：折叠(替换为空范围)到选择起点，
+ collapseToEnd()：折叠到选择终点，
+ extend(node, offset)：将选择的焦点(focus)移到给定的node，位置偏移offset，
+ setBaseAndExtent(anchorNode, anchorOffset, focusNode, focusOffset)：用给定的起点anchorNode/anchorOffset和终点focusNode/focusOffset来替换选择范围。选中它们之间的所有内容。
+ selectAllChildren(node)：选择 node 的所有子节点。
+ deleteFromDocument()：从文档中删除所选择的内容。
+ containsNode(node, allowPartialContainment = false)：检查选择中是否包含 node(特别是如果第二个参数是 true 的话)。

所以要选择一个范围，可以通过创建Range，然后添加进Selection。反之，不创建Range，通过Selection的方法，仍然可以实现选择一个范围。关键就是对于元素的定位和偏移的计算。具体问题具体分析，不盲从某个方法，理解内在原理，往往能达到事半功倍的效果。

> 如果选择已存在，则首先使用removeAllRanges()将其清空。然后再addRange()来添加范围。不然除了Firefox，其他浏览器都不会理你~

### 实战需求
基本概念说了七七八八了，接下来就把实战中遇到的点点滴滴记录一下。

#### 关于输入框内变色的问题
输入框最基本的就是input和textarea，但经过调研，并不能满足部分文字颜色变换的要求，所以只能通过contenteditable="true"来当富文本操作。

#### 初始化创建变色的范围
既然定了富文本，那么自然会记录输入内容的富文本字符串。我是根据将富文本挂载进DOM后，然后选取富文本节点下的子节点```childNodes```。通过childNodes的遍历，对不是文本节点的，且具有『huaci-id』的span元素，依次创建范围。具体代码如下所示：
```javascript
// 假设页面文档已经挂载，如下形式:
// <div id="box">
//   北京
//   <span huaci-id="bymvz9" style="background-color: rgb(197, 165, 171);">招商银行</span>
//   <span huaci-id="12mdc0" style="background-color: rgb(161, 226, 188);">信用卡</span>
//   今年冬天会有
//   <span huaci-id="cd2w41" style="background-color: rgb(239, 132, 142);">活动</span>
//   吗？
// </div>
const box = document.querySelector('#box');
const childNodes = box.childNodes;
childNodes.forEach((node, index) => {
  if (node.nodeType === 1 && node.tagName.toUpperCase() === 'SPAN') {
    const id = node.getAttribute('huaci-id');
    const range = new Range();
    range.setStart(box, index);
    range.setEnd(box, index + 1);
    const selection = document.getSelection();
    // ...
  }
});
```
这里省略了业务的部分，只是把创建的过程罗列，通过对富文本子节点的遍历，正好依次能创建，也是非常的完美。

#### 输入框的划词弹框
在上面的gif图中，通过划词，能够弹出一个选择框，这个就是通过监听```document.onselectionchange```事件来实现的。这里记录两点：
+ 因为事件类似委托的形式，所以在页面上滑动，会**频繁触发**，所以要用```debounce```来缓解这个问题。
```javascript
document.onselectionchange = debounce((e) => {
  // ...
}
```
+ 对于要进行操作的元素，要有一些判断，判断的依据，仍然是上文说的Range或Selection的属性。比如我在业务中的一些过滤用的判断条件：
  - 如果是光标插入，比如鼠标随便点了下，跳过不做后续操作。
  ```javascript
  if (selection.isCollapsed) return;
  ```
  - 如果鼠标最终在输入框啥也没选或仅仅选了空格，跳过不做后续操作。
  ```javascript
  const range = selection.getRangeAt(0);
  const content = range.toString().trim();
  if (!content) return;
  ```
  - 如果划词落地的起点节点是文本节点，并且它的父节点是个带有『huaci-wrap』class的节点，也是直接跳过。
  ```javascript
  const { commonAncestorContainer, startContainer, endContainer } = range;
  if (startContainer.nodeType === 3 && startContainer.parentNode.nodeType === 1 && startContainer.parentNode.classList.contains('huaci-wrap')) {
    return;
  }
  ```
  - 如果划词落地的结束节点是文本节点，并且它的父节点是个带有『huaci-wrap』class的节点，也是直接跳过。
  ```javascript
  const { commonAncestorContainer, startContainer, endContainer } = range;
  if (endContainer.nodeType === 3 && endContainer.parentNode.nodeType === 1 && endContainer.parentNode.classList.contains('huaci-wrap')) {
    return;
  }
  ```

那么过滤了那么多，什么才是可以执行后续操作的呢~哈哈，在这个需求里，条件是这样的：
```javascript
const isMatch = 
  (commonAncestorContainer.nodeType === 3 &&
   commonAncestorContainer.parentElement.getAttribute('type') === 'huaci')
  ||
  (commonAncestorContainer.nodeType === 1 &&
   commonAncestorContainer.getAttribute('type') === 'huaci');
```
聪明的你根据上面的说明，肯定一眼就能看出这个符合条件的含义了~(我懒得写了)

#### 选中文本的高亮
那么最后的操作就是让刚才符合条件的选中的文本，再弹窗确定后加一个背景色，背景色是一个随机生成的颜色，代码也放一下，非常实用：
```javascript
const REG_HEX = /(^#?[0-9A-F]{6}$)|(^#?[0-9A-F]{3}$)/i;
// rgb字符串解析
function parseRGB(str) {
  if (typeof str === 'string' && REG_HEX.test(str)) {
    str = str.replace('#', '');
    let arr;
    if (str.length === 3) {
      arr = str.split('').map(c => (c + c));
    }
    else if (str.length === 6) {
      arr = str.match(/[a-zA-Z0-9]{2}/g);
    }
    else {
      throw new Error('wrong color format');
    }
    return arr.map((c) => parseInt(c, 16));
  }
  throw new Error('color should be string');
}
// rgb value to hsl 色相(H)、饱和度(S)、明度(L)
function rgbToHsl(rgbStr) {
  let [r, g, b] = parseRGB(rgbStr);
  r /= 255;
  g /= 255;
  b /= 255;
  let max = Math.max(r, g, b);
  let min = Math.min(r, g, b);
  let h;
  let s;
  let l = (max + min) / 2;
  if (max === min) {
    h = s = 0; // achromatic
  }
  else {
    let d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    switch (max) {
      case r: h = (g - b) / d + (g < b ? 6 : 0); break;
      case g: h = (b - r) / d + 2; break;
      case b: h = (r - g) / d + 4; break;
    }
    h /= 6;
  }
  return [h, s, l];
}
// 判断颜色属于深色还是浅色
function isColorDarkOrLight(rgbStr, limit = 0.5) {
  let [h, s, l] = rgbToHsl(rgbStr);
  return (l > limit) ? 'light' : 'dark';
}
// 生成随机颜色
function randomColor() {
  return '#' + Math.floor(Math.random() * 0xffffff).toString(16).padEnd(6, '0');
}
// 生成随机亮色
function getRandomLightColor(limit = 0.5) {
  let color = randomColor();
  while (isColorDarkOrLight(color, limit) !== 'light') {
    color = randomColor();
  }
  return color;
}
```
既然颜色有了，那么最后在点击确定后的回调里把输入框的文本变色。我的思路是，通过创建一个内容为选中的范围文案的span，替换原来位置的文本，关键代码如下：
```javascript
const selection = document.getSelection();
const range = selection.getRangeAt(0);
const newNode = document.createElement('span');
newNode.setAttribute('huaci-id', id);
newNode.setAttribute('color', obj.color);
newNode.classList.add('huaci-wrap');
newNode.style.backgroundColor = obj.color;
range.surroundContents(newNode);
selection.removeRange(range); // 这个remove还是很重要的
```
需要注意的是，最后的```removeRange```很重要，因为```surroundContents```用新的节点替换，起始又会触发```document.onselectionchange```事件，如果不清除，就会陷入无限循环，这个是很重要的点。

### 总结
自此，这个功能就基本成型，经过这个需求后，更加深了Selection和Range在实战中的应用。起始在很多阅读站点上(比如Medium)或翻译功能上，也有类似的划词操作，也是个非常有意思的功能。到目前为止写的最长的一篇，加班也结束了，趁着假期的尾巴，好好休息下~