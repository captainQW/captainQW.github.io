title: Drupal8主题(theme)
date: 2015-11-19 16:35:16
categories: Drupal8
tags: [theme]
---

## Drupal8中的Twig

Drupal8中Twig替换了PHPTemplate作为默认的模板引擎，导致的结果之一就是模版的后缀由原来的.tpl.php变成了.html.twig

### 搜索Twig模版中的变量

在使用Twig模版的时候，大部分的变量都已注释的形式写在了模版中。然后，当我们新增了变量，我们就需要一个可以搜索到当前模版所有变量的方法。Twig提供了`dump`函数。

`dump`函数并不会打印所有的输出，除非开启了debug。

`dump`函数用于打印单个变量的信息或者模版中的所有变量。

#### 单个变量

如果你的模版中有一个title变量，下面在你的模版中dump出这个变量的内容：

```
{{ dump(title) }}
```

#### 模版中的所有变量

要打印出模版中的所有的变量，你可以：

```
{{ dump() }}
```

要打印出变量的键：

```
{{ dump(_context|keys) }}
```

PS: 这里的_context是一全局变量，指当前模版中所有的上下文和变量，比如通过theme函数传递的变量，preprocess函数处理的变量，或者调用set设置的变量。

**全局变量：**
- _context 当前模版中所有的上下文和变量
- _charset 只当前的编码方式

#### 慎用dump()

`dump()`函数虽然可以输出所有的变量，但是却带来了巨大的内存消耗，故你可以循环`_context`来查看所有的键：

```
<ol>
  {% for key, value in _context  %}
    <li>{{ key }}</li>
  {% endfor %}
</ol>
```

接下来再加上一些条件判断，就可以看到你需要的变量了。

## Drupal8主题

### 模版命名惯例

Drupal是按照一定的命名惯例来加载模版的，这就方便开发者可以覆盖原有的模版。当然，记得清除缓存。

#### HTML

HTML模版提供了包括head，title以及body标签等基础结构的html页面。

基础模版：html.html.twig(位于：core/modules/system/templates/html.html.twig)

下面是一些你可以覆盖的基础模版的例子：
1. html--internalviewpath.html.twig
2. html--node--id.html.twig
3. html.html.twig

#### Page

模版名称模式：page--[front|internal/path].html.twig

基础模版：page.html.twig(位于: core/modules/system/templates/page.html.twig)

下面是page模版的模版建议，举个例子“http://www.example.com/node/1/edit”：

1. page--node--edit.html.twig
2. page--node--1.html.twig
3. page--node.html.twig
4. page.html.twig

#### Regions

模版名称模式：region--[region].html.twig

基础模版：region.html.twig(位于: core/modules/system/templates/region.html.twig)

当一个页面的区域中有内容时，系统会从区块系统或者一个类似于`hook_page_build()`的方法中调用区域模版。


#### Blocks

模版名称模式：block--[module|-delta]].html.twig

基础模版：block.html.twig (位于: core/modules/block/templates/block.html.twig)

1. block--module--delta.html.twig
2. block--module.html.twig
3. block.html.twig

这里的module就是模块名，delta是模块给区块设置的内部id标识

#### Nodes

模版名称模式：node--[type|nodeid]--[viewmode].html.twig
基础模版：node.html.twig (位于: core/modules/node/templates/node.html.twig)

模版建议：

1. node--nodeid--viewmode.html.twig
2. node--nodeid.html.twig
3. node--type--viewmode.html.twig
4. node--type.html.twig
5. node--viewmode.html.twig
6. node.html.twig

#### Forums

模版名称模式：forums--[[container|topic]--forumID].html.twig

基础模版： forums.html.twig (位于: core/modules/forum/templates/forums.html.twig)

模版建议：

- 论坛容器：

  1. forums--containers--forumID.html.twig
  2. forums--forumID.html.twig
  3. forums--containers.html.twig
  4. forums.html.twig

- 论坛主题内容

  1. forums--topics--forumID.html.twig
  2. forums--forumID.html.twig
  3. forums--topics.html.twig
  4. forums.html.twig

