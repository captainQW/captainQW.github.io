title: web开发规范
date: 2016-10-07 21:55:39
tags:
  - web
categories:
  - web
---

## 文件编码

前端开发涉及的所文件统一使用`utf-8`编码

## 文件命名规则

全部小写方式，以下划线分隔

例：my_project

> 特例：drupal中的模版文件


## HTML

### 语法

- 缩进使用2个空格；
- 嵌套的节点应该缩进；
- 在属性上，使用双引号，不要使用单引号；
- 属性名全小写，用中划线做分隔符；
- 不要在自动闭合标签结尾处使用斜线（[HTML5 规范](http://dev.w3.org/html5/spec-author-view/syntax.html#syntax-start-tag) 指出他们是可选的）；
- 不要忽略可选的关闭标签，例：`</li>` 和 `</body>`。


```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page title</title>
  </head>
  <body>
    <img src="images/company_logo.png" alt="Company">
    <h1 class="hello-world">Hello, world!</h1>
  </body>
</html>
```
<!--more-->

### HTML5 doctype

在页面开头使用这个简单地doctype来启用标准模式，使其在每个浏览器中尽可能一致的展现；

```html
<!DOCTYPE html>
<html>
  ...
</html>
```

### lang属性

根据HTML5规范：

> 应在html标签上加上lang属性。这会给语音工具和翻译工具帮助，告诉它们应当怎么去发音和翻译。

在sitepoint上可以查到[语言列表](http://reference.sitepoint.com/html/lang-codes)；

但sitepoint只是给出了语言的大类，例如中文只给出了zh，但是没有区分香港，台湾，大陆。而微软给出了一份更加[详细的语言列表] [language]，其中细分了zh-cn, zh-hk, zh-tw。

[language]: http://msdn.microsoft.com/en-us/library/ms533052(v=vs.85).aspx "详细的语言列表"

### 字符编码

通过声明一个明确的字符编码，让浏览器轻松、快速的确定适合网页内容的渲染方式，通常指定为`UTF-8`。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
  </head>
  ...
</html>
```

### IE兼容模式

用 `<meta>` 标签可以指定页面应该用什么版本的IE来渲染；

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
  </head>
  ...
</html>
```

### 引入CSS, JS

根据HTML5规范, 通常在引入CSS和JS时不需要指明 type，因为 `text/css` 和 `text/javascript` 分别是他们的默认值。

```html
<!-- External CSS -->
<link rel="stylesheet" href="code_guide.css">

<!-- In-document CSS -->
<style>
  ...
</style>

<!-- External JS -->
<script src="code_guide.js"></script>

<!-- In-document JS -->
<script>
  ...
</script>
```

### 属性顺序

属性应该按照特定的顺序出现以保证易读性；

- `class`
- `id`
- `name`
- `data-*`
- `src`, `for`, `type`, `href`, `value` , `max-length`, `max`, `min`, `pattern`
- `placeholder`, `title`, `alt`
- `aria-*`, `role`
- `required`, `readonly`, `disabled`

class是为高可复用组件设计的，所以应处在第一位；

id更加具体且应该尽量少使用，所以将它放在第二位。

```html
<a class="..." id="..." data-modal="toggle" href="#">Example link</a>

<input class="form-control" type="text">

<img src="..." alt="...">
```

### boolean属性

boolean属性指不需要声明取值的属性，XHTML需要每个属性声明取值，但是HTML5并不需要；

更多内容可以参考 [WhatWG section on boolean attributes](http://www.whatwg.org/specs/web-apps/current-work/multipage/common-microsyntaxes.html#boolean-attributes)：

> boolean属性的存在表示取值为true，不存在则表示取值为false。

```html
<input type="text" disabled>

<input type="checkbox" value="1" checked>

<select>
  <option value="1" selected>1</option>
</select>
```

### 减少标签数量

在编写HTML代码时，需要尽量避免多余的父节点；

很多时候，需要通过迭代和重构来使HTML变得更少。

```html
<!-- Not well -->
<span class="avatar">
    <img src="...">
</span>

<!-- Better -->
<img class="avatar" src="...">
```

### 语义化

使用符合语义的标签书写 HTML 文档, 选择恰当的元素表达所需的含义。

- 结构性元素
  1. `p` 表示段落. 只能包含内联元素, 不能包含块级元素;
  2. `li` 本身无特殊含义, 可用于布局. 几乎可以包含任何元素;
  3. `br` 表示换行符;
  4. `hr` 表示水平分割线;
  5. `h1-h6` 表示标题. 其中 h1 用于表示当前页面最重要的内容的标题;
  6. `blockquote` 表示引用, 可以包含多个段落. 请勿纯粹为了缩进而使用blockquote, 大部分浏览器默认将 blockquote 渲染为带有左右缩进;
  7. `pre` 表示一段格式化好的文本；


- 头部元素
  1. `title` 每个页面必须有且仅有一个 title 元素;
  2. `base` 可用场景：首页、频道等大部分链接都为新窗口打开的页面;
  3. `link`用于引入 css 资源时, 可省去 media(默认为all) 和 type(默认为text/css) 属性;
  4. `style` type 默认为 text/css, 可以省去;
  5. `script` type 属性可以省去; 不赞成使用lang属性; 不要使用古老的<!– //–>这种hack脚本, 它用于阻止第一代浏览器(Netscape 1和Mosaic)将脚本显示成文字;
  6. `noscript` 在用户代理不支持 JavaScript 的情况下提供说明；


- 文本元素
  1. `a` 存在 href 属性时表示链接, 无 href 属性但有 name 属性表示锚点;
  2. `em,strong em` 表示句意强调, 加与不加会引起语义变化, 可用于表示不同的心情或语调; strong 表示重要性强调, 可用于局部或全局, strong强调的是重要性, 不会改变句意;
  3. `abbr` 表示缩写;
  4. `sub,sup` 主要用于数学和化学公式, sup还可用于脚注;
  5. `span` 本身无特殊含义;
  6. `ins,del` 分别表示从文档中增加(插入)和删除；


- 媒体元素
  1. `img` 请勿将img元素作为定位布局的工具, 不要用他显示空白图片; 必要时给img元素增加alt属性;
  2. `object` 可以用来插入Flash;


- 列表元素
  1. `dl` 表示关联列表, `dd`是对`dt`的解释; `dt`和dd的对应关系比较随意：一个`dt`对应多个`dd`、多个`dt`对应一个`dd`、多个`dt`对应多个`dd`, 都合法; 可用于名词/单词解释、日程列表、站点目录;
  2. `ul` 表示无序列表;
  3. `ol` 表示有序列表, 可用于排行榜等;
  4. `li` 表示列表项, 必须是`ul/ol`的子元素;


- 表单元素
  1. 推荐使用 `button` 代替 `input`, 但必须声明 `type`;
  2. 推荐使用 `fieldset`, `legend` 组织表单；
  3. 表单元素的 `name` 不能设定为 `action`, `enctype`, `method`, `novalidate`, `target`, `submit` 会导致表单提交混乱；

### id 和 class 命名约定

- 建议少用id，必须保证id唯一性
id 和 class 的命名总规则为： 内容优先, 位置次之, 表现为辅. 首先根据内容来命名, 比如 main-nav，其次是位置，比如 top-nav, left-nav ，最后再结合表现来定, 比如 skin-blue, present-tab, col-main
- id 和 class 名称一律小写, 多个单词用连字符连接, 比如 recommend-presents
- id 和 class 名称中只能出现小写的 `26` 个英文字母、数字和连字符(-), 任何其它字符都严禁出现
- id 和 class 尽量用英文单词命名
- 在不影响语义的情况下, id 和 class 名称中可以适当采用英文单词缩写, 比如 `col`, `nav`, `hd`, `bd`, `ft` 等, 但切忌自造缩写
- id 和 class 名称中的第一个词必须是单词全拼或语义非常清晰的单词缩写, 比如 `present`, `col`


## CSS

### 缩进

使用2个空格缩进

```css
.element {
  position: absolute;
  top: 10px;
  left: 10px;

  border-radius: 10px;
  width: 50px;
  height: 50px;
}
```

### 分号

每个属性声明末尾都要加分号

```css
.element {
  width: 20px;
  height: 20px;

  background-color: red;
}
```

### 空格

以下几种情况不需要空格：
- 属性名后
- 多个规则的分隔符','前
- `!important` '!'后
- 属性值中'('后和')'前
- 行末不要有多余的空格

以下几种情况需要空格：
- 属性值前
- 选择器`>`, `+`, `~`前后
- `{`前
- `!important` '!'前
- 属性值中的`,`后
- 注释`/*`后和`*/`前
- 属性冒号后

```css
/* not good */
.element {
  color :red! important;
  background-color: rgba(0,0,0,.5);
}

/* good */
.element {
  color: red !important;
  background-color: rgba(0, 0, 0, .5);
}

/* not good */
.element ,
.dialog{
  ...
}

/* good */
.element,
.dialog {

}

/* not good */
.element>.dialog{
  ...
}

/* good */
.element > .dialog {
  ...
}

/* not good */
.element{
  ...
}

/* good */
.element {
  ...
}
```

### 换行

以下几种情况不需要换行：
- `{`前

以下几种情况需要换行：
- `{`后和`}`前
- 每个属性独占一行
- 多个规则的分隔符`,`后

```css
/* not good */
.element
{color: red; background-color: black;}

/* good */
.element {
  color: red;
  background-color: black;
}

/* not good */
.element, .dialog {
  ...
}

/* good */
.element,
.dialog {
  ...
}
```

### 注释

注释统一用`/* */`

缩进与下一行代码保持一致

可位于一个代码行的末尾，与代码间隔一个空格

```css
/* Modal header */
.modal-header {
  ...
}

/*
 * Modal header
 */
.modal-header {
  ...
}

.modal-header {
  /* 50px */
  width: 50px;

  color: red; /* color red */
}
```

### 引号

最外层统一使用双引号；

url的内容要用引号；

属性选择器中的属性值需要引号。

```css
.element:after {
  content: "";
  background-image: url("logo.png");
}

li[data-type="single"] {
  ...
}
```

### 命名

- 类名使用小写字母，以中划线分隔
- id采用驼峰式命名

```css
/* class */
.element-content {
  ...
}

/* id */
#myDialog {
  ...
}
```

### 属性声明顺序

相关的属性声明按照顺序做分组处理，组之间需要有一个空行，这个可选

由外向里写属性，先是外壳怎么属性，`display`，`float`，接着是定位相关的属性，`position`等，然后是开始描外边距`margin`，边框粗细、颜色`border`，然后是内边距，哦，当然，可能有时候还来个圆角，框子描完了就开始描高度宽度`width`, `height`，接着是里面的内容，字体怎么显示（字体颜色，居中还是居左等等），然后是背景图啊之类的，最后是一些杂项，透明度，鼠标，动画等

```css
.declaration-order {
  display: block;
  float: right;

  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  border: 1px solid #e5e5e5;
  border-radius: 3px;
  width: 100px;
  height: 100px;

  font: normal 13px "Helvetica Neue", sans-serif;
  line-height: 1.5;
  text-align: center;

  color: #333;
  background-color: #f5f5f5;

  opacity: 1;
}
// 下面是推荐的属性的顺序
[
  [
    "display",
    "visibility",
    "float",
    "clear",
    "overflow",
    "overflow-x",
    "overflow-y",
    "clip",
    "zoom"
  ],
  [
    "table-layout",
    "empty-cells",
    "caption-side",
    "border-spacing",
    "border-collapse",
    "list-style",
    "list-style-position",
    "list-style-type",
    "list-style-image"
  ],
  [
    "-webkit-box-orient",
    "-webkit-box-direction",
    "-webkit-box-decoration-break",
    "-webkit-box-pack",
    "-webkit-box-align",
    "-webkit-box-flex"
  ],
  [
    "position",
    "top",
    "right",
    "bottom",
    "left",
    "z-index"
  ],
  [
    "margin",
    "margin-top",
    "margin-right",
    "margin-bottom",
    "margin-left",
    "-webkit-box-sizing",
    "-moz-box-sizing",
    "box-sizing",
    "border",
    "border-width",
    "border-style",
    "border-color",
    "border-top",
    "border-top-width",
    "border-top-style",
    "border-top-color",
    "border-right",
    "border-right-width",
    "border-right-style",
    "border-right-color",
    "border-bottom",
    "border-bottom-width",
    "border-bottom-style",
    "border-bottom-color",
    "border-left",
    "border-left-width",
    "border-left-style",
    "border-left-color",
    "-webkit-border-radius",
    "-moz-border-radius",
    "border-radius",
    "-webkit-border-top-left-radius",
    "-moz-border-radius-topleft",
    "border-top-left-radius",
    "-webkit-border-top-right-radius",
    "-moz-border-radius-topright",
    "border-top-right-radius",
    "-webkit-border-bottom-right-radius",
    "-moz-border-radius-bottomright",
    "border-bottom-right-radius",
    "-webkit-border-bottom-left-radius",
    "-moz-border-radius-bottomleft",
    "border-bottom-left-radius",
    "-webkit-border-image",
    "-moz-border-image",
    "-o-border-image",
    "border-image",
    "-webkit-border-image-source",
    "-moz-border-image-source",
    "-o-border-image-source",
    "border-image-source",
    "-webkit-border-image-slice",
    "-moz-border-image-slice",
    "-o-border-image-slice",
    "border-image-slice",
    "-webkit-border-image-width",
    "-moz-border-image-width",
    "-o-border-image-width",
    "border-image-width",
    "-webkit-border-image-outset",
    "-moz-border-image-outset",
    "-o-border-image-outset",
    "border-image-outset",
    "-webkit-border-image-repeat",
    "-moz-border-image-repeat",
    "-o-border-image-repeat",
    "border-image-repeat",
    "padding",
    "padding-top",
    "padding-right",
    "padding-bottom",
    "padding-left",
    "width",
    "min-width",
    "max-width",
    "height",
    "min-height",
    "max-height"
  ],
  [
    "font",
    "font-family",
    "font-size",
    "font-weight",
    "font-style",
    "font-variant",
    "font-size-adjust",
    "font-stretch",
    "font-effect",
    "font-emphasize",
    "font-emphasize-position",
    "font-emphasize-style",
    "font-smooth",
    "line-height",
    "text-align",
    "-webkit-text-align-last",
    "-moz-text-align-last",
    "-ms-text-align-last",
    "text-align-last",
    "vertical-align",
    "white-space",
    "text-decoration",
    "text-emphasis",
    "text-emphasis-color",
    "text-emphasis-style",
    "text-emphasis-position",
    "text-indent",
    "-ms-text-justify",
    "text-justify",
    "letter-spacing",
    "word-spacing",
    "-ms-writing-mode",
    "text-outline",
    "text-transform",
    "text-wrap",
    "-ms-text-overflow",
    "text-overflow",
    "text-overflow-ellipsis",
    "text-overflow-mode",
    "-ms-word-wrap",
    "word-wrap",
    "-ms-word-break",
    "word-break"
  ],
  [
    "color",
    "background",
    "filter:progid:DXImageTransform.Microsoft.AlphaImageLoader",
    "background-color",
    "background-image",
    "background-repeat",
    "background-attachment",
    "background-position",
    "-ms-background-position-x",
    "background-position-x",
    "-ms-background-position-y",
    "background-position-y",
    "-webkit-background-clip",
    "-moz-background-clip",
    "background-clip",
    "background-origin",
    "-webkit-background-size",
    "-moz-background-size",
    "-o-background-size",
    "background-size"
  ],
  [
    "outline",
    "outline-width",
    "outline-style",
    "outline-color",
    "outline-offset",
    "opacity",
    "filter:progid:DXImageTransform.Microsoft.Alpha(Opacity",
    "-ms-filter:\\'progid:DXImageTransform.Microsoft.Alpha",
    "-ms-interpolation-mode",
    "-webkit-box-shadow",
    "-moz-box-shadow",
    "box-shadow",
    "filter:progid:DXImageTransform.Microsoft.gradient",
    "-ms-filter:\\'progid:DXImageTransform.Microsoft.gradient",
    "text-shadow"
  ],
  [
    "-webkit-transition",
    "-moz-transition",
    "-ms-transition",
    "-o-transition",
    "transition",
    "-webkit-transition-delay",
    "-moz-transition-delay",
    "-ms-transition-delay",
    "-o-transition-delay",
    "transition-delay",
    "-webkit-transition-timing-function",
    "-moz-transition-timing-function",
    "-ms-transition-timing-function",
    "-o-transition-timing-function",
    "transition-timing-function",
    "-webkit-transition-duration",
    "-moz-transition-duration",
    "-ms-transition-duration",
    "-o-transition-duration",
    "transition-duration",
    "-webkit-transition-property",
    "-moz-transition-property",
    "-ms-transition-property",
    "-o-transition-property",
    "transition-property",
    "-webkit-transform",
    "-moz-transform",
    "-ms-transform",
    "-o-transform",
    "transform",
    "-webkit-transform-origin",
    "-moz-transform-origin",
    "-ms-transform-origin",
    "-o-transform-origin",
    "transform-origin",
    "-webkit-animation",
    "-moz-animation",
    "-ms-animation",
    "-o-animation",
    "animation",
    "-webkit-animation-name",
    "-moz-animation-name",
    "-ms-animation-name",
    "-o-animation-name",
    "animation-name",
    "-webkit-animation-duration",
    "-moz-animation-duration",
    "-ms-animation-duration",
    "-o-animation-duration",
    "animation-duration",
    "-webkit-animation-play-state",
    "-moz-animation-play-state",
    "-ms-animation-play-state",
    "-o-animation-play-state",
    "animation-play-state",
    "-webkit-animation-timing-function",
    "-moz-animation-timing-function",
    "-ms-animation-timing-function",
    "-o-animation-timing-function",
    "animation-timing-function",
    "-webkit-animation-delay",
    "-moz-animation-delay",
    "-ms-animation-delay",
    "-o-animation-delay",
    "animation-delay",
    "-webkit-animation-iteration-count",
    "-moz-animation-iteration-count",
    "-ms-animation-iteration-count",
    "-o-animation-iteration-count",
    "animation-iteration-count",
    "-webkit-animation-direction",
    "-moz-animation-direction",
    "-ms-animation-direction",
    "-o-animation-direction",
    "animation-direction"
  ]
]
```

### 颜色

颜色16进制用小写字母；

颜色16进制尽量用简写。

```css
/* not good */
.element {
  color: #ABCDEF;
  background-color: #001122;
}

/* good */
.element {
  color: #abcdef;
  background-color: #012;
}
```

### 属性简写

属性简写需要你非常清楚属性值的正确顺序，而且在大多数情况下并不需要设置属性简写中包含的所有值，所以建议尽量分开声明会更加清晰；

`margin` 和 `padding` 相反，需要使用简写；

常见的属性简写包括：
- font
- background
- transition
- animation

```css
/* not good */
.element {
  transition: opacity 1s linear 2s;
}

/* good */
.element {
  transition-delay: 2s;
  transition-timing-function: linear;
  transition-duration: 1s;
  transition-property: opacity;
}
```

### 媒体查询

尽量将媒体查询的规则靠近与他们相关的规则，不要将他们一起放到一个独立的样式文件中，或者丢在文档的最底部，这样做只会让大家以后更容易忘记他们

```css
.element {
  ...
}

.element-avatar{
  ...
}

@media (min-width: 480px) {
  .element {
    ...
  }

  .element-avatar {
    ...
  }
}
```

### 杂项

不允许有空的规则；

元素选择器用小写字母；

去掉小数点前面的0；

去掉数字中不必要的小数点和末尾的0；

属性值`0`后面不要加单位；

同个属性不同前缀的写法需要在垂直方向保持对齐，具体参照右边的写法；

无前缀的标准属性应该写在有前缀的属性后面；

不要在同个规则里出现重复的属性，如果重复的属性是连续的则没关系；

不要在一个文件里出现两个相同的规则；

用 `border: 0`; 代替 `border: none;`；

选择器不要超过4层（在scss中如果超过4层应该考虑用嵌套的方式来写）；

发布的代码中不要有 `@import`；

尽量少用'*'选择器。

```css
/* not good */
.element {
}

/* not good */
LI {
  ...
}

/* good */
li {
  ...
}

/* not good */
.element {
  color: rgba(0, 0, 0, 0.5);
}

/* good */
.element {
  color: rgba(0, 0, 0, .5);
}

/* not good */
.element {
  width: 50.0px;
}

/* good */
.element {
  width: 50px;
}

/* not good */
.element {
  width: 0px;
}

/* good */
.element {
  width: 0;
}

/* not good */
.element {
  border-radius: 3px;
  -webkit-border-radius: 3px;
  -moz-border-radius: 3px;

  background: linear-gradient(to bottom, #fff 0, #eee 100%);
  background: -webkit-linear-gradient(top, #fff 0, #eee 100%);
  background: -moz-linear-gradient(top, #fff 0, #eee 100%);
}

/* good */
.element {
  -webkit-border-radius: 3px;
     -moz-border-radius: 3px;
          border-radius: 3px;

  background: -webkit-linear-gradient(top, #fff 0, #eee 100%);
  background:    -moz-linear-gradient(top, #fff 0, #eee 100%);
  background:         linear-gradient(to bottom, #fff 0, #eee 100%);
}

/* not good */
.element {
  color: rgb(0, 0, 0);
  width: 50px;
  color: rgba(0, 0, 0, .5);
}

/* good */
.element {
  color: rgb(0, 0, 0);
  color: rgba(0, 0, 0, .5);
}
```
