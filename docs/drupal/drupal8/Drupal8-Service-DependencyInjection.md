title: Drupal8-Service-DependencyInjection
date: 2015-12-15 16:53:05
categories: Drupal8
tags: [Service_DependencyInjection]
---


先来认识几个名词：

1、Service

Drupal8的service帮助代码去耦合，提高重复利用。比如操作数据库，发送mail等。一般来说，service是一个PHP类，包含了一些全局的方法。

2、Service Container

Service Container用来管理service。service的类会被注册在container中，来控制访问权限。在自定义模块中，可以用module_name.services.yml来实例化services，进而保存进container对象中。

3、Dependency Injection

如果要传递对象给另一个对象，一般用Dependency Injection来去耦。简单点说，一个service只处理一件事情，如果它要调用另外一个service，那么后面的这个service可以注入到前面的service中。

下文我们以service表示Service，container表示Service Container，DI表示Dependency Injection。

###一、core services

Drupal8的核心servive都在[core.services.yml](https://api.drupal.org/api/drupal/core%21core.services.yml/8)中。

当然，也可以在module_name.services.yml中自定义service。

*.services.yml的写法如：

```php
  path.alias_manager:
    class: Drupal\Core\Path\AliasManager
    arguments: ['@path.crud', '@path.alias_whitelist', '@language_manager']
  breadcrumb:
    class: Drupal\Core\Breadcrumb\BreadcrumbManager
    arguments: ['@module_handler']
```

如要把别的service当作工厂使用，写法如下：

```php
  cache.entity:
    class: Drupal\Core\Cache\CacheBackendInterface
    tags:
      - { name: cache.bin }
    factory: cache_factory:get
    arguments: [entity]
```

1、第一行定义了一个唯一的service名称，如果是自定义的模块，必须以模块名作为前缀。
2、第二行定义了默认调用的类、工厂类、接口类等。
3、arguments带@表示依赖于该service。
4、factory表示调用哪个工厂方法。
5、tags标签定义了一组关联的service，或指定具有某些相似特性的service。如果定义了service标签，在service类中必须实现相应的接口，如：
	> access_check: 标明路由访问检查service
	> cache.bin: 标明cache bin service
	> event_subscriber: 标明一个事件订阅service。
	> needs_destruction: 标明一个destructor()方法，通常被调用于一个请求完成时。
	> context_provice: 标明一个block上下文提供者。
	> http_client_middleware: 标明该service提供者是一个guzzle中间件。

###二、访问service

####1、全局函数(global functions)

Drupal8的全局类提供了一些静态方法去访问一些通用的service。如Drupal::moduleHandler()将返回模块处理service，Drupal.translation()将返回翻译service。用 Drupal::service()可以获取所有已经定义的service。

如：访问一个数据库service

```php
<?php
// Returns a Drupal\Core\Database\Connection object.
$connection = \Drupal::database();
$result = $connection->select('node', 'n')
  ->fields('n', array('nid'))
  ->execute();
?>
```

如：通过\Drupal::service()访问一个date service

```php
<?php
// Returns a Drupal\Core\Datetime\Date object.
$date = \Drupal::service('date');
?>
```

如：自定义的demo模块

####2、创建demo.services.yml

```php
services:
    demo.demo_service:
        class: Drupal\demo\DemoService
```

####3、在src目录下创建DemoService.php

```php
<?php

/**
 * @file
 * Contains Drupal\demo\DemoService.
 */

namespace Drupal\demo;

class DemoService {
  
  protected $demo_value;
  
  public function __construct() {
    $this->demo_value = 'Upchuk';
  }
  
  public function getDemoValue() {
    return $this->demo_value;
  }
  
}
?>
```

可以直接这样访问demo service

```php
$service = \Drupal::service('demo.demo_service');
```

####4、重写默认service

Drupal8允许在自定义的模块中重写已存在的service。如自定义了一个名为demo的模块。

1)、在demo的src文件中创建一个名为DemoServiceProvider.php文件。

2)、DemoServiceProvider必须实现[ \Drupal\Core\DependencyInjection\ServiceModifierInterface](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21DependencyInjection%21ServiceModifierInterface.php/interface/ServiceModifierInterface/8).

3)、这个类必须包含一个方法：alter()。这个方法是告诉Drupal用你的类来替代默认的类。如：

```php
  public function alter(ContainerBuilder $container) {
    // Override the language_manager class with a new class.
    $definition = $container->getDefinition('language_manager');
    $definition->setClass('Drupal\my_module\MyLanguageManager');
  }
```

###三、通过DI访问service

这块比较重要，单独来讲一下。还是以demo模块为例。

####1、创建demo.services.yml

```php
services:
    demo.demo_service:
        class: Drupal\demo\DemoService
```

####2、在src目录下创建DemoService.php

```php
<?php

/**
 * @file
 * Contains Drupal\demo\DemoService.
 */

namespace Drupal\demo;

class DemoService {
  
  protected $demo_value;
  
  public function __construct() {
    $this->demo_value = 'Upchuk';
  }
  
  public function getDemoValue() {
    return $this->demo_value;
  }
  
}
?>
```

####3、创建demo.routing.yml

```php
demo.demo:
  path: '/demo'
  defaults:
    _controller: '\Drupal\demo\Controller\DemoController:demo'
  requirements:
    _permission: 'access content'
```

####4、在src/Controller目录下创建DemoController.php

```php
<?php

/**
 * @file
 * Contains \Drupal\demo\Controller\DemoController.
 */

namespace Drupal\demo\Controller;

use Drupal\Core\Controller\ControllerBase;
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * DemoController.
 */
class DemoController extends ControllerBase {
  
  protected $demoService;
  
  /**
   * Class constructor.
   */
  public function __construct($demoService) {
    $this->demoService = $demoService;
  }
  
  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('demo.demo_service')
    );
  }
  
  /**
   * Generates an example page.
   */
  public function demo() {
    return array(
      '#markup' => t('Hello @value!', array('@value' => $this->demoService->getDemoValue())),
    );
  }
}
?>
```

create()方法创建了一个控制器类的实例，参数为container中获取到的service。这样，DemoService实例就可以存储到$demoService中，我们就可以调用getDemoValue()这个方法了。

####5、访问/demo的时候屏幕上就会出现Hello Upchuk!



参考文章：
1、https://docs.acquia.com/articles/drupal-8-dependency-injection
2、http://katbailey.github.io/2013/05/15/dependency-injection-in-drupal-8/
3、https://www.drupal.org/node/2133171
4、http://redcrackle.com/blog/drupal-8/dependency-injection
5、https://api.drupal.org/api/drupal/core!core.api.php/group/container/8
6、http://www.sitepoint.com/building-drupal-8-module-configuration-management-service-container/
7、https://api.drupal.org/api/drupal/core%21core.api.php/group/service_tag/8