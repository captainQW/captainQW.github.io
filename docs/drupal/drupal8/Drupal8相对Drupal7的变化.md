title: Drupal8相对Drupal7的变化
date: 2015-12-26 10:49:45
categories: Drupal8
tags: [drupal8]
---

### 1、概述

Drupal 8已经正式发布，与Drupal 7相比，Drupal 8在实现机制上发生了根本性的改变。最本质的改变，应该就是从主要以面向过程的编码方式转为了主要以面向对象的编码方式。虽然Drupal 8仍然保留了部分面向过程的文件，比如.module文件，但是可以预见，在Drupal 9中，这些面向过程的代码将被全部移除，所以在使用Drupal 8做开发的时候，应该尽量避免引用面向过程的代码，以免从Drupal 8升级到Drupal 9的过程中增加不必要的麻烦。

本文将从模块开发的角度（而非其他角度，比如网站编辑、性能优劣等）讨论Drupal 8 与Drupal 7的不同之处，通过对Drupal 8的研究，我将Drupal 8模块开发与Drupal 7的不同点概括为以下几点：

1. 目录结构；
2. 配置文件；
3. 路由方式；
4. 主题；
5. 表单；
6. 钩子机制在Drupal 8中的变化及实现；
7. Drupal 8中的新增机制。

下面将详细论述这些不同点。

### 2、目录结构

Drupal 7的目录结构为：

```
|-includes
|-misc
|-modules
|-scripts
|-sites
|  |-all
|  |  |-modules
|  |  |-themes
|  |-default
|-themes
```

Drupal 8的目录结构为：

```
|-core
|  |-assets
|  |-config
|  |-includes
|  |-lib
|  |-misc
|  |-modules
|  |-profiles
|  |-scripts
|  |-themes
|-modules
|-sites
|  |-default
|-themes
|-vendor
```

可以看出，Drupal 8与Drupal 7的目录结构大不相同，在Drupal7中核心的代码都直接放在根目录下，比如includes等，核心模块放在根目录下的modules目录下，核心主题放在根目录下的themes目录下。用户自定义或者第三方的模块放在sites\all\modules、主题放在sites\all\themes目录下。

而在Drupal 8里面，核心的东西都被移到根目录下的core目录下，原来用来放核心模块和主题的modules、themes目录分别用来放用户自定义的模块和主题。按照约定俗称的规则，一般在modules目录下新建contrib和custom目录，分别用来放贡献和自定义模块。

### 2、配置文件
在Drupal模块开发中，很多功能的实现都要靠配置文件来实现，比如模块的基本信息在Drupal 7中就必须配置在*.info文件中。在Drupal 8中，配置文件的使用得到了进一步的扩充，可以说，Drupal 8的几乎所有功能都离不开配置文件。

但是同时配置文件的格式也与Drupal 7有所不同。Drupal 7中的配置文件主要是.info文件，.info文件采用使用等号隔开的键值对的形式配置信息，例如模块信息的配置文件demo.info：

```
core            = "7.x"
description     = "Demo Module"
dependencies[] = system 
name            = "demo"
package         = "demo"
version = "7.x-1.0"
```

Drupal 8中配置文件广泛使用yaml格式的文件，例如模块信息的配置文件demo.info.yml:

```  
name: demo
type: module
description: "demo module"
package: demo
core: 8.x
dependencies:
  - rest
```

对比Drupal 7和Drupal 8的配置文件，不难看出，里面的内容基本一致，所不同的只是文件格式的不同。需要注意的是Drupal 8的配置文件后缀名都是.yml，表明这是一个yaml文件。Drupal 8的模块配置信息比Drupal 7的多一项type: module，这是因为Drupal 8的主题也是用同样格式的配置文件，为了区分这是一个主题还是模块，该项是必须的。

Drupal 8 不仅需要模块信息配置文件，还需要各种其他的配置文件，比如后面将要讲到的路由配置文件\*.routing.yml、服务配置文件\*.services.yml等。


