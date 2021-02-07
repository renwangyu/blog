---
title: 非常好用的js代码片段（持续收集）
date: 2021-02-07 11:12:19
tags:
- javascript
categories:
- 前端
top_img: /img/post/js-snippet.jpg
cover: /img/post/js-snippet.jpg
---

### 前言
平时开发过程经常会碰到的snippet，防止google到处乱搜，记录一下，也好没事温故知新。

### 数值部分
#### 避免js精度问题造成的小数计算
```javascript
parseFloat((0.1 + 0.2).toFixed(10))
```
#### 保留小数点（非四舍五入）
```javascript
(n, fixed) => ~~(Math.pow(10, fixed) * n) / Math.pow(10, fixed)
```

### 日期部分
#### 检查是否为工作日
```javascript
date => date.getDay() % 6 !== 0
```
#### 从日期中获取时间
```javascript
date => date.toTimeString().slice(0, 8)
```

### 随机部分
#### 两数范围内的随机
```javascript
(min, max) => Math.floor(Math.random() * (max - min + 1) + min)
```
#### 生成长度为10的随机字母数字字符串
```javascript
Math.random().toString(36).substring(2)
```
#### 生成随机16进制颜色
```javascript
() => '#' + Math.floor(Math.random() * 0xffffff).toString(16).padEnd(6, '0')
```

### 正则部分
#### 千分位
```javascript
/\B(?=(\d{3})+(?!\d))/g
```
#### html的tag匹配
```javascript
/<("[^"]*"|'[^']*'|[^'">])*>/ig
```

### 浏览器部分
#### 检查元素当前是否为聚焦状态
```javascript
el => el === document.activeElement
```
#### 判断当前tab页是否在浏览器中显示
```javascript
() => document.hidden
```
#### 检查当前用户是否为苹果设备
```javascript
() => /Mac|iPod|iPhone|iPad/.test(navigator.platform)
```
#### 检查浏览器是否支持触摸事件
```javascript
() => { ('ontouchstart' in window || window.DocumentTouch && document instanceof window.DocumentTouch) }
```

### code部分
#### 将判断条件作为对象的属性名，将处理逻辑作为对象的属性值
```javascript
const Statistics = () => console.log('执行');
const comparativeTotles = new Map([
  [0, Statistics],
  [1, Statistics],
  [2, Statistics],
  [3, Statistics]
]);
const map = val => comparativeTotles.get(val);
let getMap = map(1); //如果查找不到返回undefined
if (!getMap){
  console.log('查找不到')
} else {
  concaozuole.log('执行操作')
  getMap();
}
```
#### 一个成功就成功，所有失败才失败的promise数组
```javascript
promises => new Promise((resolve, reject) => {
  promises = promises.map(p => Promise.resolve(p)); // make sure promises are all promises
  promises.forEach(p => p.then(resolve));           // resolve this promise as soon as one resolves
  promises.reduce((a, b) => a.catch(() => b))       // reject if all promises reject
    .catch(() => reject(Error("All failed")));
})
```
#### 位操作符权限操作
```javascript
var EnvFlags = {
  None: 0,
  QQ: 1 << 0,
  Weixin: 1 << 1
}
var flag = EnvFlags.None;
flag |= EnvFlags.QQ;    // 加入QQ标识位
Flag &= ~EnvFlags.QQ;   // 清除QQ标识位
Flag |=  EnvFlags.QQ | EnvFlags.Weixin; // 加入QQ和微信标识位
```
> 使用 |= 增加标志位
使用 &= 和 ~清除标志位
使用 | 联合标识位

### 其他
#### scss对象循环用法
```scss
@for $i from 1 through $length {
  $obj: nth($items, $i);
  $name: map-get($obj, 'name');
  $width: map-get($obj, 'width');
  $height: map-get($obj, 'height');
  $left: map-get($obj, 'left');
  $top: map-get($obj, 'top');
  .item#{$i} {
    position: absolute;
    top: r($top);
    left: r($left);
    width: r($width);
    height: r($height);
    animation: xAxisItem#{$i} 2.5s linear .2s*$i infinite;
    &:after {
      content: '';
      display: block;
      position: absolute;
      width: r($width);
      height: r($height);
      @include bgUrl($name);
      animation: yAxisItem#{$i} 2.5s ease-in .2s*$i infinite;
    }
  }
}
```
#### 用字符串返回一个键盘图形
```javascript
(_=>[..."`1234567890-=~~QWERTYUIOP[]\~ASDFGHJKL;'~~ZXCVBNM,./~"].map(x=>(o+=`/${b='_'.repeat(w=x<y?2:' 667699'[x=["BS","TAB","CAPS","ENTER"][p++]||'SHIFT',p])}\|`,m+=y+(x+'    ').slice(0,w)+y+y,n+=y+b+y+y,l+=' __'+b)[73]&&(k.push(l,m,n,o),l='',m=n=o=y),m=n=o=y='|',p=l=k=[])&&k.join(``))()
```
