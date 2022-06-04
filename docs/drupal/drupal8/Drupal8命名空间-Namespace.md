title: Drupal8命名空间(Namespace)
date: 2015-12-11 16:05:20
categories: Drupal8
tags: [namespace]
---

命名空间是php5.3引进的概念，这边文章主要介绍在Drupal中如何使用命名空间，如果你对命名空间还不熟悉，请移步到[这里](http://php.net/manual/zh/language.namespaces.php "PHP:命名空间")学习，这篇文章[article introducing namespaces](http://www.sitepoint.com/php-53-namespaces-basics/ "php53:namespace")也讲的不错。

在Drupal中，并不是每一个文件都需要指定命名空间。Drupal8之前为了跟php5.2保持兼容，从未用过命名空间。Drupal8限定了php版本必须是5.5以上，So，让我们尽情的享受OOP的乐趣吧！

###用"use"导入类

> 在访问系统内部或包含在命名空间中的类名称时，可以不使用完全限定名称。命名空间和use关键字的声明必须放在文件的开头，并且以反斜杠\做为分隔符。如

```php
<?php
namespace Drupal\mymodule\Tests\Foo;

use Drupal\simpletest\WebTestBase;

/**
 * Tests that the foo bars.
 */
class BarTest extends WebTestBase {
	// WebTestBase前不需要加\
}
?>
```

> 在访问系统内部或不包含在命名空间中的类名称时，必须使用完全限定名称。如

```php
new \Exception();
```

> 在一个没有声明命名空间的文件里，就算它是全局命名空间，如果要使用其他非全局命名空间的类，必须使用"use"关键字，并且需要声明在文件的开头哦！

> 当用"use"导入类时，不要在开头加\。大PHP官方也是这么推荐的，见[PHP documentation](http://www.php.net/manual/en/language.namespaces.importing.php)

> 当在字符串中指定类名时，必须使用带命名空间的全类名，并且不能以\开头。
	
	如果要在双引号中使用命名空间，必须转义，如：
	
```php
"Drupal\\Content\\ContentInterface"
```

	在单引号中就不需要转义了，如：

```php
'Drupal\Context\ContextInterface'
```

	通常情况下，推荐使用单引号的写法！

> 一个"use"只能声明一个类，不建议使用"use"导入多个类.如这种写法就是不推荐的
	
```php
use My\Full\Classname, My\Full\NSname;
```

	而这种写法是推荐的

```php
use My\Full\Classname;
```

> 在.api.php这种API文件中必须使用类的全名，这样别人要想hook你的类，他们就可以"use"导入了。如

定义Drupal\Subsystem\Foo

```php
<?php
/**
 * @file
 * Contains \Drupal\Subsystem\Foo.
 */

namespace Drupal\Subsystem;

// This imports just the Cat class from the Drupal\Othersystem namespace.
use Drupal\Othersystem\Cat;

// Bar is a class in the Drupal\Subsystem namespace in another file.
// It is already available without any importing.

/**
 * Defines a Foo.
 */
class Foo {

  /**
   * Constructs a new Foo object.
   */
  public function __construct(Bar $b, Cat $c) {
    // Global classes must be prefixed with a \ character.
    $d = new \DateTime();
  }

}
?>
```

hook Drupal\Subsystem\Foo

```php
<?php
/**
 * @file
 * The Example module.
 *
 * This file is not part of any namespace, so all global namespaced classes
 *  are automatically available.
 */

use Drupal\Subsystem\Foo;

/**
 * Does stuff with Foo stuff.
 *
 * @param \Drupal\Subsystem\Foo $f
 *   A Foo object used to bar the baz.
 */
function do_stuff(Foo $f) {
  // The DateTime class does not need to be imported as it is already global
  $d = new DateTime();
}
?>
```

###类的别名

php允许导入类的时候使用别名，这样可以避免可恶的重名！一旦两个类重名了，可以将各自命名空间里的上一级的名字拿出来作为相应的前缀。

如：

```php
<?php
use Foo\Bar\Baz as BarBaz;
use Stuff\Thing\Baz as ThingBaz;

/**
 * Tests stuff for the whichever.
 */
function test() {
  $a = new BarBaz(); // This will be Foo\Bar\Baz
  $b = new ThingBaz(); // This will be Stuff\Thing\Baz
}
?>
```

###导入的顺序

如果要导入多个类，Drupal8并没有限制导入的顺序。如：

```php
<?php
namespace Drupal\block;

use Drupal\Core\Entity\EntityForm;
use Drupal\Core\Entity\EntityManagerInterface;
use Drupal\Core\Form\FormState;
use Drupal\Core\Form\FormStateInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;
?>
```

###模块

模块中的类必须申明为特定的命名空间，如模块名为example_module。

```php
namespace Drupal\example_module
```

要在Drupal8中自动加载(autoload)，还要这些类归置到指定的文件夹下，一般约定在

```php
modules/example_module/src
```

举个简单的例子，如：

> NO.1

```php
// 声明命名空间
namespace Drupal\example_module

// 创建类
class Foo {}
```

这个对应的文件就是example_module/src/Foo.php。

> NO.2
```php
// 声明命名空间
namespace Drupal\example_module\Foo

// 创建类
class Bar {}
```

这个对应的文件就是example_module/src/Foo/Bar.php。

以上，就是对Drupal8的命名空间做简单的梳理和介绍。欢迎拍砖交流！

参考文章：
1、https://www.drupal.org/node/1353118
2、https://www.drupal.org/node/2156625