### 3、路由方式
Drupal 7和Drupal 8的路由也存在很大的区别。Drupal 7 使用Hook的方式，而Drupal 8使用Symfony框架的路由配置文件的方式。本文只讨论Drupal 8和Drupal 7 的不同之处，更深入的介绍请参考[http://verynull.com/2015/12/23/Drupal8-routing/](http://verynull.com/2015/12/23/Drupal8-routing/ "Drupal8路由")和[http://verynull.com/2015/12/23/Drupal8-Controller/](http://verynull.com/2015/12/23/Drupal8-Controller/ "Drupal8控制器")

Drupal 7需要在module\_name.module中实现hook\_menu钩子来将某个url和请求处理方法关联起来，示例代码如下：

```php
function demo_menu() {
  $items = array();

  $items['demo'] = array(
    'title' => '示例',
    'page callback' => 'theme',
    'page arguments' => array('demo'),
    'access callback' => 'user_is_logged_in',
    'type' => MENU_CALLBACK
  );
  return $items;
}
```

上述代码表示，当用户访问/demo路径时。调用theme方法渲染demo主题，主题需要在hook\_theme中注册，这一点Drupal 8没有变化。

实现同样的功能，Drupal 8 首先需要配置module\_name.routing.yml，将路径和控制器中的某个action关联起来，然后编写控制器，实现里面的action。这里需要提到一个概念MVC，MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码。Drupal 8里面的控制器就是MVC中controller，用于衔接界面（view）和数据（model），即在控制器里面，接收参数，然后读取数据，传递给界面，最后返回给终端用户。

example.rounting.yml格式如下：

```
example.demo:
  path: '/demo/{name}'
  defaults:
    _controller: '\Drupal\example\Controller\ExampleController::demo'
    _title: 'test drupal 8 routing'
  requirements:
    _permission: 'access content'
```

example.demo为路由的机读名称，格式为"模块名.路由"，path是实际的url路径，其中{name}，表示参数，该参数会传递给action，参数名称就是name。

defaults下有的_controller: '\Drupal\example\Controller\ExampleController::demo'表示控制器为\Drupal\example\Controller\ExampleController，action为demo方法。

defaults下的_title: 'test drupal 8 routing'表示页面标题为：test drupal 8 routing。

requirements下的_permission表示可以访问该路径的权限。


ExampleController的代码如下：

```php
<?php
/**
 * @file
 * Contains ExampleController
 */

namespace Drupal\example\Controller;

use Drupal\Core\Controller\ControllerBase;

/**
 * Provides route responses for example.module.
 */
class ExampleController extends ControllerBase {
  /**
   * 展示hello {name}.
   * @param $name
   *
   * @return array
   *
   */
  public function demo($name) {
    return array(
      '#markup' => t('Hello ' . $name),
    );
  }
}
```

当完成上述代码后，清除缓存，在浏览器中访问http://yourdomain/demo/World，界面就会显示Hello World。example.info.yml、example.routing.yml和ExampleController就构成了Drupal 8中的一个最基本模块，一般来说，这三类文件是必须的。

### 4、主题
Drupal 8和Drupal 7相比，主题上也发生了本质的变化，首先是主题引擎的变化，其次是配置方式的变化，最后是js、css引入方式的变化。本文只讨论Drupal 8 与Drupal 7主题的不同之处，主题更深入的介绍请参考[http://verynull.com/2015/11/19/Drupal8主题-theme/](http://verynull.com/2015/11/19/Drupal8主题-theme/ "Drupal8主题")和[http://verynull.com/2015/11/18/Drupal8模板引擎-twig/](http://verynull.com/2015/11/18/Drupal8模板引擎-twig/ "Drupal8的模板引擎-twig")。

#### 4.1、模板引擎的变化 ####
Drupal 7使用的模板引擎是PHPTemplate，该模板引擎的特点是模板里面可以写php代码。这种模板的缺点显而易见，可能会产生XSS和SQL注入等攻击，如下代码：

```php
<?php
echo $name;
?>
```

如果$name = '&lt;script>js语句&lt;/script>'，这样输出到页面中就会执行script标签中的js代码，这样就会被黑客利用来攻击网站。

而Drupal 8则使用twig模板引擎，twig模板相对于PHPTemplate引擎，主要有以下有点：

1. 快速：Twig把模板编译成为优化后的PHP代码。相对普通PHP代码来说，其额外开销非常轻微。
2. 安全：Twig会使用一个沙箱模式来运行不信任的模板代码。这使得Twig可以在用户可以修改模板设计的应用中工作良好。
3. 弹性：Twig试用了弹性的词法和语法分析器。开发者可以定义自己的标记和过滤，并创建自己的DSL。

可以看出，twig里面不能写php代码，他有一套自己的语法，并且最终被编译为php，运行在安全沙箱中，一句话，就是安全。像上面的输出变量，在twig中如下：

```
{{ name }}
```

这样会自动把不安全的字符过滤掉。下面来比较PHPTemplate和twig的典型的不同之处。

1.后缀名

PHPTemplate的后缀名是.tpl.php，twig的后缀名为.html.twig.

2.变量

输出一个变量

PHPTemplate:

```php
<div class="content"><?php print $content; ?></div>
```

Twig:
```
<div class="content">{{ content }}</div>
```

输出一个哈希键值

PHPTemplate:

```php
<?php print $item['#item']['alt']; ?>
```

Twig:

```
{{ item['#item'].alt }}
```

变量赋值

PHPTemplate:

```php
<?php $custom_var = $content->comments; ?>
```

Twig:

```
{% set custom_var = content.comments %}
```

数组初始化

PHPTemplate:

```php
<?php
  $args = array('!author' => $author, '!date' => $created);
?>
```

Twig:

```
{% set args = {'!author': author, '!date': created} 
```

3.条件判断

PHPTemplate:

```php
<?php if ($content->comments): endif; ?>
```

Twig:

```
{% if content.comments %} {% endif %}
```

PHPTemplate:

```php
<?php if (!empty($content->comments)): endif; ?>
```

Twig:
```
{% if content.comments is not empty %} {% endif %}
```

PHPTemplate:

```php
<?php if (isset($content->comments)): endif; ?>
```
Twig:

```
{% if content.comments is defined %} {% endif %}
```

PHPTemplate:

```php
<?php if ($count > 0): endif; ?>
```

Twig:

```
{% if count > 0 %} {% endif %}
```

4.控制结构

PHPTemplate:

```php
<?php foreach ($users as $user) {} ?>
```

Twig:

```
{% for user in users %} {% endfor %}
```

5.过滤

check_plain：

PHPTemplate:

```php
<?php print check_plain($title); ?>
```

Twig:

```
{{ title|striptags }}
```

Translate：

PHPTemplate:

```php
<?php print t('Home'); ?>
```

Twig:

```
{{ 'Home'|t }}
```

更深入的介绍请参考上述文章。

#### 4.2、主题配置
Drupal 7的主题配置文件bartik.info格式如下：

```
name = Bartik
description = A flexible, recolorable theme with many regions.
package = Core
version = VERSION
core = 7.x

stylesheets[all][] = css/layout.css
stylesheets[all][] = css/style.css
stylesheets[all][] = css/colors.css
stylesheets[print][] = css/print.css

regions[header] = Header
regions[help] = Help
regions[page_top] = Page top
regions[page_bottom] = Page bottom
regions[highlighted] = Highlighted

regions[featured] = Featured
regions[content] = Content
regions[sidebar_first] = Sidebar first
regions[sidebar_second] = Sidebar second

regions[triptych_first] = Triptych first
regions[triptych_middle] = Triptych middle
regions[triptych_last] = Triptych last

regions[footer_firstcolumn] = Footer first column
regions[footer_secondcolumn] = Footer second column
regions[footer_thirdcolumn] = Footer third column
regions[footer_fourthcolumn] = Footer fourth column
regions[footer] = Footer

settings[shortcut_module_link] = 0
```

本文第二章节已经提到，Drupal 8 的配置文件都是用yaml文件，bartik.info.yml：

```
name: Bartik
type: theme
base theme: classy
description: 'A flexible, recolorable theme with many regions and a responsive, mobile-first layout.'
package: Core
# version: VERSION
# core: 8.x
libraries:
  - bartik/global-styling
ckeditor_stylesheets:
  - css/base/elements.css
  - css/components/captions.css
  - css/components/table.css
regions:
  header: Header
  primary_menu: 'Primary menu'
  secondary_menu: 'Secondary menu'
  page_top: 'Page top'
  page_bottom: 'Page bottom'
  highlighted: Highlighted
  featured_top: 'Featured top'
  breadcrumb: Breadcrumb
  content: Content
  sidebar_first: 'Sidebar first'
  sidebar_second: 'Sidebar second'
  featured_bottom_first: 'Featured bottom first'
  featured_bottom_second: 'Featured bottom second'
  featured_bottom_third: 'Featured bottom third'
  footer_first: 'Footer first'
  footer_second: 'Footer second'
  footer_third: 'Footer third'
  footer_fourth: 'Footer fourth'
  footer_fifth: 'Footer fifth'
```

从以上可以看出，Drupal 7 和Drupal 8 的主题配置文件大致相同，只是格式不一致，主要的不同有两点：

1.Drupal 8的配置文件多了type: theme，这是用来表明这是一个主题，而不是模块。

2.js、css的全局引入，Drupal 7可以直接引入单个js、css文件，而Drupal 8需要先将css、js设置成library，然后在主题配置文件中引入library，如上例中的：
    libraries:
	  - bartik/global-styling

表示引入bartik主题下的global-styling库。library库是在theme_name.libraries.yml文件中定义的，bartik.libraries.yml的部分代码如下:

```
color.preview:
  version: VERSION
  css:
    theme:
      color/preview.css: {}
  js:
    color/preview.js: {}
  dependencies:
    - color/drupal.color

maintenance_page:
  version: VERSION
  css:
    theme:
      css/maintenance-page.css: {}
  dependencies:
    - system/maintenance
    - bartik/global-styling
```

其中color.preview、maintenance_page就表示一个library，每个library下可以有css、js、dependencies等键，分别表示引入css、引入js和引入依赖的库。

#### 4.3、引入js、css
在4.2节中已经提到，Drupal 8中引入js和css必须使用library，不能单独引入单个js或css文件。在主题配置文件中引入全局的库也已经在4.2节中阐述，那在单个页面中引入js和css有何不同呢？

我们在Drupal 7中通常通过drupal\_add\_js和drupal\_add\_css来引入js和css，例如：

```php
drupal_add_js(drupal_get_path('module', 'demo') . '/js/demo.js');
drupal_add_css(drupal_get_path('module', 'demo') . '/css/demo.css');
```

在Drupal 8中，我们首先将js、css配置成库，如demo.libraries.yml:

```
base:
  version: VERSION
  css:
    theme:
      css/demo.css: {}
  js:
    js/demo.js: {}
  dependencies:
    - core/jquery
```

上面配置了一个base库，Drupal 8中除了要配置库，还要在渲染数组中使用#attached才能将库加到页面中：

```php
return array(
  '#theme' => 'example_demo',
  '#attached' => array(
    'library' => array(
      'example/base'
    )
  )
);
```

上述代码表示，渲染example_demo主题，并且加载example/base'库。

### 5、表单
我们将以一个简单的示例来分析Drupal 7和Drupal 8表单的异同。
在Drupal 7中要实现表单的展示、提交，首先需要配置菜单，将路径和表单函数关联起来在hook\_menu中定义如下菜单：

```php
$items['demo/form'] = array(
  'title' => 'Create demo form',
  'page callback' => 'drupal_get_form',
  'page arguments' => array('demo_form'),
  'access callback' => 'access content',
  'type' => MENU_LOCAL_TASK,
);
```

接下来，定义表单函数demo_form：

```php
function demo_form($form, &$form_state) {
  $form['name'] = array(
    '#type' => 'textfield',
    '#title' => '姓名'
  );
  $form['actions']['#type'] = 'actions';
  $form['actions']['submit'] = array(
    '#type' => 'submit',
    '#value' => $this->t('Save'),
    '#button_type' => 'primary',
  );
  $form['#validate'][] = 'demo_validate';
  $form['#submit'][] = 'demo_submit';
  return $form;
}
```

上述代码表明表单有一个文本框，一个提交按钮，表单的验证函数书demo\_validate，提交处理函数是demo\_submit。

下面实现demo\_validate、demo\_submit：

```php
function demo_validate($form, &$form_state) {
  $name = $form_state['value']['name'];
  if($name != 'world') {
    form_set_error('name', '参数错误');
  }
}

function demo_submit($form, &$form_state) {
  $name = $form_state['value']['name'];
  drupal_set_message('Hello' . $name);
}
```

到这里，我们就可以访问demo/form了，在文本框中输入world，则界面就会显示Hello world，但是输入其他的文本，则会报错“参数错误”。

接下来，我们来看一下Drupal 8中的表单是如何实现的，首先我们要配置一个路由指向表单，

```
example.demo_form:
  path: '/demo/form'
  defaults:
    _form: '\Drupal\example\Form\ExampleForm'
    _title: 'test drupal 8 form'
  requirements:
    _permission: 'access content'
```

path：'/demo/form'表示表单路径是/demo/form，_form: '\Drupal\example\Form\ExampleForm'表示表单类是\Drupal\example\Form\ExampleForm，那么下面就来实现这个类：

```php
<?php
	/**
	 * @file
	 * contains Drupal\example\Form\ExampleForm
	 */
	
	namespace Drupal\example\Form;
	
	
	use Drupal\Core\Form\FormBase;
	use Drupal\Core\Form\FormStateInterface;
	
	class ExampleForm extends FormBase
	{
	  /**
	   * @inheritdoc
	   */
	  public function getFormId() {
	    return 'example_form';
	  }
	
	  /**
	   * @inheritdoc
	   */
	  public function buildForm(array $form, FormStateInterface $form_state) {
	    $form['name'] = array(
	      '#type' => 'textfield',
	      '#title' => '姓名'
	    );
	    $form['actions']['#type'] = 'actions';
	    $form['actions']['submit'] = array(
	      '#type' => 'submit',
	      '#value' => $this->t('Save'),
	      '#button_type' => 'primary',
	    );
	    return $form;
	  }
	
	  /**
	   * @inheritdoc
	   */
	  public function validateForm(array &$form, FormStateInterface $form_state) {
	    parent::validateForm($form, $form_state);
	    if($form_state->getValue('name') != 'world') {
	      $a = array('name');
	      $form_state->setErrorByName('name', '参数错误');
	    }
	  }
	
	  /**
	   * @inheritdoc
	   */
	  public function submitForm(array &$form, FormStateInterface $form_state) {
	    $name = $form_state->getValue('name');
	    drupal_set_message('Hello' . $name);
	  }
	}
```

可见，Drupal 8中的表单就是一个类，继承自formBase，需要覆写父类的四个方法：getFormId、buildForm、validateForm和submitForm。

1.getFormId

返回表单ID，对应于Drupal 7中的表单的方法名；

2.buildForm

构造并返回表单，对应于Drupal 7中的表单函数。

3.validateForm

表单验证方法，对应于Drupal 7中的$form['#validate'][] = 'demo_validate'中指定的方法

4.submitForm
表单提交处理方法，对应于Drupal 7 中$form['#submit'][] = 'demo_submit'中指定的方法。

通过对比可以发现，Drupal 8将表单的各个元素组织在一个类里面，结构清晰，逻辑分明，相对于Drupal 7松散的代码是很大的进步。

### 6、钩子机制在Drupal 8中的变化及实现 ###
钩子机制在Drupal 7中起着举足轻重的作用，主要体现在两个方面：

一是info类的钩子，例如hook\_menu、hook\_theme、hook\_services\_resources等，起到关联信息的作用，比如hook\_menu告诉系统URL路径和处理方法的对应关系以便用户访问某个路径的时候找到正确的函数来处理；

二是调用类钩子，例如hook\_user\_delete等，调用类钩子的作用是系统在做了某个操作之后通过module\_invoke\_all来调用其他模块中实现的钩子函数。比如，删除某个用户之后，可能需要删除该用户关联的其他资源，则模块需要通过实现hook\_user\_delete钩子。

Drupal 8中这两类钩子都有了相应的替代方法，info类型的钩子大部分被插件（Plugin）代替，调用类型的钩子则被事件（Event）代替。下面分别针对两种钩子选择具有代表性的功能做分析。

#### 6.1、调用钩子的替代与实现
调用钩子在Drupal 7 中最有代表性的应用是用户的删除，实现需要分两步：

1.实现hook\_user\_delete方法，以demo模块为例：

```php
function demo_user_delete($account) {
  db_delete('user_demo', 'ud')
  ->condition('ud.uid', $account->uid)
  ->execute();
} 
```

2.在用户删除的方法里面通过module\_invoke\_all来调用钩子：

```php
module_invoke_all('user_delete', $account);
```

只需要两步就实现了用户删除之后，删除关联资源的功能。

Drupal 8里面的事件的三个要素：事件、事件监听和事件触发。

要实现事件机制，首先我们必须先定义一个事件（Event），比如站点配置保存事件，其次有了这个事件后，需要有订阅者来监听这个事件，最后在某个时间点触发这个事件，将事件传递给所有监听这个事件的订阅者。

我们以站点配置保存事件为例，介绍事件订阅的实现：

1.定义事件

Drupal 8的核心模块Config模块中定义了ConfigEvent，代码如下：

```php
<?php

/**
 * @file
 * Contains \Drupal\Core\Config\ConfigEvents.
 */

namespace Drupal\Core\Config;

/**
 * Defines events for the configuration system.
 *
 * @see \Drupal\Core\Config\ConfigCrudEvent
 */
final class ConfigEvents {

  /**
   * Name of the event fired when saving a configuration object.
   *
   * This event allows modules to perform an action whenever a configuration
   * object is saved. The event listener method receives a
   * \Drupal\Core\Config\ConfigCrudEvent instance.
   *
   * See hook_update_N() documentation for safe configuration API usage and
   * restrictions as this event will be fired when configuration is saved by
   * hook_update_N().
   *
   * @Event
   *
   * @see \Drupal\Core\Config\ConfigCrudEvent
   * @see \Drupal\Core\Config\Config::save()
   * @see \Drupal\Core\Config\ConfigFactory::onConfigSave()
   * @see hook_update_N()
   *
   * @var string
   */
  const SAVE = 'config.save';

  /**
   * Name of the event fired when deleting a configuration object.
   *
   * This event allows modules to perform an action whenever a configuration
   * object is deleted. The event listener method receives a
   * \Drupal\Core\Config\ConfigCrudEvent instance.
   *
   * See hook_update_N() documentation for safe configuration API usage and
   * restrictions as this event will be fired when configuration is deleted by
   * hook_update_N().
   *
   * @Event
   *
   * @see \Drupal\Core\Config\ConfigCrudEvent
   * @see \Drupal\Core\Config\Config::delete()
   * @see \Drupal\Core\Config\ConfigFactory::onConfigDelete()
   * @see hook_update_N()
   *
   * @var string
   */
  const DELETE = 'config.delete';

  /**
   * Name of the event fired when renaming a configuration object.
   *
   * This event allows modules to perform an action whenever a configuration
   * object's name is changed. The event listener method receives a
   * \Drupal\Core\Config\ConfigRenameEvent instance.
   *
   * See hook_update_N() documentation for safe configuration API usage and
   * restrictions as this event will be fired when configuration is renamed by
   * hook_update_N().
   *
   * @Event
   *
   * @see \Drupal\Core\Config\ConfigRenameEvent
   * @see \Drupal\Core\Config\ConfigFactoryInterface::rename()
   * @see hook_update_N()
   *
   * @var string
   */
  const RENAME = 'config.rename';

  /**
   * Name of the event fired when validating imported configuration.
   *
   * This event allows modules to perform additional validation operations when
   * configuration is being imported. The event listener method receives a
   * \Drupal\Core\Config\ConfigImporterEvent instance.
   *
   * @Event
   *
   * @see \Drupal\Core\Config\ConfigImporterEvent
   * @see \Drupal\Core\Config\ConfigImporter::validate().
   * @see \Drupal\Core\EventSubscriber\ConfigImportSubscriber::onConfigImporterValidate().
   *
   * @var string
   */
  const IMPORT_VALIDATE = 'config.importer.validate';

  /**
   * Name of the event fired when importing configuration to target storage.
   *
   * This event allows modules to perform additional actions when configuration
   * is imported. The event listener method receives a
   * \Drupal\Core\Config\ConfigImporterEvent instance.
   *
   * @Event
   *
   * @see \Drupal\Core\Config\ConfigImporterEvent
   * @see \Drupal\Core\Config\ConfigImporter::import().
   * @see \Drupal\Core\EventSubscriber\ConfigSnapshotSubscriber::onConfigImporterImport().
   *
   * @var string
   */
  const IMPORT = 'config.importer.import';

  /**
   * Name of event fired when missing content dependencies are detected.
   *
   * Events subscribers are fired as part of the configuration import batch.
   * Each subscribe should call
   * \Drupal\Core\Config\MissingContentEvent::resolveMissingContent() when they
   * address a missing dependency. To address large amounts of dependencies
   * subscribers can call
   * \Drupal\Core\Config\MissingContentEvent::stopPropagation() which will stop
   * calling other events and guarantee that the configuration import batch will
   * fire the event again to continue processing missing content dependencies.
   *
   * @see \Drupal\Core\Config\ConfigImporter::processMissingContent()
   * @see \Drupal\Core\Config\MissingContentEvent
   */
  const IMPORT_MISSING_CONTENT = 'config.importer.missing_content';

  /**
   * Name of event fired to collect information on all config collections.
   *
   * This event allows modules to add to the list of configuration collections
   * retrieved by \Drupal\Core\Config\ConfigManager::getConfigCollectionInfo().
   * The event listener method receives a
   * \Drupal\Core\Config\ConfigCollectionInfo instance.
   *
   * @Event
   *
   * @see \Drupal\Core\Config\ConfigCollectionInfo
   * @see \Drupal\Core\Config\ConfigManager::getConfigCollectionInfo()
   * @see \Drupal\Core\Config\ConfigFactoryOverrideBase
   *
   * @var string
   */
  const COLLECTION_INFO = 'config.collection_info';

}
```

该事件类中定义了一系列的事件，例如const SAVE = 'config.save'的意思即定义了配置保存事件，当配置保存完成后触发该事件。

有了事件之后，我们就需要有事件订阅者来监听这个事件，一旦配置保存，则执行相应的代码。我们仍然以demo模块为例，假设配置保存后，我们的demo模块需要记录一条日志，那demo模块需要实现相应的事件订阅类：

```php
<?php

/**
 * @file
 * Contains \Drupal\demo\EventSubscriber\ConfigSubscriber.
 */

namespace Drupal\demo\EventSubscriber;

use Drupal\Core\Config\ConfigFactoryInterface;
use Drupal\Core\Config\ConfigCrudEvent;
use Drupal\Core\Config\ConfigEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

/**
 * 记录日志，如果配置保存了
 */
class ConfigSubscriber implements EventSubscriberInterface {

  /**
   * 日志记录
   *
   *
   * @param ConfigCrudEvent $event
   *   The configuration event.
   */
  public function onConfigSave(ConfigCrudEvent $event) {
    $saved_config = $event->getConfig();
    $logger = \Drupal::logger('config save');
    $logger->notice($saved_config->getName() . '保存了');
  }

  /**
   * {@inheritdoc}
   */
  static function getSubscribedEvents() {
    $events[ConfigEvents::SAVE][] = array('onConfigSave', 0);
    return $events;
  }

}
```

这个事件订阅类实现了EventSubscriberInterface，这个接口只有一个方法getSubscribedEvents，这个方法返回事件对用的处理方法，比如$events[ConfigEvents::SAVE][] = array('onConfigSave', 0)表示ConfigEvents::SAVE事件的处理方法为onConfigSave，后面的第二个参数0表示优先级，越大越先执行。

实现了这个订阅类之后，为了让事件调度器（event dispacher）知道这个订阅者，我们需要把这个订阅类配置成服务，在demo.services.yml中我们增加如下代码：

```
demo.config_subscriber:
  class: Drupal\demo\EventSubscriber\ConfigSubscriber
  arguments: []
  tags:
    - { name: event_subscriber }
```

到此事件订阅的工作就完成了，要触发事件，必须要在配置保存的时候使用事件调度器将事件传递给订阅者，代码如下：

```php
public function save($has_trusted_data = FALSE) {
	// Validate the configuration object name before saving.
	static::validateName($this->name);
	
	// If there is a schema for this configuration object, cast all values to
	// conform to the schema.
	if (!$has_trusted_data) {
	  if ($this->typedConfigManager->hasConfigSchema($this->name)) {
	    // Ensure that the schema wrapper has the latest data.
	    $this->schemaWrapper = NULL;
	    foreach ($this->data as $key => $value) {
	      $this->data[$key] = $this->castValue($key, $value);
	    }
	  }
	  else {
	    foreach ($this->data as $key => $value) {
	      $this->validateValue($key, $value);
	    }
	  }
	}
	
	$this->storage->write($this->name, $this->data);
	if (!$this->isNew) {
	  Cache::invalidateTags($this->getCacheTags());
	}
	$this->isNew = FALSE;
	$this->eventDispatcher->dispatch(ConfigEvents::SAVE, new ConfigCrudEvent($this));
	$this->originalData = $this->data;
	// Potentially configuration schema could have changed the underlying data's
	// types.
	$this->resetOverriddenData();
	return $this;
}
```

上述代码为Config模块中配置保存的方法，其中的$this->eventDispatcher->dispatch(ConfigEvents::SAVE, new ConfigCrudEvent($this))这句代码就是讲事件传递给所有监听了ConfigEvents::SAVE事件的订阅者，当然也包括我们demo模块里面的订阅者。

关于自定义事件的实现请参考[http://www.sitepoint.com/drupal-8-hooks-symfony-event-dispatcher/](http://www.sitepoint.com/drupal-8-hooks-symfony-event-dispatcher/ "自定义事件")

#### 6.2、info类钩子的替换与实现
info类钩子的一个重要应用是web services。在Drupal 7中，内核不带rest，需要第三方模块。而在drupal8中，核心自带rest，除了rest界面管理模块（rest UI）外，不需要第三方模块。

Drupal 7中实现一个web services需要以下几步：

1.下载Services、REST Server、Chaos tools、OAuth Authentication第三方模块放入相应目录，进入后台->模块，勾选这几个模块保存启用。

2.实现hook\_services\_resources,代码如下：

```php
/**
 * Implements hook_services_resources().
 */
function demo_services_resources() {  
  return array(
    'demo' => array(
      'actions' => array(
        'add' => array(
          'help' => '创建示例',
          'file' => array('file' => 'inc', 'module' => 'demo', 'name' => 'demo.apis'),
          'callback' => 'demo_add_resources',
          'access arguments' => array('access_to_demo_api'),
          'args' => array(
            array(
              'name' => 'data',
              'source' => 'data',
              'type' => 'array',
              'optional' => FALSE,
            ),
          ),
        ),
      ),           
    ),
  );
}
```

其中数组的第一层表示接口地址的第一个参数，actions下的每一个键表示一个接口，同时也是接口地址的第二个参数，例如上面的例子表示demo/add接口，callback表示接口的处理函数。file表示接口方法所在的文件。

接下来，我们就要实现demo\_add\_resources方法：

```php
function demo_add_resources($data) {
  $return = array(
    'code' => 0,
    'message' => '操作失败'
  );
  return $return;
}
```

最后我们要进入系统管理界面-》结构-》services添加一个端点，我们将机读名称设置为api，服务器选择rest，端点路径设为API，保存，则端点新建成功。点击编辑，将我们刚才新建的接口勾选，保存。这时，接口就发布成功，访问路径为API/demo/add。

Drupal 8中，web services用插件的形式实现，具体步骤：

1.下载rest UI第三方模块，放入相应目录，进入后台-》扩展，勾选rest UI、RESTful Web Services、Serialization、HAL、HTTP Basic Authentication，保存。

2.新建一个自定义的反序列化类。因为Drupal 8 在处理web service的请求时，首先将接收的数据反序列化为某个类，然后处理数据，最后将返回值序列化为json或者xml等数据返回给客户端。所以必须要有反序列化和序列化的类。但是Drupal 8核心中的反序列化类只支持将数据反序列化为entity，这样，对于写接口会非常不便，比如，某个API传递的参数可能并不能和某个entity对应，这样这个接口就没法实现，所以必须要自定义一个反序列化的类，比如可以将数据反序列化为数组。代码如下：

```php
<?php
/**
 * @file
 * Contains \Drupal\demo\Normalizer\CustomNormalizer.
 */

namespace Drupal\demo\Normalizer;

class CustomNormalizer extends AbstractNormalizer {

  /**
   * {@inheritdoc}
   */
  public function supportsNormalization($data, $format = null)
  {
    return false;
  }

  /**
   * {@inheritdoc}
   *
   */
  public function normalize($object, $format = null, array $context = array())
  {
	return '';
  }


  /**
   * {@inheritdoc}
   */
  public function supportsDenormalization($data, $type, $format = null)
  {
    return $type == 'Array';
  }

  /**
   * {@inheritdoc}
   */
  public function denormalize($data, $class, $format = null, array $context = array())
  {
    $array = (array) $data;
    return $array;
  }

}
```

实现上述类之后，我们需要将该类定义为service，在demo.services.yml中添加如下代码：

```
demo.normalizer.custom:
  class: Drupal\demo\Normalizer\CustomNormalizer
  tags:
    - { name: normalizer }
```

接下来，再实现插件，插件其实是一个类，该类继承自该种插件的基类，并且需要用注解表明该类的插件性质。

```php
<?php
/**
 * @file
 * Contains \Drupal\demo\Plugin\rest\resource\ExampleResource
 */

namespace Drupal\demo\Plugin\rest\resource;

use Drupal\rest\Plugin\ResourceBase;
use Drupal\rest\ResourceResponse;
use Symfony\Component\HttpFoundation\Request;

/**
 * Represents demo as resources.
 *
 * @RestResource(
 *   id = "demo",
 *   label = @Translation("Demo Resource"),
 *   serialization_class = "Array",
 *   uri_paths = {
 *     "canonical" = "/API/demo",
 *     
 *   }
 * )
 *
 */
class ExampleResource extends ResourceBase {

  /**
   * Responds to GET requests.
   *
   * @param array $data
   * @param Symfony\Component\HttpFoundation\Request $request
   *
   * @return \Drupal\rest\ResourceResponse
   *   The response containing the entity with its accessible fields.
   *
   */
  public function get($data, Request $request) {
    $logger = \Drupal::logger('rest_get');
    $logger->notice($request->get('sid'));
    $return = array(
      'code' => 1,
      'message' => '这是get方法',
    );
    $response = new ResourceResponse($return, 200);
    return $response;
  }

  /**
   * Responds to POST requests.
   *
   * @param array $data
   * @param Symfony\Component\HttpFoundation\Request $request
   *
   * @return \Drupal\rest\ResourceResponse
   *   The response containing the entity with its accessible fields.
   *
   */
  public function post($data, Request $request) {
    $logger = \Drupal::logger('rest_post');
    $logger->notice('<pre>' . print_r($data, TRUE). '</pre>');
    $return = array(
      'code' => 1,
      'message' => '这是post方法',
      'data' => $data
    );
    $response = new ResourceResponse($return, 200);
    return $response;
  }

  /**
   * Responds to DELETE requests.
   *
   * @param array $data
   * @param Symfony\Component\HttpFoundation\Request $request
   *
   * @return \Drupal\rest\ResourceResponse
   *   The response containing the entity with its accessible fields.
   *
   */
  public function delete($data, Request $request) {
    $logger = \Drupal::logger('rest_delete');
    $logger->notice('<pre>' . print_r($data, TRUE). '</pre>');
    $return = array(
      'code' => 1,
      'message' => '这是delete方法',
      'data' => $data
    );
    $response = new ResourceResponse($return, 200);
    return $response;
  }

}
```

下面我们来分析一下上述代码，

1.@RestResource注解表示该类是RestResource类型的插件，也就是该类实现rest web services；

2.注解中的ID是该插件的ID，label是插件的名称；

3.serialization_class就是序列化和反序列化的类，我们这里设置为“Array”，则代码会自动调用我们的自定义的反序列化类来反序列化数据；

4."canonical" = "/API/demo"表示api的路径是/API/demo；"https://www.drupal.org/link-relations/create" = "/API/demo"，表示POST方法提交时的路径，当用POST方法提交时，会覆盖canonical指定路径。

这个web services插件内部实现了三个方法，get、post、delete，分别对应以get方法、post方法和delete方法提交的处理方法。换句话说，如果我以get方法访问API/demo，则调用get，以post方法访问API/demo，则调用post，以delete方法访问API/demo则调用delete。

到目前为止，我们的代码就全部写完，要使用该API，需要进入后台-》配置-》rest，启用这个api，您可以选择性的启用get、post和delete方法，同时选择请求和返回的数据格式，比如hal\_json、json、xml，点击保存后，您就可以访问该api了。

### 7、Drupal 8中的新增机制
Drupal 8采用了面向对象的方法，自然也会引入很多面向对象的新特性，比如依赖注入、服务容器，本文只是提出这些新特性，更深入的介绍请参考相应的文章。

1. 依赖注入和服务容器，请参考[http://verynull.com/2015/12/15/Drupal8-Service-DependencyInjection/](http://verynull.com/2015/12/15/Drupal8-Service-DependencyInjection/ "依赖注入和服务容器")
2. 注解，请参考[http://verynull.com/2015/12/12/Drupal8注解-Annotations-语法/](http://verynull.com/2015/12/12/Drupal8注解-Annotations-语法/ "注解")
3. content entity，请参考[http://verynull.com/2015/12/22/Drupal8-Content-Entity/](http://verynull.com/2015/12/22/Drupal8-Content-Entity/ "content entity")
4. configuration entity，请参考[http://verynull.com/2015/12/21/Drupal8-configuration-Entity/](http://verynull.com/2015/12/21/Drupal8-configuration-Entity/ "Configuration Entity")


参考文献

1. [http://drupalchina.cn/node/3122](http://drupalchina.cn/node/3122)
2. [https://www.drupal.org/node/2216195](https://www.drupal.org/node/2216195)
3. [https://drupalize.me/blog/201409/unravelling-drupal-8-plugin-system](https://drupalize.me/blog/201409/unravelling-drupal-8-plugin-system)
4. [http://www.sitepoint.com/drupal-8-hooks-symfony-event-dispatcher/](http://www.sitepoint.com/drupal-8-hooks-symfony-event-dispatcher/)
5. [http://drupalchina.cn/node/3353](http://drupalchina.cn/node/3353)
6. [https://dev.acquia.com/blog/introduction-restful-web-services-drupal-8](https://dev.acquia.com/blog/introduction-restful-web-services-drupal-8)











