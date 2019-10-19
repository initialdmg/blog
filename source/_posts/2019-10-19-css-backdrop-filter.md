---
title: 'CSS 中的毛玻璃效果：backdrop-filter'
date: 2019-10-19 16:44:23
excerpt: 本文介绍一个尚处在草案阶段的全新 CSS 属性：`backdrop-filter`，它最令我心仪的应用就是毛玻璃效果。
tags:
- css
- design
---

> 今天我再次打开了 Apple 的官网来研究新款 iPhone 是多么的强大。对，没错，这个自称崇尚开放自由的安卓用户（🤫:shh: 其实是因为穷）又妄想换 iPhone 了。

我们知道，苹果的设计一向是非常好的，审美一级棒！这种设计也延伸到他家的官网。今天我就打开了 Apple 官网来学习一下（咳……），发现这个[导航栏](https://www.apple.com/cn/iphone-11/specs/)好漂亮呀，各种配色的 iPhone 作为毛玻璃效果的背景，爱了……

![苹果官网iPhone导航毛玻璃效果](/blog/images/iPhone-site-nav.png)

众所周知，苹果和微软是毛玻璃效果应用的较好的两个厂商。微软在 Win7 和 WP 中用了一段时间后，就因为种种问题被砍了，后来又拾了起来，改名部将其命名为 Fluent Design。至于苹果，这种毛玻璃效果一直在其操作系统中广泛应用，其效果也确实非常好。

既然这个效果这么好，我要扒一下这是个怎么用法。通过 F12 我们搞到了它的 CSS 代码：

```css
.test {
    backdrop-filter: saturate(180%) blur(20px);
    background-color: rgba(255,255,255,0.7);
    transition: background-color 0.5s cubic-bezie(0.28, 0.11, 0.32, 1);
    transition-property: background-color,backdrop-filter, -webkit-backdrop-filter;
}
```

嗯，这个白色很好看，这个过渡效果也很赞，毛玻璃效果从这个 `backdrop-filter` 来，可是这个属性我咋没见过呢...没见过咱们就去 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/backdrop-filter) 见见。好嘛，实验性特性，草案阶段，不能怪自己。那么接下来，我们就看一下这个属性是怎么回事。

## backdrop-filter

> The `backdrop-filter` CSS property lets you apply graphical effects such as blurring or color shifting to the area behind an element. Because it applies to everything behind the element, to see the effect you must make the element or its background at least partially transparent.

> 可以让你为一个元素后面区域添加图形效果（如模糊或颜色偏移）。因为它适用于元素背后的所有元素，为了看到效果，必须使元素或其背景至少部分透明。

一个示例列表：

```css
.list {
    backdrop-filter: blur(2px);
    backdrop-filter: brightness(60%);
    backdrop-filter: contrast(40%);
    backdrop-filter: drop-shadow(4px 4px 10px blue);
    backdrop-filter: grayscale(30%);
    backdrop-filter: hue-rotate(120deg);
    backdrop-filter: invert(70%);
    backdrop-filter: opacity(20%);
    backdrop-filter: sepia(90%);
    backdrop-filter: saturate(80%);
}
```

可用的 `filter-function` 如下：

- blur: 模糊半径
- brightness: 亮度
- contrast: 对比度
- saturate: 饱和度
- opacity: 透明度
- drop-shadow: 元素阴影滤镜（投影），(x偏移, y偏移, 模糊半径, 颜色)
- grayscale: 灰度
- hue-rotate: 颜色旋转
- invert: 颜色翻转
- sepia: 暖色调（不咋用）

## 效果

现在我们再看一下苹果网站的那个磨玻璃效果是咋回事儿。

```css
nav {
    backdrop-filter: saturate(180%) blur(20px);
    background-color: rgba(255,255,255,0.7);
}
```

首先要使用这个效果，先得让元素有一点透明度。然后模糊个 20px，现在有模糊的效果了，至于加饱和度，我觉得是因为白色叠在上面作为主色，显得太淡，加一点饱和度让背景的颜色更浓一点（反正我这个开发眼看不出区别多大）。

以前这种毛玻璃效果都是使用模糊滤镜`filter: blur(20px)`加一些手段实现。现在有了这个 CSS 属性就变得简单多了。

## 兼容性

一开始说道没见多 `backdrop-filter` 这个属性，且处在草案阶段，那就看看他现在的兼容性如何。[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/backdrop-filter#Browser_compatibility)显示：

- Chrome: 76
- Edge: 17
- Safari: 9
- Firefox: 70，需通过 flag 开启
- Opera: 34，需通过 flag 开启

我才刚刚用上 Chrome 77 稳定版没多久啊。根据 [caniuse](https://caniuse.com/#search=backdrop-filter) ：Chrome 76 是 2019-7 发布的，Edge 17 是 2018-4 发布，而 Safari 9 是 2015-10 发布。由此可见，还是苹果和微软支持的给力啊，毕竟自己要用呀。

# position: sticky

除了 `dropback-filter`，还在导航栏看到了 `position: sticky` 这一用法，从字面上看也是让元素固定在某一位置的意思，但是既然有了 `fixed` ，那么它一定还是有它的不同之处。

> 该特性同样处于 `draft` 阶段。

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position#Sticky_positioning) 将其翻译为`粘性定位`。它的作用是依据元素相对于视口的位置来决定其位置为相对`relative`还是固定`fixed`。元素在跨越特定阈值前为相对定位，之后为固定定位。

```css
.sticky {
    position: sticky;
    top: 0;
}
```

这个对于导航来说就是，当视口 viewport 滚动到元素与视口top距离为0时变为固定定位，在这之前是相对定位。

**注意**：须指定 `top`, `right`, `bottom` 或 `left` 四个阈值其中之一，才可使粘性定位生效。否则其行为与相对定位相同。

> 粘性定位常用于定位字母列表的头部元素。标示 B 部分开始的头部元素在滚动 A 部分时，始终处于 A 的下方。而在开始滚动 B 部分时，B 的头部会固定在屏幕顶部，直到所有 B 的项均完成滚动后，才被 C 的头部替代。

## 兼容性

目前可见其兼容性已经OK（当然，除了IE，即使是IE11）。

- chrome: 56
- Edge: 16
- Firefox: 32
- Safari: 6
- Opera: 43