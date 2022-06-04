title: Drupal8-css-guide
date: 2016-05-24 10:14:35
categories: Drupal8
tags: [css]
---

# CSS格式

## 空白

> 缩进

不要使用tab键进行缩进，采用2个空格进行缩进。

```css
@media print {
  /* This line is indented with 2 spaces, 2 spaces x 1 level of indentation. */
  .example {
    /* This line is indented with 4 spaces, 2 spaces x 2 levels of indentation. */
    padding: 0;
  }
}
```

> 空白行

* 通常，规则块之间不需要空白行作为分隔
* 如果规则块之前有单行或者块级注释，那么需要在注释之前空白行
* 如果两个规则块之间没有空白行，那么必然具有逻辑关系。如果没有的话，需要添加空白行。

```css
/* A comment describing the ruleset. */
.selector-1,
.selector-2,
.selector-3[type="text"] {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
  display: block;
  margin: 0;
  font-family: Times, "Times New Roman", sans-serif;
  color: #333;
  background: #fff;
  background: linear-gradient(#fff, rgba(0, 0, 0, 0.8));
}

/**
 * A longer comment describing this ruleset. Note
 * the blank line before the docblock.
 */
.selector-4,
.selector-5 {
  background-color: lime;
}

/* This logical grouping of rulesets has no interleaving blank lines. */
.profile {
  margin: 16px 0;
  margin: 1rem 0;
}
.profile__picture {
  float: right; /* LTR */
}
```

> 行尾

行尾绝不能有任何空白(空白或者tab)。所有的文件应该以单个空白行结尾。

文件应该以Unix行尾格式结尾(`\n`或者`LF`)，同时也是MAC OS X的默认方式，不要使用windows的行尾格式(`\r\n`或者`CRLF`)

小提示：文本编辑启用`show invisibles`。


## 注释

> 文件注释

每一个文件应该以注释开头，注释中写明文件的用途。例如：

```css
/**
 * @file
 * Short description describing the file.
 *
 * The first sentence of the long description starts here and continues on this
 * line for a while finally concluding here at the end of this paragraph.
 */
```


> 多行注释

当用于描述一个或者一组规则集的时候，注释应该遵循Doxygen注释风格(也被称作"docblock")

```css
/**
 * Short description using Doxygen-style comment format.
 *
 * The first sentence of the long description starts here and continues on this
 * line for a while finally concluding here at the end of this paragraph.
 *
 * The long description is ideal for more detailed explanations and
 * documentation. It can include example HTML, URLs, or any other information
 * that is deemed necessary or useful.
 */
.example-rule {
}
```

> 单行注释

当用于描述一个属性或者一个规则集的时候，注释应该控制在80个字符并遵循简单CSS注释风格。

```css
.example {
  /* Override the default margins. */
  margin: 0;
}

/* This is a variant of the .example component. */
.example--item {
  display: inline;
}
```

将注释直接放在属性或者它的规则集前面，并保持与它所属的属性或者规则集相同的缩进。

如果注释用于描述一个规则集，记得在规则集前面添加一空白行。


## 格式

CSS格式规范确保代码的可读性，注释的清晰性，最大化减少意外错误的情况。

> 规则集

* 每个选择器独占一行，即使有多个逗号相隔的选择器。
* 起始大括号`{`与选择器保持同行(如果有多个就与最后一个选择器保持同一行)，并以单个空格相隔。
* 结束大括号`}`保持与规则集中选择器的第一个字符保同一列。
* 每一行样式规则都应该相对于它的选择器进行缩进。

```css
.selector-alpha,
.selector-beta {
  counter-reset: section;
  text-transform: small-caps;
}
```

> 属性

