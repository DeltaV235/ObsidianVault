---
title: 字体高度 font-size 与行高 line-height
created: 2024-11-09
tags:
    - Frontend
    - Font
    - CSS
---

# 字体高度

当按住鼠标左键选中一段文字的时候，这段文字背后会有一个颜色变化的区域，这个区域可以视为是这段文字的 `content-area`（内容区域）。这个区域的高度就是字体的高度。

## font-size

`font-size` 属性用于设置字体的大小。字体大小可以是绝对长度单位，也可以是相对长度单位。`font-size` 的值指的是这个字体的 `em-square` 的绝对高度。在 `em-square` 的上下还有一部分空间，这部分空间就是字体的上下间距。

`Ascender` 线到 `Descender` 线之间的高度加上 `Line Gap` 就是 `content-area` 的总高度。修改 `font-size` 会间接的影响 `content-area` 的高度。

## line-height

`line-height` 属性用于设置行高。行高是指一行文字的高度，包括文字的高度、行间距、行距等（两行文字基线之间的垂直距离）。`line-height` 的值可以是绝对长度单位，也可以是相对长度单位。

在行高没有被指定时，浏览器默认的 `line-height` 为 normal，也就是 `content-area` 的高度。

在 `line-height` 小于 `content-area` 时，`content-area` 会保持不变。在 `line-height` 大于 `content-area` 时，`content-area` 会被垂直拉伸。

## Reference

- [CSS Font Metrics](https://www.w3.org/TR/css-fonts-3/#font-metrics)
- [真的理解font-size和line-height了吗？](https://juejin.cn/post/6971673576017494053)
