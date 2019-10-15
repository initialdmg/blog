---
title: 前端性能优化意识层小结
date: 2019-10-15 23:26:45
excerpt: 本文由平时工作中遇到的性能问题，以及学习中的笔记总结而成，更偏向于概括，因此称之为意识层小结。持续更新……
tags:
- js
- 性能优化
---

> 本文由平时工作中遇到的性能问题，以及学习中的笔记总结而成，更偏向于概括，因此称之为意识层小结。持续更新……

## 前端性能问题

我们平时浏览网页和使用 Web 应用中遇到的常见性能问题大致有：

- **加载时间长**：资源载入解析慢、网络较差
- **页面卡顿**：操作响应变慢
- **内存溢出**：拖慢电脑、甚至网页崩溃

*当然还有网络问题：无法控制的因素*

这些问题的来源中最显而易见的原因就是 Web 应用本身变的越来越大，Web 应用正在吃掉更多的资源：

- web 内容越来越丰富（图片等增多）
- 功能和应用场景越来越复杂（SPA、动态页面）
- 第三方库在 Web app 中使用率增加
- webpack 等工具让我们习惯模块化引入和打包，我们开始肆无忌惮地引用第三方库

### 解决方案

提升 UX 最快方法：减少资源占用，加快载入速度。具体来说就是：**压缩文件大小，减少网络请求**。

#### 图片

- 使用jpg、webp（支持透明度、兼容性率差）等格式
- 对图片进行压缩
- 裁剪图片到必要的、适合的尺寸
- JPG/JPEG 渐进式图片（文件大小确定快，避免回流）
- 按需加载（懒加载）
- 小icon转为base64

**关于dataUrl 的思考**

- css 文件变大，阻塞渲染（同步）
- css: 还没有用到的图片（如组件内）已经被转换进css
- 模块化组件中重复的图片引用，转换后不利于利用缓存
- http2 正在普及：多路复用，并行请求

#### css、js

- 公共代码提取（css样式、js 通用方法）
- 代码分割
- js 压缩
- TreeShake （打包工具）

#### 其他

- 第三方包的使用：
  - 使用独立后的版本（lodash \ Underscore）、js 内建函数
  - 直接引用需要的（echarts）
  - 按需加载（UI 框架）
  - ...
- 服务器端 gzip 压缩

## JS 性能

这里所说的 JS 性能，不仅仅是 JS 本身，还包括DOM、浏览器引擎以及网络等方面。总的来说，可以归为一下几个方面：

- 定时器
- DOM 操作
- 减少重绘和回流
- 算法
- Workers
- Ajax

### 定时器

`setInterval` 和 `clearInterval` **同时出现*：设置定时器以后一定记得在合适的时机清除定时器，否则它可能会成为一颗炸弹，让你的页面崩溃！

在使用框架时，如果定时器伴随组件存在，那么应该在组件销毁、卸载时清楚定时器：`beforeDestroy`、 `componentWillUnmount`。

和定时器类似，`onmousemove`鼠标移动函数会随着鼠标移动一直触发其回调函数，我们可以把它想象为一个和手速绑定的定时器，如果回调中的逻辑比较复杂，那么必将非常消耗资源。因此最好对`onmousemove`进行回流。一般当帧率达到 24fps 时，人眼即认为是流畅的（视觉停留效应，24fps是电影制作标准）。在 25fps 时即 40ms 触发一次，应当可以满足需求。

```js
onmousemove = throttle(Foo, 40) // **不要忘了注销**
```

### DOM 操作、重绘与回流

JS 操作 DOM 是比较消耗资源的，当触发重绘或回流时更甚。

- DOM 读写时，将多个操作合并；缓存DOM；使用 MV* 等虚拟 DOM 框架，减少接口互操作

