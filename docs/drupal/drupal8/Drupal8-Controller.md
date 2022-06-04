title: Drupal8 Controller
date: 2015-12-23 10:42:39
categories: Drupal8
tags: [controller]
---

学习了这么多的章节了，对Drupal8应该不陌生了吧！

Controller类主要用于处理相关的HTTP请求逻辑，一般在routing.yml中会指定调用哪个Controller，通常情况下，在：

```bash
{module_name}/src/Controller/ExampleController.php
```

本文github： https://github.com/RamboLau/drupal8-demos/tree/master/controller_example

###一、创建模块

```bash
mkdir controller_example

touch controller_example.info.yml
```

编辑内容为：

```bash
name: controller_example
type: module
description: Provides controller example.
core: 8.x
package: Other
```

###二、controller_example.routing.yml

编辑内容为：

```bash

controller_example.default_controller_index:
  path: '/controller_example/index/hello/{name}'
  defaults:
    _controller: '\Drupal\controller_example\Controller\DefaultController::index'
    _title: 'index page'
  requirements:
    _permission: 'access content'

controller_example.default_controller_default_page:
  path: '/controller_example/default_page/hello/{name}'
  defaults:
    _controller: '\Drupal\controller_example\Controller\DefaultController::default_page'
    _title: 'default page'
  requirements:
    _permission: 'access content'

```

###三、src/Controller/DefaultController.php

创建一个名为Default的Controller。

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\controller_example\Controller\DefaultController.
 */

namespace Drupal\controller_example\Controller;

use Drupal\Core\Controller\ControllerBase;

/**
 * Class DefaultController.
 *
 * @package Drupal\controller_example\Controller
 */
class DefaultController extends ControllerBase {
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

  /**
   * Default_page.
   *
   * @return string
   *   Return Hello string.
   */
  public function default_page($name) {
    return [
        '#type' => 'markup',
        '#markup' => $this->t("Implement method: default_page with parameter(s): $name")
    ];
  }

}

?>
```

###四、调试

打开浏览器，输入 http://127.0.0.1:8080/drupal8/controller_example/default_page/hello/this_is_a_default_page，显示：

![default_page](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223162504_21051.png)

输入 http://127.0.0.1:8080/drupal8/controller_example/index/hello/this_is_a_index_page，显示：

![index_page](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223162405_48525.png)


参考文章：
1、https://www.drupal.org/node/2116767
2、http://www.sitepoint.com/build-drupal-8-module-routing-controllers-menu-links/
3、https://www.previousnext.com.au/blog/using-drupal-8s-new-route-controllers
4、http://www.vdmi.nl/blog/using-controllers-drupal-8