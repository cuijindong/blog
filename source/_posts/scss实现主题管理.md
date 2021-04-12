---
title: scss实现主题切换
categories: css
tags: 
- 主题切换
wordcount: true
---
## 主题切换
关于主题切换应该不是很陌生，一般都是白天黑夜进行切换提高用户体验。利用 **setAttribute** 和 **scss** 可以实现一个便于管理的主题切换。
## 实现思路
1. 利用**setAttribute**为html设置主题标识。
2. 使用scss map 管理主题样式。
3. 遍历scss map 获取对应主题的样式。
## setAttribute
>设置指定元素上的某个属性值。如果属性已经存在，则更新该值；否则，使用指定的名称和值添加一个新的属性。

利用setAttribute为html设置属性来区分主题。下面为html设置一个theme属性，属性值为'light'
```javascript
document.documentElement.setAttribute('theme', 'light')
```
添加个按钮用于主题切换，下面例子只设置两种主题，一个是白天'light'，一个是黑天'dark'。
```html
<body>
  <button id="but">切换</button>
  <div class="text">theme</div>
  <script>
    let theme = 'light'
    document.documentElement.setAttribute('theme', theme)
    document.getElementById('but').onclick = function () {
      theme = theme === 'light' ? 'dark' : 'light'
      document.documentElement.setAttribute('theme', theme)
    }
  </script>
</body>
```
上述代码可实现为html设置一个theme属性，点击按钮进行属性值切换。

![](https://img-blog.csdnimg.cn/20210411161818955.PNG)
## scss
通过上述代码已经可是实现主题管理了，例如：
```css
[theme="light"] .test{
  color: #000
}
[theme="dark"] .test{
  color: #fff
}
```
通过theme经行匹配使用哪个样式，但是问题在于这样主题样式会散落在各个地方，及其不方便管理，所以要借助scss帮忙管理主题样式。下面介绍一些scss的API。
### 定义混合指令@mixin
>混合指令的用法是在 @mixin 后添加名称与样式，比如名为 color 的混合通过下面的代码定义：
```scss
@mixin color {
  color: #ff0000;
}
```
混入指令也可以接受传值：

```scss
@mixin color($color) {
  color: $color;
}
```
### 引用混合样式 @include
>使用 @include 指令引用混合样式，格式是在其后添加混合名称，以及需要的参数（可选）：
```scss
// 使用前面定义好的混入 color，并传参数
.test{
  @include color(#fff);
}
```
编译为：
```css
.test{
  color: #fff;
}
```
### 向混合样式中导入内容@content
>在引用混合样式的时候，可以先将一段代码导入到混合指令中，然后再输出混合样式，额外导入的部分将出现在 @content 标志的地方：

```scss
@mixin color {
  body {
    @content;
  }
}

@include color {
  color: #fff;
}
```
编译为：
```css
body{
  color: #fff;
}
```
### 函数指令@function
>Sass 支持自定义函数，并能在任何属性值或 Sass script 中使用，一个函数可以含有多条语句，需要调用 @return 输出结果。
```scss
@function themed($key) {
  @return $key;
}

.test{
  color: themed(#fff)
}
```
编译为：
```css
.test{
  color: #fff;
}
```
### 父选择器 &
>在嵌套 CSS 规则时，有时也需要直接使用嵌套外层的父选择器，例如，当给某个元素设定 hover 样式时，或者当 body 元素有某个 classname 时，可以用 & 代表嵌套规则外层的父选择器。

```scss
@mixin themeify {
  [theme = 'dark'] & {
    color: #fff;
  }
}

.test{
  @include themeify;
}
```
编译为：
```css
[theme = "dark"] .test{
  color: #fff;
}
```
### Maps
>Maps可视为键值对的集合，键被用于定位值 在css种没有对应的概念。 和Lists不同Maps必须被圆括号包围，键值对被都好分割 。 Maps中的keys和values可以是sassscript的任何对象。（包括任意的sassscript表达式 arbitrary SassScript expressions） 和Lists一样Maps主要为sassscript函数服务，如 map-get函数用于查找键值，map-merge函数用于map和新加的键值融合，@each命令可添加样式到一个map中的每个键值对。 Maps可用于任何Lists可用的地方，在List函数中 Map会被自动转换为List ， 如 (key1: value1, key2: value2)会被List函数转换为 key1 value1, key2 value2 ，反之则不能。

```scss
$themes: (
  font_color: #000,
  background_color: #fff
);

.test {
  color: map-get($themes, font_color);
}
```
编译为：
```css
.test{
  color: #000;
}
```
### 编译Maps @each
>@each 指令的格式是 $var in <list>, $var 可以是任何变量名，比如 $length 或者 $name，而 <list> 是一连串的值，也就是值列表。

```scss
$themes: (
  light: (
    font_color: #000,
    background_color: #fff
  ),
  dark: (
    font_color: #FFF,
    background_color: #000
  )
);
@mixin themeify{
  @each $theme-name, $theme-map in $themes {
    [theme = "#{$theme-name}"] & {
      color: map-get($theme-map, 'font_color')
    }
  }
};

.test{
  @include themeify;
}
```
首先设置了一个名为 **$themes** 的Maps，其中包含两个名为light和dark的Maps。然后指定一个名为 **themeify** 的混入，遍历 **$themes** 。将键指定给theme，在值中取'font_color'。上述代码编译为：
```css
[theme = "light"] .test{
  color: #000;
}

[theme = "dark"] .test{
  color: #fff;
}
```
## 实现主题管理
通过上述scss的api介绍，已经很容易的实现主题管理了。
先创建一个theme.scss 用于管理主题和主题样式。
```scss
// theme.scss
$themes: (
  light: (
    font_color: #000,
    background_color: #fff
  ),
  dark: (
    font_color: #FFF,
    background_color: #000
  )
)
```
然后创建handle.scss 用于遍历 **$themes** 提供混入。
```scss
@import './theme.scss';

// 遍历主题map
@mixin themeify {
  @each $theme-name, $theme-map in $themes {
    $theme-map: $theme-map !global;   // 将$theme-map提升到全局防止themed方法获取不到。
    [theme = "#{$theme-name}"] & {
      @content;
    }
  }
};
// 获取样式的值
@function themed($key) {
  @return map-get($theme-map, $key);
}
// 提供color混入
@mixin color($color) {
  @include themeify{
    color: themed($color);
  }
}
// 提供backgroundColor混入
@mixin backgroundColor($color) {
  @include themeify{
    background-color: themed($color);
  }
}
```
## 使用
```scss
body{
  @include backgroundColor('background_color');

  display: flex;
  align-items: center;
  justify-content: center;
}
.text{
  font-size: 30px;
  @include color('font_color');
}
```
上述代码会编译成：
```css
[theme="light"] body {
    background-color: #fff;
}
[theme="dark"] body {
    background-color: #000;
}
body {
    display: flex;
    align-items: center;
    justify-content: center;
}
[theme="light"] .text {
    color: #000;
}
[theme="dark"] .text {
    color: #fff;
}
.text {
    font-size: 30px;
}
```
通过html的theme属性决定使用那套样式。效果图如下：

![](https://s31.aconvert.com/convert/p3r68-cdx67/hqpfx-9aqnp.gif)

## 总结
通过scss提供的Maps进行主题管理可以将主题样式都放在一起，方便管理和维护。对于主题的添加和删除也很灵活。
## 源码地址
[源码](https://github.com/cuijindong/example/tree/master/css/theme)