![浏览器解析过程](https://mmbiz.qpic.cn/mmbiz_png/aVp1YC8UV0elY5J3ByiaACdNibia22r5uZ21T0AvAZibO8Q5gkNcwfOSSib7bRB7VR7eyeFApS0R30qyOicm9s40HYgw/640)

- 回流(reflow): 元素的内容、结构、位置或尺寸发生变化，需重新计算样式和生成渲染树；读取元素的某些属性时也会触发
- 重绘(repaint): 元素样式改变，调用 GPU 重新绘制新样式

**回流必引起重绘**，回流将导致大量计算成本开销很大。

优化：

- 避免逐个修改节点样式，特别是循环
- 优化需频繁修改元素的过程：`DocumentFragment`、`display: none`等
- 合并composite读写 DOM 操作（尽管浏览器会优化）、虚拟DOM
- 替换导致回流的 API：如 `innerText` 改为 `textContent`，......

### 其他方面

1. 算法：提高效率，减少内存开销
2. Workers: 使用多进程，加快计算型任务执行速度
3. Ajax：避免同步操作阻塞进程和UI

## 内存管理

JS 引擎采用垃圾自动回收（GC）机制：当分配的内存不再被任何变量或函数使用时，便将其释放。这也是引用计数 GC 法的基本原理，其一大缺点是循环引用的内存无法释放。

```js
function f(){
  var o = {}, o2 = {};
  o.a = o2; // o 引用 o2
  o2.a = o; // o2 引用 o
  return "azerty";
}
f();
```

现代浏览器使用`标记清除算法`。如果内存没能收回，就会产生内泄漏。

### 标记清除法

将“不再使用的对象”定义为“无法达到的对象”，定时将存在标记的变量销毁并回收内存。

![标记清除法](https://user-gold-cdn.xitu.io/2019/6/17/16b637393a752456?w=800&h=423&f=gif&s=208239)

### 常见内存泄漏来源

- 意外的全局变量
- 被遗忘的定时器和回调函数
- 闭包
- DOM 引用

#### 意外的全局变量

```js
function foo() {
  bar1 = 'some text'; // 没有声明变量 实际上是全局变量 => window.bar1
  this.bar2 = 'some text' // 全局变量 => window.bar2
}
foo();
```

#### 定时器和回调函数

```js
var serverData = loadData();
setInterval(function() {
  var renderer = document.getElementById('renderer');
  if(renderer) {
    renderer.innerHTML = JSON.stringify(serverData);  // serverData无法回收
  }
}, 5000);
```

#### DOM 引用

即使删除了DOM节点，对DOM节点的引用会导致其对应的内存无法回收。

```js
var elements = {
    button: document.getElementById('button')
};
function removeButton() {
    document.body.removeChild(document.getElementById('button'));
    // 此时，仍旧存在一个全局的 #button 的引用
    // elements 字典中 button 元素仍旧在内存中，不能被 GC 回收。
}
removeButton();
elements.button = null; //释放
```

#### 闭包

在函数外部能够通过其内部（子）函数访问包含它的变量(共享作用域)。

MDN: 闭包是由函数以及创建该函数的词法环境组合而成。**这个环境包含了这个闭包创建时所能访问的所有局部变量**。

```js
var makeAdd = x => y => x + y // 工厂函数
var add5 = makeAdd(5) // function add5(y) { return 5 + y  }

console.log(add5(2)) // 7

add5 = null // 释放
```

通过闭包可读取函数的内部变量，也让变量的值始终保持在内存中

```js
var theThing = null;
function replaceThing() {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing) { console.log("hi"); } // 对于'originalThing'的引用
  };
  theThing = {
    longStr: new Array(100).join('*'),
    someMethod: function () { /* 即便是空函数 */ }
  };
};
setInterval(replaceThing, 1000);  // 定时器结合闭包或DOM引用造成严重内存泄露
```

### 内存泄漏检查

自动 GC 不是万能的……

DevTools: **Peformance** 和 **Memory**

当 Peformance 中的 JS Heap 曲线一直上升，就是有内存泄漏。
通过 Memory 中两个内存堆栈对比可以检查是哪里导致的泄漏。