* 属性名后面紧跟着`:`，然后是一个空格，接着是属性值。
* 每行样式规则以`;`结尾，包括规则集中的最后一条样式规则。
* 十六进制的颜色请使用小写，如果可能的话，使用速写，例如：`#aaa`。
* 对于需要引号的属性值，请使用双引号，例如：`font-family: "Arial Black", Arial, sans-serif; and content: " ";`。
* 如果属性值不要求带有引号，那就不加引号。例如：`background-image: url(path/image.png)`，不需要写成`background-image: url("path/image.png")`。
* 使用rem作为单位替代px，除非它造成了意外的效果。
* 选择器中的属性值加双引号，例如：`input[type="checkbox"]`。
* 对于0值，不需要带单位。例如：`margin: 0;`，代替`margin: 0px;`。
* 每一个`,`后面跟一个空格。
* 括号两边不要加空格，例如：`color: rgba(0, 0, 0, 0.8);`。

```css
/*  Basic syntax */
display: block;

/* 
 * Use shorthand syntax for hexadecimal colors when possible
 * Always use lowercase
 */
color: #fff;
color: #df7dcf;

/* Use double quotes instead of single quotes */
font-family: "Frutiger Ultra";

/* Do not attach units to zero-values */
text-shadow: 0 0 2px #ddd;

/* 
 * Use rem units preceded by px units for a safe fallback,
 * unless it creates an undesired effect.
 */
font-size: 24px;
font-size: 1.5rem;

/* Spaces MUST follow commas in property or function values */
color: rgba(0, 136, 18, 0.8)
```

> 属性声明顺序

1. Positioning：position, float, clear, top, right, bottom, left, direction, and z-index
2. Box model：display, [(max|min)-]height, [(max|min)-]width, margin, padding, border 以及它们对应的可变样式规则(例如：margin-top等.)，以及box-sizing
3. 其它

```css
.selector {
  /* Positioning declarations */
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
  /* Box model declarations */
  display: inline-block;
  width: 100%;
  padding: 10px;
  padding: 0.625rem;
  border: 1px solid #333;
  /* Other declarations */
  background: #000;
  color: #fff;
  font-family: sans-serif;
  font-size: 18px;
  font-size: 1.125rem;
}
```

> 特例

单行样式规则块，起始大括号的后面以及结束大括号的后面包含一个空格。

```css
.selector-1 { width: 10%; }
.selector-2 { width: 20%; }
.selector-3 { width: 30%; }
```

很长的，逗号相隔的属性值例如，`gradients`或者`shadows`，可能占据多行。

```css
.selector {
  background-image:
    linear-gradient(#fff, #ccc),
    linear-gradient(#f3c, #4ec);
  box-shadow:
    1px 1px 1px #000,
    2px 2px 1px 1px #ccc inset;
}
```

> Media Queries

Media queries的写法遵从规则集的写法。

* media feature和值之间加个空格。
* 所有的值以rem为单位。
* px值直接加在起始大括号之后。

```css
@media screen and (min-width: 28.125rem) { /* 450px */
  #page {
    margin-left: 20px;
    margin-left: 1.25rem;
    margin-right: 20px;
    margin-right: 1.25rem;
  }
}
```

## 杂项

> @charset语句

字符集语句(例如：`@charset "UTF-8";`)只在css文件最开头的时候才会生效。当Drupal的css合并器合并多个css文件到一个文件时，Drupal会忽略`@charset`语句。这就意味着，CSS文件不必要加`@charset`语句。CSS文件默认编码是`UTF-8`。任何CSS注释或者属性值必须以`UTF-8`编码。

> 综合例子：

```css
/**
 * @file
 * Layouts for this theme.
 */

/**
 * Column layout with horizontal scroll.
 *
 * This creates a single row of full-height, non-wrapping columns that can be
 * browsed horizontally within their parent.
 *
 * Example HTML:
 *
 * <div class="grid">
 *   <div class="cell cell-3"></div>
 *   <div class="cell cell-3"></div>
 *   <div class="cell cell-3"></div>
 * </div>
 */

/**
 * Grid container
 *
 * Must only contain '.cell' children.
 */
.grid {
  height: 100%;
  /* Remove inter-cell whitespace */
  font-size: 0;
  /* Prevent inline-block cells wrapping */
  white-space: nowrap;
}

/**
 * Grid cells
 *
 * No explicit width by default. Extend with '.cell-n' classes.
 */
.cell {
  position: relative;
  display: inline-block;
  overflow: hidden;
  box-sizing: border-box;
  height: 100%;
  /* Set the inter-cell spacing */
  padding: 0 10px;
  padding: 0.625rem;
  border: 2px solid #333;
  vertical-align: top;
  /* Reset white-space */
  white-space: normal;
  /* Reset font-size */
  font-size: 16px;
  font-size: 1rem;
}

/* Cell states */
.cell.is-animating {
  background-color: #fffdec;
}

/* Cell dimensions */
.cell-1 { width: 10%; }
.cell-2 { width: 20%; }
.cell-3 { width: 30%; }
.cell-4 { width: 40%; }
.cell-5 { width: 50%; }

/* Cell modifiers */
.cell--detail,
.cell--important {
  border-width: 4px;
}
```

