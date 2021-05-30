---
layout: post
title: CSS 代码风格约定
date: 2021-01-31 14:03
---

这个博客的 CSS 代码风格约定。

## 目标

- 限制样式的相互影响范围
- 鼓励重用样式
- 避免代码复制
- 易于查找已有样式
- 易于添加新样式
- 易于重构，而不用担心删减样式的副作用

## 方法：BEM（Block, Element, Modifier）

### Blocks / 块

块例子：

- 导航栏
- 博客文章详情
- 一组按钮

```css
.navigation {
  ...
}

.article {
  ...
}

.buttons {
  ...
}
```

引入面向对象编程概念的话，块就相当于 OOP 的类。

### Element / 元素

元素例子：

- 导航栏块由多个元素组成
- 博客文章块由标题元素、正文元素等组成

元素只能作为块的后缀存在。元素和块之间用两个下划线 `__` 作为分隔符。用途区别于普通的多单词单个分隔符 `-` 和 `_`。

```css
.article {
  ...
}

.article__title {
  ...
}

.article__text {
  ...
}
```

### Modifier / 修饰符

如果一个块的大部分样式跟另外一个是一样的，例如一篇小字体的博客文章、一个不同颜色的按钮，这个时候就需要用上修饰符。

```css
.button {
  background-color: #666;
  ...
}

.button.is_primary {
  background-color: #35d;
}
```

把修饰符想像成块和元素的可选参数。

一个样式代码块里的属性顺序：

1. 位置属性（position, top, right, z-index, display, float 等）
2. 大小（width, height, padding, margin）
3. 文字系列（font, line-height, letter-spacing, color- text-align 等）
4. 背景（background, border 等）
5. 其他（animation, transition 等）

## CSS 参考

- [muan / scribble](https://github.com/muan/scribble)
- [mmistakes / minimal-mistakes](https://github.com/mmistakes/minimal-mistakes)

**-EOF-**