### PHPTemplate与Twig主题范例的比较

接下来，看下PHPTemplate与Twig的区别。

#### 关于Twig

Twig是基于php的编译模版语言。当你的页面需要渲染显示的时候，Twig引擎会找到对应的模版并把它编译成php模版，存放在sites/default/files/php_storage/目录下。

##### 1.Docblock

PHPTemplate：

```
<?php 
/** 
 * @file
 * File description
 */
?>
```

Twig：

```
{# 
/** 
 * @file
 * File description
 */
#}
```

##### 2.文件和函数名

PHPTemplate文件：node--article.tpl.php
Twig文件：node--article.html.twig

PHPTemplate函数：theme_node_links()
Twig文件：node-links.html.twig

##### 3.变量

**输出一个变量**

PHPTemplate: 

```
<div class="content"><?php print $content; ?></div>
```

Twig:

```
<div class="content">{{ content }}</div>
```

**输出一个哈希键值**

PHPTemplate:

```
<?php print $item['#item']['alt']; ?>
```

Twig:

```
{{ item['#item'].alt }}
```

**变量赋值**

PHPTemplate:

```
<?php $custom_var = $content->comments; ?>
```

Twig:

```
{% set custom_var = content.comments %}
```

**数组初始化**

PHPTemplate:

```
<?php
  $args = array('!author' => $author, '!date' => $created);
?>
```

Twig:

```
{% set args = {'!author': author, '!date': created} %}
```

##### 4.条件判断

PHPTemplate:

```
<?php if ($content->comments): endif; ?>
```

Twig:

```
{% if content.comments %} {% endif %}
```

PHPTemplate:

```
<?php if (!empty($content->comments)): endif; ?>
```

Twig:

```
{% if content.comments is not empty %} {% endif %}
```

PHPTemplate:

```
<?php if (isset($content->comments)): endif; ?>
```

Twig:

```
{% if content.comments is defined %} {% endif %}
```

PHPTemplate:

```
<?php if ($count > 0): endif; ?>
```

Twig:

```
{% if count > 0 %} {% endif %}
```

##### 5.控制结构

PHPTemplate:

```
<?php foreach ($users as $user) {} ?>
```

Twig:

```
{% for user in users %} {% endfor %}
```

##### 6.过滤

**check_plain：**

PHPTemplate:

```
<?php print check_plain($title); ?>
```

Twig:

```
{{ title|striptags }}
```

**Translate：**

PHPTemplate:

```
<?php print t('Home'); ?>
```

Twig:

```
{{ 'Home'|t }}
```

**Translate with substitutions:**

PHPTemplate:

```
<?php
  print t('Welcome, @username', array('@username' => $user->name));
?>
```

Twig:

```
{{ 'Welcome, @username'|t({ '@username': user.name }) }}
```

