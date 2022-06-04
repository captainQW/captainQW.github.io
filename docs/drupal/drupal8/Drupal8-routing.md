title: Drupal8 Routing
date: 2015-12-23 16:01:09
categories: Drupal8
tags: [routing]
---

Drupal8的路由控制系统跟Drupal7完全不同，在drupal7中，路由一般是定义在module文件的hook_menu中，如链接、访问控制、回调方法等。而drupal8由于包含了Symfony2的路由组件（Symfony2 Routing component），一般定义在{module_name}.routing.yml中。

本文介绍如何创建一个路由控制器，并如何使用依赖注入(DI)。

下面这张图非常明了的说明了他们之间的关系。

![D8-routing](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223201522_69597.png)

本文github： https://github.com/RamboLau/drupal8-demos/tree/master/routing_example

###一、静态路由(Static Routes)

####1、Drupal7路由

在Drupal7中，通过hook_menu来管理路由，但是这就导致代码的耦合性太高，如果要提供丰富的返回格式，如JSON, XML or HTML，就无能为力了！

如：

```php
<?php
/**
 * Implements hook_menu()
 */
function routing_example_menu() {
  $items['hello'] = array(
    'title' => 'Hello World',
    'description' => "hello world page",
    'route_name' => 'helloWorldPage', 
    'type' => MENU_LOCAL_TASK,
  );
  return $items;
}

/**
 * Implements hook_permission().
 */
function routing_example_permission() {
  return array(
    'access hello world' => array(
      'title' => t('Access hello world page'),
    ),
  );
}

?>
```

####2、Drupal8路由

#####routing_example.routing.yml

```php
routing_example.example_controller_index:
  path: '/routing_example/index/hello/{name}'
  defaults:
    _controller: '\Drupal\routing_example\Controller\BaseController::index'
    _title: 'hello world'
  requirements:
    _permission: 'access content'

routing_example.example_controller_dependency_injection:
  path: '/routing_example/dependency_injection'
  defaults:
    _controller: '\Drupal\routing_example\Controller\ExampleDependencyInjectionController::index'
    _title: 'hello world'
  requirements:
    _permission: 'access content'
```

#####src/Controller/BaseController.php

这是一个普通的Controller类，通过访问/routing_example/index/hello/{name}调用index，输出Implement method: index with parameter(s): $name。

```php
<?php

/**
 * @file
 * Contains \Drupal\routing_example\Controller\BaseController.
 */

namespace Drupal\routing_example\Controller;

use Drupal\Core\Controller\ControllerBase;

/**
 * Class BaseController.
 *
 * @package Drupal\routing_example\Controller
 */
class BaseController extends ControllerBase {
  /**
   * Index.
   *
   * @return string
   *   Return Hello string.
   */
  public function index($name) {
    return [
        '#type' => 'markup',
        '#markup' => $this->t("Implement method: index with parameter(s): $name")
    ];
  }

}

?>
```

#####src/Controller/ExampleDependencyInjectionController.php

这是一个依赖注入的Controler类，我们这里以访问数据库为例。

```php
<?php

/**
 * @file
 * Contains \Drupal\routing_example\Controller\ExampleDependencyInjectionController.
 */

namespace Drupal\routing_example\Controller;

use Drupal\Core\DependencyInjection\ContainerInjectionInterface;
use Drupal\Core\Database\Connection;
use Symfony\Component\DependencyInjection\ContainerInterface;

/**
 * Class ExampleDependencyInjectionController.
 *
 * @package Drupal\routing_example\Controller
 */
class ExampleDependencyInjectionController implements  ContainerInjectionInterface {

  /**
   * The database connection.
   *
   * @var \Drupal\Core\Database\Connection;
   */
  protected $database;

  /**
   * Constructs a \Drupal\routing_example\Controller\ExampleDependencyInjectionController object.
   *
   * @param \Drupal\Core\Database\Connection $database
   *   The database connection.
   */
  public function __construct(Connection $database) {
    $this->database = $database;
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    return new static($container->get('database'));
  }

  /**
   * Displays a list of nodes.
   */
  public function index() {
    // query the database
    $result = $this->database->query('SELECT * from {node} WHERE nid = 1')->fetchAll();

    return [
      '#type' => 'markup',
      '#markup' => json_encode($result),
    ];
  }

}

?>
```

访问 http://127.0.0.1/routing_example/dependency_injection

输出：

![DI](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223205139_89300.png)

###二、动态路由(Dynamic Routes)

静态路由我们一般是在{module_name}.routing.yml中指定，那么如何定义动态路由呢？

####1、Drupal7动态路由

一般是通过foreach hook_menu实现，如：

```php
/**
 * Implements hook_menu().
 */
function routing_example_menu() {
  $items = array();
  foreach (routing_example_types() as $type) {
    $items['routing_example/add/' . $type->machine_name] = array(
      'title' => $type->title,
      'page callback' => 'routing_example_type_add_page',
      'access arguments' => 'create ' . $type->type,
    );
  }
  return $items;
}
```

####2、Drupal8动态路由

在D8中，我们推荐使用RouteSubscriber来实现动态路由。

#####routing_example.routing.yml

添加：

```bash
route_callbacks:
  - '\Drupal\routing_example\Routing\ExampleRoutes::routes'
```

#####src/Routing/ExampleRoutes.php

```php
<?php
/**
 * @file
 * Contains \Drupal\routing_example\Routing\ExampleRoutes.
 */

namespace Drupal\routing_example\Routing;

use Symfony\Component\Routing\Route;
use Symfony\Component\Routing\RouteCollection;

/**
 * Defines dynamic routes.
 */
class ExampleRoutes {

  /**
   * {@inheritdoc}
   */
  public function routes() {
    $route_collection = new RouteCollection();

    // Dynamically routing.
    $route = new Route(
      'routing_example/add/dynamic_route',
      array(
        '_controller' => '\Drupal\routing_example\Controller\ExampleDependencyInjectionController::index',
        '_title' => 'Hello'
      ),
      // Route requirements:
      array(
        '_permission'  => 'access content',
      )
    );

    // Add the route under the name 'example.content'.
    $route_collection->add('routing_example.add.dynamic_route', $route);
    return $route_collection;
  }

}

?>
```

访问：http://127.0.0.1/routing_example/add/dynamic_route

![dynamic_route](https://static.verycloud.cn/sites/default/files/pic/image/20151224/20151224154334_42518.png)

动态路由还有很多高级的玩法，比如使用Entity Storage等，后续再来介绍。

参考文章：
1、https://www.drupal.org/developing/api/8/routing
2、https://www.drupal.org/node/2116767
3、https://www.drupal.org/node/2092643
4、http://www.sitepoint.com/build-drupal-8-module-routing-controllers-menu-links/
5、http://www.slideshare.net/ygerasimov/drupal-8-routing
6、https://www.previousnext.com.au/blog/using-drupal-8s-new-route-controllers
7、https://www.previousnext.com.au/blog/dynamic-routes-drupal-8-routesubscriber
8、https://www.drupal.org/node/2122201