------

# CSS架构

## 目标

> 可预期性

CSS代码具有易读性，容易理解。修改时只修改你需要的而不会发生其它错误。

> 可重用

CSS规则足够简约和解耦，以便可以快速从已经存在的部分构建新的组件。

> 可维护

我们会一直添加新的组件和特性，我们需要保证添加的时候会很容易，不需要破坏原有的样式解构。

> 可扩展

## 最佳实践

> 避免依赖HTML结构

* CSS应该定义某个元素在任何位置出现时应该显示的样子
* 为元素标记class，不要在CSS中使用id选择器
* 保证选择器足够短。最优的选择器就是一个class或者一个元素
* 有时候multi-part选择器很实用。例如：

```css
/**
  * Add a horizontal rule between adjacent list rows
  *
  * Could be part of an implementation of the Pears “slats” component:
  * http://pea.rs/content/slats-html5 
  */
.slat + .slat {
  border-top: 1px solid #cccccc;
}
```

然而，需要注意的是：

1. 在multi-part选择器中尽量避免原生标记，例如：`div`, `span`
2. 如果可能的话避免出现后代选择器(例如：`.my-list li`)，特别是一个组件包含另一个组件的时候
3. 避免出现2个以上的组合。下面这个例子就很糟糕：`.my-list > li > a`
4. 如果怀疑的话，直接给元素加一个class或者加上样式

> 使用组件自己的class命名组件元素

为了避免依赖html标签结构，更加明确地定义一个组件元素，可以给组件加上以`_`相连的组件名。

```css
.component {}

/* Component elements */
.component__header {}
.component__body {}
```

> 使用修饰class扩展组件

例如：

CSS

```css
/* 按钮组件 */
.button {
  /* styles */
}

/* 按钮修饰类 */
.button--primary {
  /* modifications and additions */
}
```

HTML

```html
<!-- Button variant is created by applying both component and modifier classes -->
<button class="button button--primary">Save</button>
```

> 分离出相关的css

组件的职责不是为了站点的定位和布局。不要应用`width`和`height`，除非那个元素原生带有这些属性(例如：img)。

避免使用js改变内联样式。例如，状态的变化，这时候就该定义一个名为`is-active`的class用于描述状态的变化。


Drupal8使用[SMACSS](https://smacss.com/book/)系统类归类css规则：

* Base

Base由只渲染html元素的样式规则组成，例如用于CSS reset或者[Normalize.css](http://necolas.github.com/normalize.css/)。Base中不应该包含任何类选择器。

* Layout

页面元素布局，例如grid系统

* Component

可重用、独立的UI元素。

* State

处理组件显示变化的样式集。通常，是在用户与页面交互时触发的样式变化，例如：hover，打开一个模态框等。主要方式有：

* 自定义class，以前缀'.is-'开头，例如：`.is-transitioning`, `.is-open`
* 伪类，例如：`:hover`、 `:checked`
* 以状态语义作为属性的html元素，。例如：`details[open]`
* media queries

* Theme

纯粹的可见样式，例如：`border`，`box-shadow`，`colors`，backgrounds，font properties等

> 类名格式

类名应该使用全称而不是缩写。例如：`class="button"`而不是`class="btn"`

组件的类名应该使用`-`分隔两个单词，例如：`class="button-group"`，而不是`class="buttongroup"`


> 原文地址，[请戳这里](https://www.drupal.org/node/1887862)
