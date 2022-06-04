title: Drupal8模板引擎-twig
date: 2015-11-18 16:35:16
categories: Drupal8
tags: [twig]
---

## 一、简介

Twig是一个高效、安全并可扩展的PHP模板引擎。

如果你具有Smarty, Django或者Jinja这养的基于文本的模板语言的经验，那么你对Twig会感觉一见如故。在遵循PHP理念的基础之上，提供了具备强大功能的模板环境，不论是对开发者还是设计者来说，这一引擎都具有良好的可用性。

关键特性：

- 快速：Twig把模板编译成为优化后的PHP代码。相对普通PHP代码来说，其额外开销非常轻微。
- 安全：Twig会使用一个沙箱模式来运行不信任的模板代码。这使得Twig可以在用户可以修改模板设计的应用中工作良好。
- 弹性：Twig试用了弹性的词法和语法分析器。开发者可以定义自己的标记和过滤，并创建自己的DSL。

### 需求
Twig需要的最小运行环境为PHP **5.2.4**

### 安装
推荐使用Composer安装Twig：

```bash
composer require "twig/twig:~1.0"
```


### 基础API用法

本节对Twig PHP API做一个简单介绍。

```php
require_once '/path/to/vendor/autoload.php';

$loader = new Twig_Loader_Array(
    'index' => 'Hello {{ name }}!',
);
$twig = new Twig_Environment($loader);

echo $twig->render('index', array('name' => 'Fabien'));
```

Twig使用一个加载器(Twig_Loader_Array)来定位模板文件，使用一个环境(Twig_Environment)来存储配置，render()方法的第一个参数指定了模板，后面的参数则为渲染提供了变量。 模板一般是保存在文件系统中，所以Twig提供了一个文件系统加载器：

```php
$loader = new Twig_Loader_Filesystem('/path/to/templates');
$twig = new Twig_Environment($loader, array(
    'cache' => '/path/to/compilation_cache',
));
echo $twig->render('index.html', array('name' => 'Fabien'));
```

如果没有使用 Composer，可以使用 Twig 的内置加载器：

```php
require_once '/path/to/lib/Twig/Autoloader.php';
Twig_Autoloader::register();
```

## 二、模板设计者的Twig

本文描述了模板引擎中涉及到的语法和语义，对Twig模板的设计会很有帮助。

### 概要

模板是一个简单的文本文件。它能够生成任何文本格式（HTML, XML, CSV, LaTeX等）。他没有固定的扩展名，html、xml都没关系。

模板中包含变量和表达式在模板被处理时会被替换为真实的值，tags则对模板的逻辑进行控制。

下面用一个小例子展示一些基础内容，当然，后面会做进一步的深入。

```php
<!DOCTYPE html>
<html>
  <head>
      <title>My Webpage</title>
  </head>
  <body>
    <ul id="navigation">
    {% raw %}
    {% for item in navigation %}
        <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
    {% endfor %}
    </ul>
    {% endraw %}
    <h1>My Webpage</h1>
    {% raw %}
    {{ a_variable }}
    {% endraw %}
  </body>
</html>
```

{% raw %}
上面代码中有两种分隔符：{% ... %}和{{ ... }}。第一种用于执行for循环之类的语句，第二种用于输出表达式结果。
{% endraw %}

### 变量

应用会把变量传递给模板进行处理。变量自身也可以拥有属性或元素供外部访问。变量的具体形态取决于提供变量的应用。

可以利用句点（.）来访问变量的属性（相当于PHP对象的属性或方法，或者PHP数组的成员），或者也可以使用下标语法（[]）：

```php
{% raw %}
{{ foo.bar }}
{{ foo['bar'] }}
{% endraw %}
```

当属性名字中包含了特别字符（例如“-”会被当做减号处理）的时候，可以使用attribute函数来访问变量的属性：

```php
{# 等价于不能工作的：foo.data-foo #}
{% raw %}
{{ attribute(foo, 'data-foo') }}
{% endraw %}
```

*大括号是打印指令，而不是变量的组成部分。当在tag中访问变量的时候，不要在Tag中访问变量的时候使用这种语法。*

在strict_variables选项为false的时候，如果变量或属性不存在，会返回null；而如果该选项设置为true，这种情况就会抛出错误

### 实现

foo.bar在PHP这一层次做了如下事情：

- 检查foo是不是数组，bar是不是其中的一个有效元素；
- 如果不是，且foo是一个对象，检查bar是不是有效的属性；
- 如果不是，且foo是一个对象，检查bar是不是有效的方法（即使bar是构造器——所以请使用__construct()）；
- 如果不是，且foo是一个对象，检查getBar是不是有效的方法；
- 如果不是，且foo是一个对象，检查isBar是不是有效的方法；
- 如果都不是，返回null。

另一方面，foo['bar']只对PHP数组工作 - 检查foo是不是一个数组，bar是不是一个有效元素； - 如果不是，返回null。

**如果要访问变量的动态属性，应该使用attribute()函数。**

### 全局变量

模板中随时可以使用如下的变量：

- _self：当前的模板；
- _context：当前上下文
- _charset：当前编码

### 设置变量

可以在代码中为变量赋值。赋值过程使用settag完成：

```php
{% raw %}
{% set foo = 'foo' %}
{% set foo = [1, 2] %}
{% set foo = {'foo': 'bar'} %}
{% endraw %}
```

### Filters

变量可以用filters进行修改。Filters和变量之间用管道符（|）进行分隔，可以用括号来包含参数。多个Filter可能进行链式调用——前一个Filter的输出可作为下一个Filter的输入。

下面的例子清理所有的HTML标记，并做标题大写处理：

```php
{% raw %}
{{ name|striptags|title }}
{% endraw %}
```

Filter可以用括号的形式接收参数。这个例子用逗号连接一个列表：

```php
{% raw %}
{{ list|join(', ') }}
{% endraw %}
```

要对一段代码使用Filter，可以用filter tag操作：

```php
{% raw %}
{% filter upper %}
  This text becomes uppercase
{% endfilter %}
{% endraw %}
```

### 函数

调用函数可以生成内容。调用函数的方式是：函数名称加括号，括号中可能包含参数。

例如range函数返回一个递增的整数列表：

```php
{% raw %}
{% for i in range(0, 3) %}
  {{ i }},
{% endfor %}
{% endraw %}
```

