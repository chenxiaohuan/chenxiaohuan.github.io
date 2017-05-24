---
title: 快速了解Sass
tags: css
---

**Sass是什么？** 

`sass`（[sæs]）英文全称为Syntactically Awesome StyleSheets， 是一款强化 CSS 的辅助工具，它在 CSS 语法的基础上增加了变量 (variables)、嵌套 (nested rules)、混合 (mixins)、导入 (inline imports) 等高级功能，这些拓展令 CSS 更加强大与优雅。使用 Sass 以及 Sass 的样式库（如 [Compass](http://compass-style.org/)）有助于更好地组织管理样式文件，以及更高效地开发项目。`sass`是采用的`Ruby`语言编写的一款有着缩进式风格的`css`预处理语言，从第三代开始，放弃了缩进式风格，并且完全向下兼容普通的`css`代码，这一代的`sass`也被称为`scss` 。

安装教程参考下面的地址：

https://www.sass.hk/install/

## 常用语法

**1、变量** 

`sass`使用`$`符号来标识变量。任何可以用作`css`属性值的赋值都 可以用作`sass`的变量值，甚至是以空格分割的多个属性值，或以逗号分割的多个属性值。与`CSS`属性不同，变量可以在`css`规则块定义之外存在。当变量定义在`css`规则块内，那么该变量只能在此规则块内使用。

```scss
$basic-color: #333;
body {
  color: $basic-color;
}
nav {
  $width: 1200px;
  width: $width;
}
.color-default {
  color: $basic-color;
}

//编译后
body {
  color: #333;
}
nav {
  width: 1200px;
}
.color-default {
  color: #333;
}
```



**2、嵌套规则 (Nested Rules)** 

```scss
#content {
  article {
    h1 { color: #333 }
    p { margin-bottom: 1.4em }
  }
  aside { background-color: #EEE }
}
```

```css
 /* 编译后 */
#content article h1 { color: #333 }
#content article p { margin-bottom: 1.4em }
#content aside { background-color: #EEE }
```

- 父选择器 & (Referencing Parent Selectors: &)

在嵌套 CSS 规则时，有时也需要直接使用嵌套外层的父选择器，例如，当给某个元素设定 `hover` 样式时，或者当 `body` 元素有某个 classname 时，可以用 `&` 代表嵌套规则外层的父选择器。编译后的 CSS 文件中 `&` 将被替换成嵌套外层的父选择器，如果含有多层嵌套，最外层的父选择器会一层一层向下传递。`&` 必须作为选择器的第一个字符，其后可以跟随后缀生成复合的选择器。

 ```scss
article a {
  color: blue;
  &:hover { color: red }
}
 ```

```css
/* 编译后 */
article a { color: blue }
article a:hover { color: red }
```

- 群组选择器的嵌套  

```scss
nav, aside {
  a {color: blue}
}
```

```css
/* 编译后 */
nav a, aside a {color: blue}
```

- 子组合选择器和同层组合选择器：>、+和~

```scss
article {
  ~ article { border-top: 1px dashed #ccc }
  > section { background: #eee }
  dl > {
    dt { color: #333 }
    dd { color: #555 }
  }
  nav + & { margin-top: 0 }
}
```

```css
/* 编译后 */
article ~ article { border-top: 1px dashed #ccc }
article > section { background: #eee }
article dl > dt { color: #333 }
article dl > dd { color: #555 }
nav + article { margin-top: 0 }
```

- 属性嵌套 (Nested Properties)

有些 CSS 属性遵循相同的命名空间 (namespace)，比如 font-family, font-size, font-weight 都以 font 作为属性的命名空间。为了便于管理这样的属性，同时也为了避免了重复输入，Sass 允许将属性嵌套在命名空间中。

```scss
nav {
  border: {
  style: solid;
  width: 1px;
  color: #ccc;
  }
}

.nav {
  border: 1px solid #ccc {
  left: 0px;
  right: 0px;
  }
}
```

```css
/* 编译后 */
nav {
  border-style: solid;
  border-width: 1px;
  border-color: #ccc;
}

.nav {
  border: 1px solid #ccc;
  border-left: 0px;
  border-right: 0px;
}
```

**3、@import,导入SASS文件 **

```scss
@import "reset.scss";
```



**4、注释 /* */ 与 // (Comments: /* */ and //) **

Sass 支持标准的 CSS 多行注释 `/* */`，以及单行注释 `//`，前者会 被完整输出到编译后的 CSS 文件中，而后者则不会。

```scss
body {
  color: #333; // 这种注释内容不会出现在生成的css文件中
  padding: 0; /* 这种注释内容会出现在生成的css文件中 */
}
```

**5、 混合器 **

混合器使用`@mixin`标识符定义。通常用于把样式中的通用样式抽离出来，然后轻松地在其他地方重用。

`@include`调用会把混合器中的所有样式提取出来放在`@include`被调用的地方。

从易读性和可维护性方面来说，语义化的命名混合器能让你避免重复使用混合器，可以很容易地在样式表的不同地方共享样式。

```scss
@mixin rounded-corners {
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}

.notice {
  background-color: green;
  border: 2px solid #00aa00;
  @include rounded-corners;
}
```

```css
/* 编译后 */
.notice {
  background-color: green;
  border: 2px solid #00aa00;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}
```

**6、@extend，选择器继承**

选择器继承是说一个选择器可以继承为另一个选择器定义的所有样式。这个通过@extend语法实现。

使用@extend的选择器不仅会继承被继承选择器自身的所有样式，任何跟被继承选择器有关的组合选择器样式也会被使用@extend的选择器以组合选择器的形式继承。

```scss
.error {
  border: 1px red;
  background-color: #fdd;
}
.error a{ 
  color: red;
  font-weight: 100;
}
h1.error { 
  font-size: 1.2rem;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}
```

```css
/* 编译后 */
.error, .seriousError {
  border: 1px red;
  background-color: #fdd;
}
.error a, .seriousError a { 
  color: red;
  font-weight: 100;
}
h1.error, h1.seriousError { 
  font-size: 1.2rem;
}
.seriousError {
  border-width: 3px;
}
```

本文简单的介绍了下本人常用的Sass语法，其他如运算和一些指令等，因篇幅有限就不再介绍，如果想了解更多Sass相关语法，可到官方文档查看。感谢您花费时间浏览本文，如果本文有错误如错别字或者描述不当，请到 [issues](https://github.com/chenxiaohuan/chenxiaohuan.github.io/issues) 指出，本人会及时修正，如果有什么更好的建议也欢迎各位提出。



更多Sass语法查看：[https://www.sass.hk/docs/](https://www.sass.hk/docs/)