Drupal8 Twig(采用[trans](https://drupal.org/node/2047135)标签扩展)

```
{% set username = user.name %}
{% trans %}
  Welcome, {{ username }}
{% endtrans %}
```

**implode:**

PHPTemplate:

```
<?php echo implode(', ', $usernames); ?>
```

Twig:

```
{{ usernames | join(', ') }}
```

PHPTemplate的例子中$usernames需要是一个字符串数组。原生的Twig也需要$usernames是一个字符串数组。但是Drupal8中的twig还可以是一个可以渲染的数组对象。这就是Drupal8中的twig与原生twig的本质区别。Drupal8中的twig可以“打印”出纯文本和可渲染的数组。

举个例子：

```
{% set numbers = [{'#markup': 'One'}, {'#markup':'Two'}, {'#markup':'Three'}] %}
{{ numbers }}
```

在上面的例子中，每一个数据项都是以逗号相隔的。但是，输出的结果却是：

```html
OneTwoThree
```

**Escape：**

PHPTemplate:

```
<?php echo check_plain($title); ?>
```

原生Twig:

```
{{ title|e }}
```

Drupal 8 Twig2:

```
{{ title }}
```

##### 7.空白

```
<div class="body">
  {{- block.content -}}
</div>
```

类似于

```
<div class="body">{{ block.content }}</div>
```

### debug twig模版

twig模版引擎提供了一个debug工具。

#### 启用debugging

在sites/default/services.yml启用Twig Debugging。

设置debug变量为true

```
parameters:
  twig.config:
    debug: true 
```

#### 自动加载编译的模版

出于性能考虑，Twig模版被编译成一个PHP类存放在磁盘上，但这意味着默认情况下，你的模版不会在你做出更改时发生刷新。为了使Twig模版可以自动刷新，启用`services.yml`中的deubg选项。欲知详情，参见：[https://drupal.org/node/1903374](https://drupal.org/node/1903374)

**打印所有的变量**

```
{# 打印所有的变量 #}
{{ dump() }}
```

```
{# 打印单个变量 #}
{{ dump(var) }}
```

### Filters---修改Twig模版中的变量

Twig中的Filters用于修改变量。Filters和变量之间用管道符号(|)分隔，还可能带点可选的参数。多个filters可以链式的连在一起，前一个的输出作为下一个的输入。

例如：

```
{{ ponies|safe_join(", ")|lower}}
```

Drupal的Twig模版中使用的filter由所有的Twig引擎自带的filter和drupal中的几个特殊的filters组成。

#### Twig filters

详见：[twig filter列表](http://twig.sensiolabs.org/doc/filters/index.html)

#### Drupal自身的filters

Drupal中的filters是在`Drupal\Core\Template\TwigExtension::getFilters().`中声明的。

##### 翻译filters

**t**

t filter调用Drupal的t函数翻译给定的字符串，可用于任何字符串。

例如：

```
<a href="{{ url('<front>') }}" title="{{ 'Home'|t }}" rel="home" class="site-logo"></a>
```

placeholder输出安全的html并使用`drupal_placeholder()`进行格式化输出成强调文本。

例如：

```
{% trans %}Submitted on {{ date|placeholder }}{% endtrans %}
```

```
{{ var1|t }}
{{ var1|placeholder }}
{% trans %}{{ var1 }}{% endtrans %}
```

尽可能避免使用|raw filter，特别是你正在输出一些用户输入的数据的时候。详见[https://www.drupal.org/node/2296163](https://www.drupal.org/node/2296163)：

```
{{ var1|raw }}
```

##### 额外的filters

**drupal_escape**

drupal_escape用于将一个字符串进行安全处理后进行插入操作并输出显示，它替代了twig默认的escape filter。参见[twig_drupal_escape_filter](https://api.drupal.org/api/drupal/core%21themes%21engines%21twig%21twig.engine/function/twig_drupal_escape_filter/8)

**safe_join**

safe_join将几个字符串用给定的分隔符连接在一块儿。

例如：

```
{{ items|safe_join(', ') }}
```

**without**

without会创建出当前这个可以渲染的数组的拷贝，但是会把传入的参数当作键，删除键关联的所有子元素。但是，当前这个数组不受影响，依旧可以打印出子元素。详见：[twig_without](https://api.drupal.org/api/drupal/core%21themes%21engines%21twig%21twig.engine/function/twig_without/8)

例如：

```
{# 打印content变量，除了content.links #}
{{ content|without('links') }}
```

**clean_class**

clean_class filter会处理出一个安全的可用的html类名。详见：[Html::getClass()](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Component%21Utility%21Html.php/function/Html%3A%3AgetClass/8)

**clean_id**

clean_id filter会处理出一个安全的可用的html id。详见：[Html:getID()](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Component%21Utility%21Html.php/function/Html%3A%3AgetId/8)

**format_date**

format_date filter格式化日期字符串，详见：[DateFormatter::format()](https://api.drupal.org/api/drupal/core!lib!Drupal!Core!Datetime!DateFormatter.php/function/DateFormatter%3A%3Aformat/8)

### Functions---Twig模版中的函数

**url()**

**path()**

**url_from_path()**

**link($text, $url, $attributes)**

```
{{ link(item.title, item.url, { 'class':['foo', 'bar', 'baz']} ) }}
```

**file_url($uri)**

路径参数需要是相对网站根目录的相对路径，将会返回这个文件的绝对路径。

**attach_library()**

### Twig最佳实践---预处理函数和模版

为了使Drupal8的主题性能最大化，以及更加的定制化，请遵照如下的最佳实践方式：

#### 使用预处理函数返回可渲染数组

总是通过预处理函数返回可渲染数组，而不是调用`theme()`或者`drupal_render()`函数。

Twig会自动渲染，故没有必要调用`theme()`或者`drupal_render()`函数。此外，为了比直接输出已经渲染的html代码更具定制化，渲染数组应该传递给模版。

从预处理函数中删除`theme()`函数：

```
<?php
// 之前，直接把渲染的html传递给模版
$variables['table'] = theme('table', array('header' => $header, 'rows' => $rows));

// 之后，把渲染数组传递给模版
$variables['table'] = array(
  '#theme' => 'table',
  '#header' => $header,
  '#rows' => $rows,
);
?>
```

从预处理函数中删除`drupal_render()`函数就是不再调用它：

```
<?php
// 之前
$variables['teaser'] = drupal_render($node_teaser);

// 之后
$variables['teaser'] = $node_teaser;
?>
```

通常情况下，`drupal_render()`函数是在添加表格数据的时候调用。

```
<?php
// 之前
$row[] = drupal_render($display['title']);

// 之后  
$row[]['data'] = $display['title'];
?>
```

#### 在模版中调用filters和工具函数

举个例子：

**之前的做法：**

预处理函数中：

```
<?php
$variables['no_content_text'] = t('You have not created any content types yet. Go to the <a href="@create-content">content type creation page</a> to add a new content type.', array('@create-content' => url('admin/structure/types/add')));
?>
```

模版中：

```
<p>{{ no_content_text }}</p>
```

**现在的做法：**

模版中：

```
<p>{{ 'You have not created any content types yet. Go to the <a href="@create-content">content type creation page</a> to add a new content type.'|t({'@create-content': url('admin/structure/types/add')}) }}</p>
```

#### 显示/隐藏 & 删除drupal_render_children和element_children

在原始的模版中，如果调用了`hide()`函数，而且`drupal_render_children`函数用于输出”剩下的“数据，我们将需要在预处理的时候把这些数据拆分成单个变量。

**之前(PHPTemplate模版)：**

```
<?php
  hide($form['advanced']);
  hide($form['actions']);
?>
<div class="layout-node-form clearfix">
  <div class="layout-region layout-region-node-main">
    <?php print drupal_render_children($form); ?>
  </div>
  <div class="layout-region layout-region-node-secondary">
    <?php print render($form['advanced']); ?>
  </div>
  <div class="layout-region layout-region-node-footer">
    <?php print render($form['actions']); ?>
  </div>
</div> 
```

**之后(预处理)：**

```
<?php
function template_preprocess_node_edit_form(&$variables) {
  $form = $variables['form'];
  
  // @todo Update this once drupal.org/node/1920886 is resolved.
  $variables['advanced'] = $form['advanced'];
  $variables['actions'] = $form['actions'];
  unset($form['advanced'], $form['actions']);
  $variables['form'] = drupal_render_children($form);
}
?>
```

**之后(Twig模版)：**

```
<div class="layout-node-form clearfix">
  <div class="layout-region layout-region-node-main">
    {{ form|without('advanced', 'actions') }}
  </div>
  <div class="layout-region layout-region-node-secondary">
    {{ form.advanced }}
  </div>
  <div class="layout-region layout-region-node-footer">
    {{ form.actions }}
  </div>
</div>
```

## 创建Drupal8的主题

### 以一个.info.yml文件开始定义一个主题

创建drupal8主题的第一步，就是创建一个THEMENAME.info.yml文件。有一点很重要，.info.yml文件中的“type”键需要设置成“theme”。

#### 创建一个.info.yml文件

在主题文件夹的根目录创建.info.yml文件。文件夹的名字应该保持与.info.yml文件名相同。所以，如果你的主题命名为“Fluffiness”，那么文件夹的名字就是“fluffiness”，而.info.yml文件的名字是“fluffiness/fluffiness.info.yml”。

例如：

```
name: Fluffiness
type: theme
description: A cuddly theme that offers extra fluffiness.
core: 8.x
libraries:
  - fluffiness/global-styling
stylesheets-remove:
  - '@classy/css/components/tabs.css'
  - core/assets/vendor/normalize-css/normalize.css
regions:
  header: Header
  content: Content
  sidebar_first: 'Sidebar first'
  footer: Footer
```

在你的drupal站点中，核心提供的主题包含着很多.info.yml文件可以供你查阅实例。

#### 配置文件中的键释义

- name: Fluffiness  必选。显示在站点外观页面的，可读主题名
- description: An extra cuddly Drupal theme available in grey and blue  必选，主题描述，也会显示在站点外观管理页面
- type: theme  必选项，扩展类型，比如：module, theme 或者 profile。对于主题，这里设置成“theme”
- base theme: classy  可以从另一个主题继承其资源的主题
- core: 8.x  必选，当前主题兼容的Drupal版本
- version: 8.x-1.0
- screenshot: fluffiness.png
- libraries:
    - fluffiness/global-styling

    当主题启用时，为所有的页面加载资源库文件，包括css和js

    例如：

    ```
      global-styling:
        version: 1.x
        css:
          theme:
            css/style.css: {}
            css/print.css: { media: print }
    ```

    当html解析时，就会是：

    ```
      <link rel="stylesheet" href="css/style.css" media="all" />
      <link rel="stylesheet" href="css/print.css" media="print" />
    ```

- stylesheets-remove:

  ```
     - core/assets/vendor/normalize-css/normalize.css # 1
     - '@classy/css/components/tabs.css' # 2
  ```

  **注意：**
  1. stylesheets-remove键定义需要删除的样式，需要设置成完全路径。
  2. 为了方式Drupal核心的资源（例如，jQuery UI的css文件）被删除，请设置为完全路径。此外，为了避免当前这个文件属于某个模块或者主题提供的库的，可以使用token。注意，如果使用token，由于@会被当成YAML，请记得使用引号。

- regions:

  ```
     header: Header
     content: Content # required!
     sidebar_first: 'Sidebar first'
     footer: Footer
  ```

### 主题文件夹的结构

主题就是定了视图层的许多文件的集合。.info.yml文件是必须的，下面会列出一个典型的主题文件夹中所包含的文件和文件夹。

#### 主题的位置

主题必须放置在Drupal安装根目录的“themes”文件夹中。注意，核心的主题，例如：Bartik和Seven位于core/themes文件夹下。

一个好的建议是，创建一个名为“contrib”的文件夹用于存放下载的贡献主题，而“custom”文件夹用于存放自定义的主题。

主体名称必须小写。

#### Drupal的目录结构：

```
|-core
|  |-modules
|  |-themes
|  |  |-bartik
|  |  |-seven
..
|-modules
|-themes
|  |-contrib
|  |  |-zen
|  |  |-basic
|  |  |-bluemarine
|  |-custom
|  |  |-fluffiness
```

#### 主题文件夹结构，假设主题名称是：fluffiness

```
|-fluffiness.breakpoints.yml
|-fluffiness.info.yml
|-fluffiness.libraries.yml
|-fluffiness.theme
|-config
|  |-install
|  |  |-fluffiness.settings.yml
|  |-schema
|  |  |-fluffiness.schema.yml
|-css
|  |-style.css
|-js
|  |-fluffiness.js
|-images
|  |-buttons.png
|-logo.png
|-screenshot.png
|-templates
|  |-maintenance-page.html.twig
|  |-node.html.twig
```

下面，介绍一下主题文件夹中的通用文件。

#### *.info.yml

主题必须包含*.info.yml文件，此文件定义了meta数据，样式，还有区块和区域信息。

#### *.libraries.yml

*.libraries.yml文件用于主题启用时需要加载的定义js和css库，详情[点击这里](https://www.drupal.org/theme-guide/8/adding-javascript)

#### *.breakpoints.yml

响应式设计相关的设置，详见：[*.breakpoints.yml设置](https://www.drupal.org/documentation/modules/breakpoint)

#### *.theme

*.theme文件是一个php文件，包含了所有的逻辑处理代码和输出前的预处理。

#### css/

一个好的建议就是将css文件存放在“css”自目录下。

#### js/

存放着主题需要的js文件。

#### images/

存放主题需要的图片

#### screenshot.png

主题的预览图

#### templates/

#### 核心主题Bartik的目录结构：

```
|-bartik.breakpoints.yml
|-bartik.info.yml
|-bartik.libraries.yml
|-bartik.theme
|-color
|  |-color.inc
|  |-preview.css
|  |-preview.html
|  |-preview.js
|-config
|  |-schema
|  |  |-bartik.schema.yml
|-css
|  |-base
|  |  |-elements.css
|  |-components
|  |  |-block.css
|  |  |-book.css
|  |  |-breadcrumb.css
...
|  |-colors.css
|  |-layout.css
|  |-maintenance-page.css
|  |-print.css
|-images
|  |-add.png
|  |-required.svg
|  |-tabs-border.png
|-logo.svg
|-screenshot.png
|-templates
|  |-block--search-form-block.html.twig
|  |-block--system-branding-block.html.twig
|  |-block--system-menu-block.html.twig
|  |-block.html.twig
|  |-comment.html.twig
|  |-field--taxonomy-term-reference.html.twig
|  |-maintenance-page.html.twig
|  |-node.html.twig
|  |-page.html.twig
|  |-status-messages.html.twig
```

### 为主题添加区域

为主题添加区域需要两步：

- 在主题文件.info.yml中添加区域的meta-data
- 编辑page.html.twig文件打印新的区域

#### info文件中添加区域

区域定义成ragions键的子元素：

```
# Regions
regions:
  header: 'Header'
  content: 'Content'
  footer: 'Footer'
```

Region 键应是个字符串，可以包含下划线“_”。

#### 模版中添加区域

为了显示区域中的内容，你需要确定新的区域也添加到了page.html.twig模版中。

例如：

```
header: 'Header'
```

模版中输出：

```
{{ page.header }}
```

此外，你可以把它们当成是Twig模版的普通变量，可以再任何需要的地方打印输出。

#### 默认区域

可以在[page.html.twig模版文档](https://api.drupal.org/api/drupal/core%21modules%21system%21templates%21page.html.twig/8)中查阅默认的区域

1. page.header
2. page.primary_menu
3. page.secondary_menu
4. page.highlighted
5. page.help
6. page.content
7. page.sidebar_first
8. page.sidebar_second
9. page.footer
10. page.breadcrumb

### Drupal8主题中添加样式和js

#### 与Drupal7主题的区别

**有四个比较重要的区别：**

1. 去掉了THEME.info文件，取而代之的是THEME.info.yml文件
2. THEME.info.yml文件中的stylesheets属性不复存在，改为库的方式引入
3. THEME.info.yml文件中的scripts属性不复存在，改为库的方式引入
4. 每一个页面的js只在需要时才加载，默认情况下，Drupal对于匿名用户不会需要js。这意味着，jquery不会再在每个页面自动加载了。因此，如果你的主题需要jquery或者其它的js，你需要通过“声明依赖关系”来告诉Drupal这个js需要加载

** 加载css/js的步骤：**

1. 将css或者js保存到一个文件
2. 定义一个包含js和css文件的“library”
3. 在钩子中，把这个库追加给渲染数组

#### 定义一个库

在主题文件夹中创建*.libraries.yml文件来添加库（假设你的主题名称是`fluffiness`，那么文件名就是` fluffiness.libraries.yml`）。每一个库，都是css和js文件的详情列表：

```
cuddly-slider:
  version: 1.x
  css:
    theme:
      css/cuddly-slider.css: {}
  js:
    js/cuddly-slider.js: {}
```

这个例子中，假设`cuddly-slider.js`文件位于主题的js子目录中。

然后，需要记住的是，默认情况下，Drupal8不会在每一个页面加载jquery；只在需要时加载。因此，我们需要自己声明，主题的`cuddly-slider`库依赖`jQuery`。

因此，为了保证上例的`js/cuddly-slider.js`可用，我们需要更新下：

```
cuddly-slider:
  version: 1.x
  css:
    theme:
      css/cuddly-slider.css: {}
  js:
    js/cuddly-slider.js: {}
  dependencies:
    - core/jquery
```

当然，库也可以只包含css，或者只包含js。大部分的主题可能会有一个`global-styling`库，用于全局加载样式文件：

```
global-styling:
  version: 1.x
  css:
    theme:
      css/layout.css: {}
      css/style.css: {}
      css/colors.css: {}
      css/print.css: { media: print }
```

而且，正如你所想，库中定义的css的顺序就是将来加载的顺序。

在Drupal7中，你可以把媒体查询属性(screen, print, all)作为`stylesheets`属性的子键，而现在你可以把它当作值定义，例如：

```
css/print.css: { media: print }
```

现在默认情况下，所有的js文件都在页脚加载。

```
js-header:
  header: true
  js:
    header.js: {}

js-footer:
  js:
    footer.js: {}
```

你得设置`header`属性为true。

#### 覆盖扩展库

那些定义在`*.libraries.yml`文件中的库可以通过主题的`*.info.yml`文件中的`libraries-override`和`libraries-extend`项进行覆写和扩展。

**libraries-override**

`libraries-override`是定义在库中，主题用来操作css/js的一种方式，包括删除、替换或者删除css/js。

```
libraries-override:
  # Replace an entire library.
  core/drupal.collapse: mytheme/collapse
  
  # Replace an asset with another.
  subtheme/library:
    css:
      theme:
        css/layout.css: css/my-layout.css
  
  # Remove an asset.
  drupal/dialog:
    css:
      theme:
        dialog.theme.css: false
  
  # Remove an entire library.
  core/modernizr: false
```

**libraries-extend**

*libraries-extend*为主题提供了一种可以修改库的方式。比如，添加额外的主题依赖的库。

```
# Extend drupal.user: add assets from classy's user libraries.
libraries-extend:
  core/drupal.user: 
    - classy/user1
    - classy/user2
```

#### 追加库到页面中

**在Twig模版中追加库**

通过调用twig的方法，`attach_library()`可以在twig模版中追加库，例如：

```
{{ attach_library('fluffiness/cuddly-slider') }}
<div>Some fluffy markup {{ message }}</div>
```

**追加给所有的页面**

为了所有页面都加载这个库，需要在主题的`*.info.yml`文件中，`libraries`键的下面声明：

```
name: Example
type: theme
core: 8.x
libraries:
  - fluffiness/cuddly-slider
```

#### 为一组页面添加库

在大部分情况下，你可能不希望所有的页面加载你的库，而是部分页面加载。举个例子来说，你可能只想在某个特定的区块中加载，或者用户访问某个节点类型时加载。

主题可以通过实现`THEME_preprocess_HOOK()`方法达到这个效果。

例如，你想要在维护页面追加你的库，“HOOK”就是“maintenance_page”，然后你的方法就是：

```
<?php
function fluffiness_preprocess_maintenance_page(&$variables) {
  $variables['#attached']['library'][] = 'fluffiness/cuddly-slider';
}
?>
```

你也可以为主题的其它hook做同样的事情，当然你的函数中也可以包含逻辑处理。

**重要提示**：最通用的用法是根据当前的路由来加载库：

```
<?php
function fluffiness_preprocess_page(&$variables) {
  $variables['#cache']['contexts'][] = 'route';
  if (\Drupal::routeMatch()->getRouteName() === 'entity.node.preview') {
    $variables['#attached']['library'][] = 'fluffiness/node-preview';
  }
}
?>
```

#### 添加可定制的js

在某些情况下，你可能想要根据某些php计算信息来加载js。

在这种情况下，像之前一样，创建一个js文件并追加进来，再通过`drupalSettings`(替换Drupal7中的`Drupal.settings`)追加一些js设置项并由追加的js文件读取这些设置项。然后，为了使`drupalSettings`在我们的js文件中生效，我们得像引入jQuery一样操作：我们得在依赖中定义。

就像这样：

```
cuddly-slider:
  version: 1.x
  js:
    js/cuddly-slider.js: {}
  dependencies:
    - core/jquery
    - core/drupalSettings
```

以及

```
<?php
function fluffiness_page_attachments_alter(&$page) {
  $page['#attached']['library'][] = 'fluffiness/cuddly-slider';
  $page['#attached']['drupalSettings']['fluffiness']['cuddlySlider']['foo'] = 'bar';
}
?>
```

这样`cuddly-slider.js`就可以访问`drupalSettings.fluffiness.cuddlySlider.foo`了(这里 === 'bar')。

### 在模版中使用attributes

attributes是一个在每个twig模版中都有效的对象。attributes用来存储所有的父容器的相关属性，并提供有用的方法给开发者处理这些数据。

#### Attribute的方法

**attributes.addClass()**

单个类：

```
<div{{attributes.addClass('my-class')}}></div>
```

多个类：

```
{%
  set classes = [
    'red',
    'green',
  ]
%}
<div{{ attributes.addClass(classes) }}></div>
```

将会输出：

```
<div class="red green"></div>.
```

*html标签与twig语法之间不能有任何空格！*

**attributes.removeClass()**

```
{#classes = [ 'red', 'green', 'blue' ] #}

<div{{ attributes.addClass(classes).removeClass('green') }}></div>
```

将会输出：

```
<div class="red blue"></div>
```

**attributes.setAttribute($attribute, $value)**

```
<div{{ attributes.setAttribute('id', 'myID') }}></div>
```

将会输出：

```
<div id="myID"></div>
```

**attributes.removeAttribute($attribute, $value)**

```
<div{{ attributes.removeAttribute('id') }}></div>
```

**attributes.hasClass($class)**

```
{% if attribute.hasClass(‘myClass') %}
 {# do stuff #}
{% endif %}
```
#### 其它有用的代码片段

**链式**

```
{% set classes = ['red', 'green', 'blue'] %}
{% set my_id = 'specific-id' %}
{% set image_src = 'https://www.drupal.org/files/powered-blue-135x42.png' %}
```

```
<img{{ attributes.addClass(classes).removeClass('green').setAttribute('id', my_id).setAttribute('src', image_src) }}>
```

将会输出：

```
<img id="specific-id" class="red blue" src="https://www.drupal.org/files/powered-blue-135x42.png">
```

**使用without filter**

```
<div class=”myclass {{ attributes.class }}”{{attributes|without(‘class’) }}></div>
```

### Drupal6, 7, 8主题区别

这里列出的是比较重要的Drupal8中的主题变化

1. Drupal8默认输出html5
2. 除了jQuery2.x之外，Drupal8还引入更多的前端框架，例如：[Modernizr](http://modernizr.com/)，[Underscore.js](http://underscorejs.org/)，以及[Backbone.js](http://backbonejs.org/)。
3. Drupal8引入了Twig，替换了原来的默认模板引擎PHPTemplate。
4. Drupal8处于性能考虑，默认启用了类似css、js合并之类的默认特性。
5. 在drupal6和drupal7中，可以调用`drupal_add_css()`和`drupal_add_js()`函数添加css或者js，现在替换成了在渲染数组中追加`#attached`属性调用库的方法。
6. Drupal8不再支持IE6、7、8，启用jQuery2.0，支持现代HTML5/CSS3浏览器。
7. Drupal8不支持那些不支持SVG的浏览器(包括IE8和安卓浏览器2.3)。
8. Drupal8的css中，减少了id的使用。
9. Drupal8的css结构(文件结构)基于[SMACSS](https://smacss.com/)和[BEM](http://bem.info/)
10. Drupal8将预处理函数中的css类移到了Twig模版中。