前往[functions](http://twig.sensiolabs.org/doc/functions/index.html)一节可以学习更多内置函数的内容。

### 具名参数

1.12版中新增

```php
{% raw %}
{% for i in range(low=1, high=10, step=2) %}
  {{ i }},
{% endfor %}
{% endraw %}
```

有了具名参数，传递给模板的参数可以更加明确，让模板更加易用。

```php
{% raw %}
{{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}
{# 相比： #}
{{ data|convert_encoding(from='iso-2022-jp', to='UTF-8') }}
{% endraw %}
```

具名参数还能在不传递某些参数时候直接跳过：

```php
{% raw %}
{# the first argument is the date format, which defaults to the global date format if null is passed #}
{{ "now"|date(null, "Europe/Paris") }}

{# or skip the format value by using a named argument for the time zone #}
{{ "now"|date(timezone="Europe/Paris") }}
{% endraw %}
```

还可以用顺序参数和具名参数夹杂的方式进行调用，这里要保证顺序参数必须比具名参数先出现：

```php
{% raw %}
{{ "now"|date('d/m/Y H:i', timezone="Europe/Paris") }}
{% endraw %}
```

*每个函数和Filter的文档中，如果该函数或者Filter支持具名参数，都会有一节列出所有的参数名称。*

### 控制结构
{% raw %}
控制结构指的是所有能够控制程序流程的元素——条件判断（也就是if/elseif/else）、for循环，以及语句块等。控制结构用{% ... %}包裹
{% endraw %}

例如，要显示一个users变量中所有的列表成员，使用for：

```php
<h1>Members</h1>
<ul>
  {% raw %}
  {% for user in users %}
      <li>{{ user.username|e }}</li>
  {% endfor %}
  {% endraw %}
</ul>
```

可以用if进行条件判断：

```php
{% if users|length > 0 %}
  <ul>
    {% raw %}
    {% for user in users %}
      <li>{{ user.username|e }}</li>
    {% endfor %}
    {% endraw %}
  </ul>
{% endif %}
```

[Tags](http://twig.sensiolabs.org/doc/tags/index.html)页中描述了各种内置Tag。

### 注释

在模板中进行注释，使用注释语法`{# ... #}：

```php
{% raw %}
{# 注：禁用这一块东西，不再需要了
{% for user in users %}
  ...
{% endfor %}
{% endraw %}
#}
```

### 包含其他模板
include标签用于包含其它模版，并把该模版的渲染结果返回到当前页面：

```php
{% raw %}
{% include 'sidebar.html' %}
{% endraw %}
```

缺省状况下，当前的上下文会被传递给被包含的模板。

传递给被包含模板中的上下文包含当前模板中定义的变量：

```php
{% raw %}
{% for box in boxes %}
  {% include "render_box.html" %}
{% endfor %}
{% endraw %}
```

这个例子中，render_box.html可以访问box变量。

模板文件名的选取，取决于模板加载器。比如Twig_Loader_Filesystem允许通过文件名访问其他的模板。可以使用子目录来访问其他模板：

```php
{% raw %}
{% include "sections/articles/sidebar.html" %}
{% endraw %}
```

### 模版继承

Twig最强大的能力就是继承。这一个能力可以让你建立一个包含所有基础元素的基础模板，并定义可以被子模板覆盖的block。

这部分内容并不像听起来那么复杂。下面的例子能让你更好的理解这个概念。

我们创建一个名为base.html的基本模板，在其中定义了一个简单的HTML结构，后面可以用在一个简单的两列布局的页面上：

```php
<!DOCTYPE html>
<html>
  <head>
    {% raw %}
    {% block head %}
      <link rel="stylesheet" href="style.css" />
      <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
    {% endraw %}
  </head>
  <body>
    <div id="content">
      {% raw %}
      {% block content %}
      {% endblock %}
      {% endraw %}
    </div>
    <div id="footer">
      {% raw %}
      {% block footer %}
          &copy; Copyright 2011 by <a href="http://domain.invalid/">you</a>.
      {% endblock %}
      {% endraw %}
    </div>
  </body>
</html>
```

在这个例子中，[Block标记](http://twig.sensiolabs.org/doc/tags/block.html)定义了四个Block，子模板可以对这四个Block进行填充。所有的Block标记的作用就在于告诉模板引擎——子模板可能对模板的这一部分进行覆盖。

一个子模板看起来大概是：

```php
{% raw %}
{% extends "base.html" %}

{% block title %}Index{% endblock %}
{% block head %}
  {{ parent() }}
  <style type="text/css">
      .important { color: #336699; }
  </style>
{% endblock %}
{% block content %}
  <h1>Index</h1>
  <p class="important">
      Welcome to my awesome homepage.
  </p>
{% endblock %}
{% endraw %}
```

此处的关键就在于[Extends tag](http://twig.sensiolabs.org/doc/tags/extends.html)。他让模板引擎得知这个模板是“扩展自其他模板”。当模板系统处理这一模板的时候，就会首先定位父模板。Extends标记应该是子模板的第一个标记。

另外因为子模板中没有定义footer block，所以此处会继续使用父模板的内容。

也可以用[Parent函数](http://twig.sensiolabs.org/doc/functions/parent.html)来渲染父模板Block的内容。这一调用会返回父模板中的Block结果：

```php
{% raw %}
{% block sidebar %}
  <h3>Table Of Contents</h3>
  ...
  {{ parent() }}
{% endblock %}
{% endraw %}
```

*[extend文档](http://twig.sensiolabs.org/doc/tags/extends.html)中介绍了更深入的特性，例如Block嵌套、作用域、动态继承以及条件继承等。*

### HTML转义

从模板中生成HTML的时候，存在一种风险：一个变量可能包含了HTML代码，会影响到最终的HTML。有两种方式来解决这个问题：人工转义每一个变量，或者缺省情况下自动转义所有变量。Twig对两种方式都支持，缺省开启第二种。

*自动转义只有在escaper扩展激活的情况下才生效。*

- #### 人工转义

如果启用了人工转义，那么对需要进行处理的变量进行转义就成了开发者的责任。那么什么需要被转义呢？答案是所有不信任的变量。人工转义的方法是把变量传送给[escape filter](http://twig.sensiolabs.org/doc/filters/escape.html)或者e filter进行处理。

```php
{% raw %}
{{ user.username|e }}
{% endraw %}
```

- #### 自动转义

不管是否启用了自动转义，都可以利用[autoescape](http://twig.sensiolabs.org/doc/tags/autoescape.html)标记来标记模板中的一段内容。

```php
{% raw %}
{% autoescape %}
  Everything will be automatically escaped in this block (using the HTML strategy)
{% endautoescape %}
{% endraw %}
```

缺省情况下，自动转义使用html策略。如果输出内容属于其他上下文，则需要显式的指定其他的转义策略：

```php
{% raw %}
{% autoescape 'js' %}
这个区块中的所有字符都会被自动转义(采用JS策略)
{% endautoescape %}
{% endraw %}
```

> 转义

{% raw %}
让Twig能够忽略一部分本应当做是block或者变量来处理的东西，是个常见甚至必要的需求。例如想要以原始形态输出{{这样被语法占用的关键字，而不想被当做变量的开始，就可以使用一个小技巧。
{% endraw %}

最简单的方式就是使用一个变量表达式：

```php
{% raw %}
{{ '{{' }}
{% endraw %}
```

### 宏

在版本1.12中出现：支持缺省参数值。

宏和和其他语言中的函数类似。它可以复用经常使用的HTML片段。

使用[macro标记](http://twig.sensiolabs.org/doc/tags/macro.html)可以定义宏。下面是forms.html中的一个小例子：利用宏渲染一个form元素：

```php
{% raw %}
{% macro input(name, value, type, size) %}
  <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
{% endmacro %}
{% endraw %}
```

可以在任何模板中定义宏，然后使用[import标记](http://twig.sensiolabs.org/doc/tags/import.html)进行导入即可：

```php
{% raw %}
{% import "forms.html" as forms %}
<p>{{ forms.input('username') }}</p>
{% endraw %}
```

在从其他模板中进行引入的时候，还可以利用From标记为其赋予别名：

```php
{% raw %}
{% from 'forms.html' import input as input_field %}
<dl>
  <dt>Username</dt>
  <dd>{{ input_field('username') }}</dd>
  <dt>Password</dt>
  <dd>{{ input_field('password', '', 'password') }}</dd>
</dl>
{% endraw %}
```

还可以在为宏指定缺省参数：

```php
{% raw %}
{% macro input(name, value = "", type = "text", size = 20) %}
  <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}" size="{{ size }}" />
{% endmacro %}
{% endraw %}
```

### 表达式

Twig允许使用表达式，工作方式类似普通的PHP，即使你对PHP没那么熟悉，也可以轻松面对。

> 运算符优先级按递增顺序排列如下：
> b-and, b-xor, b-or, or, and, ==, !=, <, >, >=, <=, in, matches, starts with, ends with, .., +, -, ~, *, /, //, %, is, **, |, [], and

```php
{% raw %}
{% set greeting = 'Hello ' %}
{% set name = 'Fabien' %}

{{ greeting ~ name|lower }}   {# Hello fabien #}

{# use parenthesis to change precedence #}
{{ (greeting ~ name)|lower }} {# hello fabien #}
{% endraw %}
```

#### 常量

在1.5中新增：支持以名称和表达式作为哈希键

表达式的最简单形式就是常量了。他是PHP类型的一种实现，包含字符串、数字以及数组。以下是一些例子

- "Hello world"：单引号或双引号之间的内容是字符串。可以在任何需要字符串的情况下使用（例如作为函数或者Filter的参数，或者在extend以及include过程中使用）。
- 42 / 42.23：整数和浮点数。有小数点的是浮点数；否则就是整数。
- ["foo", "bar"]：数组由方括号包含的一组逗号分隔的表达式组成。
- {"foo": "bar"}：同数组类似，不过是以花括号进行包装。

```php
{% raw %}
{# keys as string #}
{ 'foo': 'foo', 'bar': 'bar' }

{# keys as names (equivalent to the previous hash) -- as of Twig 1.5 #}
{ foo: 'foo', bar: 'bar' }

{# keys as integer #}
{ 2: 'foo', 4: 'bar' }

{# keys as expressions (the expression must be enclosed into parentheses) -- as of Twig 1.5 #}
{ (1 + 1): 'foo', (a ~ 'b'): 'bar' }
{% endraw %}
```

- true / false：布尔值，你懂的
- null：表明无特定值。当访问一个不存在的变量时，就会返回null，none是null的别名。

数组和哈希可以嵌套：

```php
{% raw %}
{% set foo = [1, {"foo": "bar"}] %}
{% endraw %}
```

*字符串使用单双引号对性能没什么影响，但是只有双引号才支持字符串差值。*

#### 算数运算符

Twig可以进行运算。这个功能很少用得到，不过为了完整起见，还是提供了这个功能。目前支持下列操作符：

{% raw %}

- +：加法
- -：减法
- /：除法
- %：求余。{{ 11 % 7 }} 等于 4.
- //：整除，返回两个数相除得到的结果的整数部分。{{ 20 // 7 }}等于2
- *： 相乘，{{2 * 2}} 返回 4。
- **：乘方，{{ 2 3}}`等于8。

{% endraw %}

#### 逻辑运算符

可以用下面的操作符连接多个表达式：

- and：与运算
- or：或运算
- not：非运算
- (expr)：为表达式分组

*Twig还支持位操作：（b-and, b-xor 以及 b-or）。*

#### 比较运算符

Twig支持的比较运算符：==、!=、<、>、>=、<=，还可以对字符串使用starts with或者end with：

```php
{% raw %}
{% if 'Fabien' starts with 'F' %}
{% endif %}

{% if 'Fabien' ends with 'n' %}
{% endraw %}
```

*对于复杂的字符串比较，matches操作符提供了正则表达式的匹配能力：*

```php
{% raw %}
{% if phone matches '/^[\\d\\.]+$/' %}
{% endif %}
{% endraw %}
```

#### 包含操作符

操作符in提供了包含测试功能。

如果右侧操作数包含左侧的操作数，则返回true：

```php
{% raw %}
{# returns true #}

{{ 1 in [1, 2, 3] }}

{{ 'cd' in 'abcde' }}
{% endraw %}
```

还可以反着用（not in）：

```php
{% raw %}
{% if 1 not in [1, 2, 3] %}
{# 等价于 #}
{% if not (1 in [1, 2, 3]) %}
{% endraw %}
```

#### 测试操作符

is操作符用于测试。可以用于测试一个变量和表达式，右侧的操作数就是测试的名字：

```php
{% raw %}
{# 检测变量是不是奇数 #}
{{ name is odd }}
{% endraw %}
```

测试也可以接受参数：

```php
{% raw %}
{% if post.status is constant('Post::PUBLISHED') %}
{% endraw %}
```

可以使用is not来进行否定测试：

```php
{% raw %}
{% if post.status is not constant('Post::PUBLISHED') %}
{# 等价于 #}
{% if not (post.status is constant('Post::PUBLISHED')) %}
{% endraw %}
```

可以去[test](http://twig.sensiolabs.org/doc/tests/index.html)页面来了解一下内置的测试。

#### 其他操作符

1.12.0中加入了扩展的三元操作符。

- |：管道符号，应用一个filter
- ..：以左右两个操作数为基础生成一个序列

```php
{% raw %}
{{ 1..5 }}
{# 等价于 #}
{{ range(1, 5) }}
{% endraw %}
```

{% raw %}
- ~：把所有操作数转换为字符串并连接起来。{{ "Hello " ~ name ~ "!" }}会返回(假设name是'John')Hello John!。
- .,[]：获取对象的属性
- ?:：三元操作符

{% endraw %}

```php
{% raw %}
{{ foo ? 'yes' : 'no' }}
{# 在Twig 1.12.0中 #}
{{ foo ?: 'no' }} 等同于 {{ foo ? foo : 'no' }}
{{ foo ? 'yes' }} 等同于 {{ foo ? 'yes' : '' }}
{% endraw %}
```

#### 字符串插值

字符串差值(#{表达式})允许在双引号内的字符串中插入任何有效的表达式。表达式的结果会出现在最终的字符串中：

```php
{% raw %}
{{ "foo #{bar} baz" }}
{{ "foo #{1 + 2} baz" }}
{% endraw %}
```

### 空白控制

twig1.1新增tag级别的空白控制

跟PHP类似，模板标记后面的第一个空行会被自动删除。模板引擎目前已经不再自动清理空白，所有空白（空格、制表符、换行符等）都会原样输出。

使用`spaceless`标记可以移出html标记之间的空白：

```php
{% raw %}
{% spaceless %}
<div>
    <strong>foo bar</strong>
</div>
{% endspaceless %}

{# output will be <div><strong>foo bar</strong></div> #}
{% endraw %}
```

在spaceless标记之外，还可以在每个标记的级别控制空白。在标记上使用空白控制修饰符，可以清除前导或跟随的空白：

```php
{% raw %}
{% set value = 'no spaces' %}
{#- 取出首位空白 -#}
{%- if true -%}
    {{- value -}}
{%- endif -%}

{# 输出 'no spaces' #}
{% endraw %}
```

上面的例子演示了缺省的空白控制修饰符，以及如何移除标记周围的空白。还可以只清除Tag一边的空白：

```php
{% raw %}
{% set value = 'no spaces' %}
<li>
  {{- value }}
</li>
{# 输出 '<li>no spaces</li>' #}
{% endraw %}
```

### 扩展

- Twig的扩展很方便。

- 如果你要查找新的标记、filter或者函数，可以浏览官方的[扩展仓库](http://github.com/twigphp/Twig-extensions)。

- 如果想要创建自己的扩展，请阅读创建[扩展章节](https://drupal.fleeto.us/translation/extending-twig)。

## 三、开发者的Twig

本章讲述的是Twig的API，而不是Twig的模板语言。这些内容会对为应用实现模板接口的工作很有帮助，对模板的制作工作就没什么意义了。

### 基础

Twig的中心对象为**environment**（是Twig_Environment类的实例）。这个类的实例用于存储配置和扩展，并用来从文件系统或者其他位置载入模板。

绝大多数应用需要在应用初始化的时候，创建一个Twig_Environment对象，然后用它来加载模板。在有些案例中，会有多个环境同时并存，用来处理不同的配置。

下面举个简单例子，配置Twig来为应用载入模板：

```php
require_once '/path/to/lib/Twig/Autoloader.php';
Twig_Autoloader::register();

$loader = new Twig_Loader_Filesystem('/path/to/templates');
$twig = new Twig_Environment($loader, array(
    'cache' => '/path/to/compilation_cache',
));
```

上面的代码会创建一个缺省设置的环境，然后加载器会去/path/to/templates/目录查找模板。有不同的加载器可以使用，你可以实现自己的加载器用来从数据库或者其他资源中获取模板。

*注意，environment的第二个参数是一个数组选项。cache选项声明了一个目录，用于存储Twig编译的模版，以减少解析的开销。这种缓存跟你希望在处理模板过程中加入的缓存是不一样的，所以如果需要自行缓存，请使用其他的PHP缓存库。*

要从环境中调用模版，只需要调用`loadTemplate`函数，函数会返回一个`Twig_Template `实例

```php
$template = $twig->loadTemplate('index.html');
```

要渲染带有变量的模版，调用`render`函数

```php
echo $template->render(array('the' => 'variables', 'go' => 'here'));
```

*`display()`方法是用于直接输出模版的快捷方法。*

你还可以一气呵成加载和渲染模版：

```php
echo $twig->render('index.html', array('the' => 'variables', 'go' => 'here'));
```

### 环境选项

当创建一个新的Twig_Environment实例时，可以创建一个选项数组，作为构造函数的第二个参数：

```php
$twig = new Twig_Environment($loader, array('debug' => true));
```

可用的环境选项：

- debug：布尔值，设置为true的时候，生成的模版会带有一个` __toString()`方法，这个方法可以用来显示生成的Node（默认值false）
- charset：模版的字符集，默认是utf-8
- base_template_class：生成模版时使用的基本模版类，默认Twig_Template
- cache：用于存储缓存的绝对路径名，或者false用于禁用缓存（默认是false）
- auto_reload：布尔值，当用Twig开发时，源代码有修改时就重新编译对于开发者来说是非常必要的。如果不提供一个auto_reload选项，他会从debug选项中取值。
- strict_variables：布尔值，如果设置为false，Twig会忽略无效的变量（无效指的是不存在的变量或者属性/方法），并将其替换为null。如果这个选项设置为true，那么遇到这种情况的时候，Twig会抛出异常。默认值是false
- autoescape：字符串或者布尔值，如果设置为true，HTML的自动转义会默认对所有的模版起效，默认值是true。在Twig 1.8中，可以设置转义策略（html或者js，要关闭可以设置为false）。在Twig 1.9中的转义策略，可以设置为css，url，html_attr，甚至还可以设置为PHP回调函数，该函数需要接受一个模板文件名为参数，且必须返回要使用的转义策略，回调命名应该避免同内置的转义策略冲突。
- optimizations：整型值，一个标记，用于决定采用哪种优化方式（默认是-1，激活所有的优化方式；设置0的话将会禁用）

### 加载器

加载器负责从文件系统等资源中加载模版

#### 编译缓存

所有的模版加载器都会在文件系统中缓存编译的模版，已被将来调用。由于模版只编译一次，所以大大地提升了Twig的速度，如果你还使用了PHP加速器，例如APC，性能的提升将会更加的明显。参考上文提到的`Twig_Environment`中的`cache`以及`auto_reload`选项能够获得更多信息。

#### 内置加载器

这里有一个Twig提供的内置加载列表：

- Twig_Loader_Filesystem

  1.10新增prependPath()以及命名空间支持

- Twig_Loader_Filesystem
  
  从文件系统中加载模版。这个加载器可以在文件系统的目录中找到模版，也是推荐使用的加载器：
  
  ```php
    $loader = new Twig_Loader_Filesystem($templateDir);
  ```

  它还可以从目录数组中查找模版：

  ```php
    $loader = new Twig_Loader_Filesystem(array($templateDir1, $templateDir2));
  ```

  在这种配置下，Twig会首先在$templateDir1目录下查找模版，如果没有找到，再继续从templateDir2目录下查找。

  你可以通过`addPath()`和`prependPath()`方法添加或者追加路径

  ```php
    $loader->addPath($templateDir3);
    $loader->prependPath($templateDir4);
  ```

  filesystem加载器还支持命名空间模版。这一特性将允许用户把模版放在不同的带有命名空间的目录下。

  当使用`setPaths()`、`addPath()`以及`prependPath()`方法时，把命名空间作为第二个参数进行传递（如果没有传递，缺省命名空间为"main"）：

  ```php
    $loader->addPath($templateDir, 'admin');
  ```

  命名空间模版可以通过类似于`@namespace_name/template_path`的注解形式来访问。

  ```php
    $twig->render('@admin/index.html', array());
  ```

- Twig_Loader_Array

  Twig_Loader_Array从PHP数组中加载模版

  ```php
    {% raw %}
    $loader = new Twig_Loader_Array(array(
    'index.html' => 'Hello {{ name }}!',));
    $twig = new Twig_Environment($loader);
    echo $twig->render('index.html', array('name' => 'Fabien'));
    {% endraw %}
  ```

  这个加载器对于单元测试非常有用。这个加载器适合于那些需要把所有的模版都存在在一个单独php文件的小型项目中。

- Twig_Loader_Chain

  Twig_Loader_Chain委托其它加载器加在模版
  
  ```php
    {% raw %}
    $loader1 = new Twig_Loader_Array(array(
        'base.html' => '{% block content %}{% endblock %}',
    ));
    $loader2 = new Twig_Loader_Array(array(
        'index.html' => '{% extends "base.html" %}{% block content %}Hello {{ name }}{% endblock %}',
        'base.html'  => 'Will never be loaded',
    ));
    
    $loader = new Twig_Loader_Chain(array($loader1, $loader2));
    
    $twig = new Twig_Environment($loader);
    {% endraw %}
  ```

  当搜索模版的时候，Twig会依次尝试每一个加载器，一旦找到匹配模版，立即返回。当渲染上述例子中的index.html模版时，Twig会在$loader2中加载它，但是对于base.html模版，Twig会从$loader1中加载。

  Twig_Loader_Chain接受任何实现了Twig_LoaderInterface接口的加载器

  *你还可以通过addLoader()方法添加加载器*

#### 创建自己的加载器

实现了Twig_LoaderInterface接口的加载器：

```php
interface Twig_LoaderInterface
{
  /**
   * Gets the source code of a template, given its name.
   *
   * @param  string $name string The name of the template to load
   *
   * @return string The template source code
   */
  function getSource($name);

  /**
   * Gets the cache key to use for the cache for a given template name.
   *
   * @param  string $name string The name of the template to load
   *
   * @return string The cache key
   */
  function getCacheKey($name);

  /**
   * Returns true if the template is still fresh.
   *
   * @param string    $name The template name
   * @param timestamp $time The last modification time of the cached template
   */
  function isFresh($name, $time);
}
```

如果当前的缓存模版还是最新的，isFresh()方法必须返回true，否则应该返回最后一次的修改时间，或者直接false

*Twig 1.11.0，还可以实现Twig_ExistsLoaderInterface接口，让加载器在同链式加载器协同工作时稍快一些。*

### 使用扩展

Twig扩展是打包起来的一些新功能。使用扩展很简单，调用addExtension()方法：

```php
$twig->addExtension(new Twig_Extension_Sandbox());
```

Twig自带了以下扩展：

- Twig_Extension_Core：定义了Twig的所有核心特性

- Twig_Extension_Escaper：添加自动输出转义，以及代码库可能需要的转义或者反转义

- Twig_Extension_Sandbox：为Twig环境添加一个默认的沙盒模式，使不信任的可以安全的运行

- Twig_Extension_Profiler：启用内置的Twig profiler

- Twig_Extension_Optimizer：编译前优化节点树

核心、转义、以及优化扩展不需要添加到Twig环境中，他们默认就是启用的

### 内置扩展

这块儿将会描述内置扩展带来的新特性

#### 核心扩展

core扩展定义了Twig的所有核心特性：

- [Tags](http://twig.sensiolabs.org/doc/tags/index.html)

- [Filters](http://twig.sensiolabs.org/doc/filters/index.html)

- [Functions](http://twig.sensiolabs.org/doc/functions/index.html)

- [Tests](http://twig.sensiolabs.org/doc/tests/index.html)

#### 转义扩展

escaper扩展为Twig带来了自动输出转义，它定义了autoescape标记，raw filter。

自行创建escaper扩展时，你可以打开或者关闭全局的转义策略：

```php
$escaper = new Twig_Extension_Escaper('html');
$twig->addExtension($escaper);
```

如果设置成html，模版中所有的变量都会转义，除了那些raw filter输出的

```php
{% raw %}
{{ article.to_html|raw }}
{% endraw %}
```

通过使用autoescape标记，你可以改变转移模式（参考[autoescape文档](http://twig.sensiolabs.org/doc/tags/autoescape.html)）

```php
{% raw %}
{% autoescape 'html' %}
{{ var }}
{{ var|raw }}      {# var 不会被转义 #}
{{ var|escape }}   {# var 不会被再次转义 #}
{% endautoescape %}
{% endraw %}
```

*autoescape标记对包含进来的模版无效*

转义规则是这样实现的：

- 模板中的常量（整数、布尔、数组...）会直接当做变量或filter的参数，不会被自动转义：

```php
{% raw %}
{{ "Twig<br />" }} {# won't be escaped #}
{% set text = "Twig<br />" %}
{{ text }} {# will be escaped #}
{% endraw %}
```

- 对于一个结果是常量的表达式，或者一个标记为安全的变量也不会自动转义：

```php
{% raw %}
{{ foo ? "Twig<br />" : "<br />Twig" }} {# won't be escaped #}

{% set text = "Twig<br />" %}
{{ foo ? text : "<br />Twig" }} {# will be escaped #}

{% set text = "Twig<br />" %}
{{ foo ? text|raw : "<br />Twig" }} {# won't be escaped #}

{% set text = "Twig<br />" %}
{{ foo ? text|escape : "<br />Twig" }} {# the result of the expression won't be escaped #}
{% endraw %}
```

- 转义发生在打印之前，filter之后：

```php
{% raw %}
{{ var|upper }} {# 等价于 {{ var|upper|escape }} #}
{% endraw %}
```

- raw filter应该在filter链的最后使用：

```php
{% raw %}
{{ var|raw|upper }} {# 会转义 #}

{{ var|upper|raw }} {# 不转义 #}
{% endraw %}
```

- 如果最后一个filter在当前上下文中（例如html或者js）被标记为安全，escape和escape('html')被HTML标记为安全，escape('js')被JavaScript标记为安全，raw对所有内容都被标记为安全。

```php
{% raw %}
{% autoescape 'js' %}
  {{ var|escape('html') }} {# will be escaped for HTML and JavaScript #}
  {{ var }} {# will be escaped for JavaScript #}
  {{ var|escape('js') }} {# won't be double-escaped #}
{% endautoescape %}
{% endraw %}
```

{% raw %}
*注意自动转义是是在表达式运行之后发生，因此有局限性的。例如当进行连续操作时，{{ foo|raw ~ bar }}因为转义是最后发生的，因此无法给出期待的结果（这里的raw filter没有任何效果）。*
{% endraw %}

#### 沙盒扩展

沙箱扩展用于运行不受信任的代码,会限制访问不安全的属性和方法。沙箱安全性由一个策略实例来管理。缺省情况下，Twig会带有一个策略类：Twig_Sandbox_SecurityPolicy。这个类允许把某些标记、Filter、属性以及方法放入白名单：

```php
$tags = array('if');
$filters = array('upper');
$methods = array(
    'Article' => array('getTitle', 'getBody'),
);
$properties = array(
    'Article' => array('title', 'body'),
);
$functions = array('range');
$policy = new Twig_Sandbox_SecurityPolicy($tags, $filters, $methods, $properties, $functions);
```

在上面的例子中，安全策略只允许使用if标签和upper过滤器。另外，模版只能调用`Article`对象的`getTitle()`方法和`getBody()`方法，只能访问`title`和`body`这俩个公共属性。其它的都被禁止访问，如果尝试访问，则会抛出一个`Twig_Sandbox_SecurityError`异常。

策略对象是沙盒构造器的第一个参数：

```php
$sandbox = new Twig_Extension_Sandbox($policy);
$twig->addExtension($sandbox);
```

默认情况下，沙盒模式处于禁用状态。当包含的模版中含有不可信的代码时，使用`sandbox`标签激活沙盒模式。

```php
{% raw %}
{% sandbox %}
  {% include 'user.html' %}
{% endsandbox %}
{% endraw %}
```

你可以通过将扩展构造器的第二个参数传值true来对所有的模版启用沙盒模式：

```php
$sandbox = new Twig_Extension_Sandbox($policy, true);
```

#### 简介扩展
Twig1.18新增简介扩展

profiler扩展为Twig模版开启一个profiler，它应该只在开发机器上使用。

```php
$profile = new Twig_Profiler_Profile();
$twig->addExtension(new Twig_Extension_Profiler($profile));

$dumper = new Twig_Profiler_Dumper_Text();
echo $dumper->dump($profile);
```

一个profile包含了模版，块以及宏扩展的耗时和内存消耗信息。

你还可以以[blackfire.io](https://blackfire.io/)兼容格式dump这些数据。

```php
$dumper = new Twig_Profiler_Dumper_Blackfire();
file_put_contents('/path/to/profile.prof', $dumper->dump($profile));
```

上传并显示它(先创建个[免费帐号](https://blackfire.io/signup))

```bash
blackfire --slot=7 upload /path/to/profile.prof
```

#### 优化扩展

optimizer扩展会在编译模板之前对节点树进行优化：

```php
$twig->addExtension(new Twig_Extension_Optimizer());
```

默认情况下，所有的优化都是启用的。你也可以在构造器中传递您需要激活的扩展。

```php
$optimizer = new Twig_Extension_Optimizer(Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR);

$twig->addExtension($optimizer);
```

Twig支持下列优化：

- Twig_NodeVisitor_Optimizer：OPTIMIZE_ALL  默认值，启用所有优化
- Twig_NodeVisitor_Optimizer：OPTIMIZE_NONE  禁用所有的优化，这回减少编译时间，但是会增加执行时间和内存消耗。
- Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR  尽可能减少loop变量的创建，以提高for标记的效率。
- Twig_NodeVisitor_Optimizer::OPTIMIZE_RAW_FILTER  尽可能移出raw filter
- Twig_NodeVisitor_Optimizer::OPTIMIZE_VAR_ACCESS  尽可能的简化编译模版中变量的创建和访问

### 异常

Twig能抛出的异常：

- Twig_Error：所有异常
- Twig_Error_Syntax：语法错误
- Twig_Error_Runtime：运行时错误（当前实例中不包含某个filter）
- Twig_Error_Loader：模板载入时的异常
- Twig_Sandbox_SecurityError: 沙盒模式下出现了不允许使用的标记，filter或者调用了不允许调用的方法抛出的异常

## 扩展Twig

*本章描述了如何对Twig 1.12进行扩展。如果你在使用一个旧版本，请移步到[过往文档](http://twig.sensiolabs.org/doc/advanced_legacy.html)继续阅读。*

有多重扩展Twig的方法，可以加入额外的标记、Filter、Test、操作符、全局变量以及函数，甚至还能利用Node visitors对解析器本身进行扩展。

*本文的第一章节会介绍如何简单地扩展Twig。如果你想要在不同项目中进行复用，或者共享给他人使用，你应该按照接下来章节描述的那样创建一个扩展。*

在开始之前，你必须理解不同扩展点的区别及其应用场景。

首先，你需要记住的是，Twig有两个主要的语法结构：

{% raw %}
- {{}}：用来输出一个表达式的结果
- {%%}：用来执行语句
{% endraw %}

要理解为什么Twig提供这么多扩展点，先让我们看一下如何实现一个`Lorem ipsum`生成器。

你可以使用一个名为`lipsum`的标记：

```php
{% raw %}
{% lipsum 40 %}
{% endraw %}
```

这样的确可以起到效果，但是，使用`lipsum`标记的确不是一个好主意，因为：

- lipsum不是一个语言结构
- 这个标记输出了东西
- 这个标记不够灵活，不能在表达式中复用

```php
{% raw %}
{{ 'some text' ~ {% lipsum 40 %} ~ 'some more text' }}
{% endraw %}
```

事实上，你很少会使用标记；不过这也是个好消息——Tag的扩展是Twig扩展方式中最复杂的一种。

接下来，我们来试试filter的方式：

```php
{% raw %}
{{ 40|lipsum }}
{% endraw %}
```

再一次，它达到了效果，但是却看上去很古怪。filter用来把输入转换成输出，但是这里我们输入了的值是用于指示生成单词的数量——也就是说，40是一个参数，但不是我们转换的目标。

接下来，我们使用`lipsum`函数：

```php
{% raw %}
{{ lipsum(40) }}
{% endraw %}
```

这里我们也成功了。对于这个例子来说，这个扩展点的选择是合适的。现在可以在任何接受表达式的地方使用它了。

```php
{% raw %}
{{ 'some text' ~ lipsum(40) ~ 'some more text' }}

{% set lipsum = lipsum(40) %}
{% endraw %}
```

最后，还可以使用带有方法的全局对象来生成这些文字：

```php
{% raw %}
{{ text.lipsum(40) }}
{% endraw %}
```

### 全局变量

全局变量和其它的模版变量类似，只不过全局变量在所有的模版和宏定义中都有效：

```php
$twig = new Twig_Environment($loader);
$twig->addGlobal('text', new Text());
```

接下来你就可以在任何地方使用`text`变量了。

```php
{% raw %}
{{ text.lipsum(40) }}
{% endraw %}
```

### Filters

创建一个filter就是简单的给一个PHP回调一个名字：

```php
// 匿名函数
$filter = new Twig_SimpleFilter('rot13', function ($string) {
    return str_rot13($string);
});

// 或者一个简单的php函数
$filter = new Twig_SimpleFilter('rot13', 'str_rot13');

// 或者一个类的方法
$filter = new Twig_SimpleFilter('rot13', array('SomeClass', 'rot13Filter'));
```

传递给`Twig_SimpleFilter`构造器的第一个参数是将要在模版中使用的filter的名字，第二个参数是关于这个参数的php回调函数。

接下来，将filter加入到你的Twig环境中：

```php
$twig = new Twig_Environment($loader);
$twig->addFilter($filter);
```

然后就可以在模版中使用了：

```php
{% raw %}
{{ 'Twig'|rot13 }}

{# 输出： Gjvt #}
{% endraw %}
```

当Twig调用的时候，会把管道符左边的值作为第一个参数，并把括号中的Filter参数作为附加参数传递给回调。

例如，下面的代码：

```php
{% raw %}
{{ 'TWIG'|lower }}
{{ now|date('d/m/Y') }}
{% endraw %}
```

会被编译成下面这个样子：

```php
<?php echo strtolower('TWIG') ?>
<?php echo twig_date_format_filter($now, 'd/m/Y') ?>
```

`Twig_SimpleFilter`类将选项数组作为它的最后一个参数。

```php
$filter = new Twig_SimpleFilter('rot13', 'str_rot13', $options);
```

#### Filter环境感知

如果要在filter中访问当前的environment实例，需要在option中设置needs_environment为true；这样Twig就会把当前环境作为第一个参数传给filter：

```php
$filter = new Twig_SimpleFilter('rot13', function (Twig_Environment $env, $string) {
    // 获取当前实例的字符集
    $charset = $env->getCharset();

    return str_rot13($string);
}, array('needs_environment' => true));
```

#### Filter上下文感知

在Twig中访问上下文，则需要在option中设置needs_context为true，Twig就会吧当前上下文作为第一个参数（如果还设置了needs_environment为true，那么就会是第二个参数）进行传递：

```php
$filter = new Twig_SimpleFilter('rot13', function ($context, $string) {
    // ...
}, array('needs_context' => true));

$filter = new Twig_SimpleFilter('rot13', function (Twig_Environment $env, $context, $string) {
    // ...
}, array('needs_context' => true, 'needs_environment' => true));
```

#### 自动转义

如果启用了自动转义，filter的输出会在打印之前被转义。如果你的filter就是作为一个转义器（或者显式的输出HTML以及JavaScript代码），那么就需要输出原文。这种情况下，可以使用is_safe选项：

```php
$filter = new Twig_SimpleFilter('nl2br', 'nl2br', array('is_safe' => array('html')));
```

#### 参数可变的Filters

Twig1.19中新增可变参数filters

当需要一个filter接收的参数个数可变时，设置`is_variadic`选项为true；Twig会把余下的参数作为一个数组传递给filter的最后一个参数：

```php
$filter = new Twig_SimpleFilter('thumbnail', function ($file, array $options = array()) {
    // ...
}, array('is_variadic' => true));
```

#### 动态Filters

名字中带有特殊字符”*“的filter就是一个动态filter：

```php
$filter = new Twig_SimpleFilter('*_path', function ($name, $arguments) {
    // ...
});
```

上面定义的动态filter将会匹配下述列表：

- product_path
- category_path

动态Filter能够定义一个或者多个通配符部分：

```php
$filter = new Twig_SimpleFilter('*_path_*', function ($name, $suffix, $arguments) {
    // ...
});
```

这个Filter能够接受所有的动态值，并摆放在（环境和上下文之后）其他参数之前。例如，对'foo'|a_path_b()的调用，会传递这些参数：('a', 'b', 'foo')。

### 函数

函数的定义类似于filter，但是你需要创建一个`Twig_SimpleFunction`的实例：

```php
$twig = new Twig_Environment($loader);
$function = new Twig_SimpleFunction('function_name', function () {
    // ...
});
$twig->addFunction($function);
```

### Tests

Tests的定义和filter和函数的定义方法相同，你只需要创建一个`Twig_SimpleTest`实例：

```php
$twig = new Twig_Environment($loader);
$test = new Twig_SimpleTest('test_name', function () {
    // ...
});
$twig->addTest($test);
```

Tests允许创建自定义的带有逻辑处理的应用来处理布尔值的条件判断。举个简单的例子，创建一个Twig test来检测对象是否是‘red’：

```php
$twig = new Twig_Environment($loader);
$test = new Twig_SimpleTest('red', function ($value) {
  if (isset($value->color) && $value->color == 'red') {
      return true;
  }
  if (isset($value->paint) && $value->paint == 'red') {
      return true;
  }
  return false;
});
$twig->addTest($test);
```

Test函数只返回true/false。

当创建Test的时候，可以使用node_class选项来提供自定义的测试编制。如果你的test可以编译成PHP原语，这个办法就很有用了，这个办法在很多Twig的内置Test中使用：

```php
$twig = new Twig_Environment($loader);
$test = new Twig_SimpleTest(
    'odd',
    null,
    array('node_class' => 'Twig_Node_Expression_Test_Odd'));
$twig->addTest($test);

class Twig_Node_Expression_Test_Odd extends Twig_Node_Expression_Test
{
  public function compile(Twig_Compiler $compiler)
  {
    $compiler
      ->raw('(')
      ->subcompile($this->getNode('node'))
      ->raw(' % 2 == 1')
      ->raw(')')
    ;
  }
}
```

### Tags

Twig最令人激动的特性就是可以定义新的语言结构。这也是你需要理解的关于Twig最复杂的关于如何工作的特性。

让我们创建一个简单的set标记，这个标记用来在模板中定义一个简单变量，这个标记的用法如下：

```php
{% raw %}
{% set name = "value" %}
{{ name }}
{# 输出 value #}
{% endraw %}
```

三个步骤即可定义一个新的标记：

- 定义一个Token解析器（用来解析模板代码）
- 定义一个节点类(用于把解析的结果转换成PHP)
- 注册这个标记

#### 注册一个新的标记

简单的调用`Twig_Environment`实例的`addTokenParser`方法即可添加一个标记：

```php
$twig = new Twig_Environment($loader);
$twig->addTokenParser(new Project_Set_TokenParser());
```

#### 定义一个token解析器

来看下代码实例：

```php
class Project_Set_TokenParser extends Twig_TokenParser
{
  public function parse(Twig_Token $token)
  {
    $parser = $this->parser;
    $stream = $parser->getStream();

    $name = $stream->expect(Twig_Token::NAME_TYPE)->getValue();
    $stream->expect(Twig_Token::OPERATOR_TYPE, '=');
    $value = $parser->getExpressionParser()->parseExpression();
    $stream->expect(Twig_Token::BLOCK_END_TYPE);

    return new Project_Set_Node($name, $value, $token->getLine(), $this->getTag());
  }

  public function getTag()
  {
    return 'set';
  }
}
```

`getTag()`方法必须返回我们要解析的标记，这里返回的就是`set`.

`parse()`方法在解析器遇到`set`标记的时候就会被调用。这个方法应该返回一个`Twig_Node`实例用于显示这个结点。

Token流（$this->parser->getStream()）提供了一系列方法，简化了解析过程：

- getCurrent()：从流中获取当前的token
- next()：移动到流中的下一个Token，并返回之前的一个
- test($type), test($value）或者test($type, $value)：判断当前token是不是某种类型或值（或同时满足）。值可以是一个包含很多值的数组。
- expect($type[, $value[, $message]])：如果当前token不是指定的类型或值，就给出一个语法错误。否则，如果值和类型都正确，则返回当前token，并移动到下一个Token。
- look()：查看但不使用下一个token

像set标记这样，通过调用`parseExpression()`完成对表达式的解析。

#### 定义一个节点

Project_Set_Node类很简单：

```php
class Project_Set_Node extends Twig_Node
{
  public function __construct($name, Twig_Node_Expression $value, $line, $tag = null)
  {
    parent::__construct(array('value' => $value), array('name' => $name), $line, $tag);
  }

  public function compile(Twig_Compiler $compiler)
  {
    $compiler
        ->addDebugInfo($this)
        ->write('$context[\''.$this->getAttribute('name').'\'] = ')
        ->subcompile($this->getNode('value'))
        ->raw(";\n")
    ;
  }
}
```

编译器实现了一套接口和方法用于帮助开发者生成美观易读的PHP代码：

- subcompile()：编译一个结点
- raw()：原文输出字符串
- write()：输出首行缩进的字符串
- string()：输出引用字符
- repr()：输出php结果值
- addDebugInfo()：把与当前节点相关的原始模板文件当作注释添加进来
- indent()：缩进生成的代码
- outdent()：伸出生成的代码

### 创建一个扩展

开发扩展的一个主要目的可能就是为了将经常使用的功能规范称一个可复用的类，比如说，国际化支持。一个扩展可以指定标记、Filter、Test、运算符、全局变量、函数以及Node visitors。

扩展还可以分离编译时和运行时的代码，加快代码的运行速度。

大多数时候，创建一个单独的包含所有标记和filters的扩展将对你的项目很有用。

一个扩展就是一个实现了下面这个接口的类：

```php
interface Twig_ExtensionInterface
{
  /**
   * Initializes the runtime environment.
   *
   * This is where you can load some file that contains filter functions for instance.
   *
   * @param Twig_Environment $environment The current Twig_Environment instance
   *
   * @deprecated since 1.23 (to be removed in 2.0), implement Twig_Extension_InitRuntimeInterace instead
   */
  function initRuntime(Twig_Environment $environment);

  /**
   * Returns the token parser instances to add to the existing list.
   *
   * @return array An array of Twig_TokenParserInterface or Twig_TokenParserBrokerInterface instances
   */
  function getTokenParsers();

  /**
   * Returns the node visitor instances to add to the existing list.
   *
   * @return array An array of Twig_NodeVisitorInterface instances
   */
  function getNodeVisitors();

  /**
   * Returns a list of filters to add to the existing list.
   *
   * @return array An array of filters
   */
  function getFilters();

  /**
   * Returns a list of tests to add to the existing list.
   *
   * @return array An array of tests
   */
  function getTests();

  /**
   * Returns a list of functions to add to the existing list.
   *
   * @return array An array of functions
   */
  function getFunctions();

  /**
   * Returns a list of operators to add to the existing list.
   *
   * @return array An array of operators
   */
  function getOperators();

  /**
   * Returns a list of global variables to add to the existing list.
   *
   * @return array An array of global variables
   *
   * @deprecated since 1.23 (to be removed in 2.0), implement Twig_Extension_GlobalsInterface instead
   */
  function getGlobals();

  /**
   * Returns the name of the extension.
   *
   * @return string The extension name
   */
  function getName();
}
```

为了保持扩展类的整洁与稳定，还可以只继承内置的`Twig_Extension`类而不需要实现所有的接口。采用这种该方式的话，你只需要实现`getName()`方法。

`getName()`方法必须返回这个扩展的唯一标识符

现在我们来是一个最简单的扩展：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getName()
  {
    return 'project';
  }
}
``` 

Twig不关心你的扩展存在何处，因为所有的扩展都需要显示的注册。通过调用`Environment`对象的`addExtension()`方法即可注册一个扩展。

```php
$twig = new Twig_Environment($loader);
$twig->addExtension(new Project_Twig_Extension());
```

#### 全局变量

通过调用`getGlobals()`方法即可以在扩展中注册全局变量：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getGlobals()
  {
    return array(
        'text' => new Text(),
    );
  }

  // ...
}
```

#### 函数

通过调用`getFunctions()`方法即可以在扩展中注册函数：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getFunctions()
  {
    return array(
        new Twig_SimpleFunction('lipsum', 'generate_lipsum'),
    );
  }

  // ...
}
```

#### Filters

为了将filter添加到一个扩展中，你需要覆写`getFilters()`方法。这个方法必须返回一个添加到Twig环境的filters数组：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getFilters()
  {
    return array(
        new Twig_SimpleFilter('rot13', 'str_rot13'),
    );
  }

  // ...
}
```

#### 标记

通过覆写`getTokenParsers()`方法就可以将一个标记添加到一个扩展中。这个方法必须返回一个天极到Twig环境的tags数组：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getTokenParsers()
  {
    return array(new Project_Set_TokenParser());
  }

  // ...
}
```

#### 操作符

`getOperators()`方法用来新增新的操作符。下面的例子是如何添加!，||以及&&操作符：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getOperators()
  {
    return array(
      array(
        '!' => array('precedence' => 50, 'class' => 'Twig_Node_Expression_Unary_Not'),
      ),
      array(
        '||' => array('precedence' => 10, 'class' => 'Twig_Node_Expression_Binary_Or', 'associativity' => Twig_ExpressionParser::OPERATOR_LEFT),
        '&&' => array('precedence' => 15, 'class' => 'Twig_Node_Expression_Binary_And', 'associativity' => Twig_ExpressionParser::OPERATOR_LEFT),
      ),
    );
  }

  // ...
}
```

#### Tests

getTests()方法用来添加新的Test函数：

```php
class Project_Twig_Extension extends Twig_Extension
{
  public function getTests()
  {
    return array(
        new Twig_SimpleTest('even', 'twig_test_even'),
    );
  }

  // ...
}
```