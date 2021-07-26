---
title: 前端面试笔记：css
date: 2021-07-24 13:37:48
tags:
- 面试
categories:
- 前端
keywords: css
top_img:
---

### 块格式化上下文BFC（Block Formatting Context）
+ 含义：BFC是CSS中的一种渲染机制。是一个拥有独立渲染区域的盒子，规定了内部元素如何布局，并且盒子内部元素与外部元素互不影响。
+ 特性：
  + 从BFC顶部开始，每个元素上下排列
  + 两个兄弟盒之间的垂直距离由`margin`属性决定，上下元素`margin`重叠
  + 每个元素的margin-box紧贴容器的border-box
  + BFC中浮动元素参与高度计算
  + 形成了BFC的区域不会与float box重叠
+ 触发条件：
  + 根元素即`html`
  + `float`不为`none`
  + `overflow`的值不为`visible`
  + `display`的值为`inline-block`、`table-cell`、`table-caption`
  + `position`的值为`absolute`或`fixed`
+ 解决问题：
  + 上下元素margin不重叠
  + 左右结构（left: float: left,  right: overflow: hidden）
  + 解决高度塌陷清除浮动（父元素添加overflow: hidden）

### opacity: 0、visibility: hidden、display: none优劣和适用场景
+ opacity:  页面加载dom，可以点击，无继承性，修改子元素无法展示，重绘
+ visibility：页面加载dom，不可点击，有继承性，修改子元素可以显示，重绘
+ display：页面加载dom，但是render不会渲染，不可点击，无继承性，修改子元素无法展示，回流

### 绝对1px解决方案
+ 造成原因：因为不同机型（视网膜屏）设备像素比DPR的不同使得在1物理像素下的css像素达到2甚至3，而导致显示的1px变粗
+ 解决方法：
  方案 | 描述 | 优点 | 缺点
  :-: | :-: | :-: | :-:
  border | 使用border的宽度为0.5或0.333px可以直接进行解决 | 简单 | android不支持 | 
  box-shadow | 使用box-shadow进行外表化的边框实现 | 简单，圆角也可以 | 模拟实现，终究只是阴影 |
  border-image | 边框设置border-image进行边框图片的设置 | 简单 | 边框颜色变化需要重做图片 |
  伪元素(单边) | 使用伪元素创建一个宽度为100%，高度为1的元素，而后使用transform进行scale变换 | 全机型兼容，实现了真正的1px，而且可以圆角 | after伪元素占用，清除浮动受影响 |
  伪元素(四边) | 创建一个父元素二倍宽高的元素，在用单边的方案 | 全机型兼容，实现了真正的1px，而且可以圆角 | after伪元素占用，清除浮动受影响，部分元素不支持伪元素 |
  viewport | 初始化页面中，针对不同的window.devicePixelRatio进行meta标签的viewport属性的值进行设置 | 全机型兼容 | meta级改动，可能影响全局 |

### 水平垂直居中方案
+ 对于内联元素，父级display设置为table-cell, vertical-align: middle, text-align: center
+ 父级使用绝对布局，子元素有宽高，top、bottom、left、right设置为0，margin: auto
+ 父级flex布局，设置两个轴的布局形式（justify-content：center, align-items: center）
+ 父级grid布局，子元素设置自身对父级对布局形式（align-self: center, justify-self: center）
+ 子元素绝对定位，调节margin-top和margin-left
+ 子元素绝对定位，调节transform: translateX和transform: translateY