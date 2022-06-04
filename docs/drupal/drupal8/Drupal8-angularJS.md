title:  Drupal8中使用AngularJS
date: 2016-06-06 11:51:00
categories: Drupal8
tags: [angularjs]
---

## Drupal8中使用AngularJS

首先我们需要一个基础模块引入并启动`angular`

OK，先创建一个模块，我们起名叫`cloud_angular`

在`cloud_angular`中，我们新建目录`libs`，里面存放所有与`angular`相关的库文件，`js`目录，里面存放angular的启动js：`app.js`

接着，定义angular的library，就像这样：

```yaml
cloud_angular.angular:
  version: VERSION
  css:
    angular:
      libs/angular-loading-bar/build/loading-bar.css: {}
  js:
    libs/angular/angular.js: {}
    libs/angular-animate/angular-animate.js: {}
    libs/angular-ui-router/release/angular-ui-router.js: {}
    libs/oclazyload/dist/ocLazyLoad.js: {}
    libs/angular-loading-bar/build/loading-bar.js: {}
    js/app.js: {}
  dependencies:
    - core/drupalSettings
    - core/drupal
```

保存为`cloud_angular.libraries.yml`文件

最后，打开`cloud_angular.module`，实现`page`的`hook`，引入上面定义的库，就像下面这样：

```php
/**
 * Implements hook_preprocess_page().
 */
function cloud_angular_preprocess_page(&$variables) {
  $variables['#attached']['library'][] = 'cloud_angular/cloud_angular.angular';
}
```

这样我们在启用`cloud_angular`模块的时候，就全局加载了`angular`

------

好了，有了`angular`的启动支撑模块，接下来看下其它模块怎么使用`angular`

举个例子，用户模块，命名：`cloud_user`，我们定义一个路由`profile/console`，用于显示用户的控制台首页(俗称dashboard)

按照drupa8的路由定义规则，我们需要新建一个文件`cloud_user.routing.yml`定义路由：

```yaml
cloud_user.console:
  path: '/profile/console'
  defaults:
    _controller: '\Drupal\cloud_user\Controller\UserController::console'
    _title: '控制台首页'
  requirements:
    _permission: 'access content'
```

接着，在`cloud_user`模块下新建`src`目录及其子目录`Controller`，里面存放路由指定的控制器文件`UserController.php`，就像下面这样：

![cloud_user模块目录解构](https://static.verycloud.cn/sites/default/files/pic/image/20160606/dir.png)


控制器代码如下：

```php
<?php
/**
 * @file
 * @contains \Drupal\cloud_user\Controller\UserController.
 */

namespace Drupal\cloud_user\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\Database\Connection; 
use Symfony\Component\DependencyInjection\ContainerInterface;

class UserController extends ControllerBase {

  protected $database;

  protected $userPath;

  function __construct($database) {
    $this->database = $database;
    $this->userPath = drupal_get_path('module', 'cloud_user');
  }

  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('database')
    );
  }

  function console() {
    $build = array(
      '#theme' => 'cloud_common',
    );
    // 这里我们把用户模块的路径传递到前端
    $build['#attached']['drupalSettings']['userPath'] = $this->userPath;
    return $build;
  }
}
```

`cloud_common`的theme模版`cloud-common.html.twig`统一输出是

```twig
<div ui-view></div>
```

用于angular前端路由加载模版

接着，在`cloud_user`模块下新建`scripts`目录及其子目录`controller`和`directive`，对了，在`scripts`目录下新建文件`user.route.js`，用于定义用户模块的前端路由：

```javascript
(function(angular, drupalSettings) {
  'use strict';
  /**
   * @name verycloud
   * @description all user's ng route here.
   *
   */
  var verycloud = angular.module('verycloud');
  verycloud.config(['$stateProvider', function($stateProvider) {
    $stateProvider
      .state('dashboard', {
        url: '/profile',
        templateUrl: 'dashboard/container.html',
        resolve: {
          loadMyFile: function($ocLazyLoad) {
            return $ocLazyLoad.load({
              name: 'console',
              files: [
                drupalSettings.userPath + '/scripts/directive/header/header.js'
              ]
            })
          }
        },
        abstract: true
      })
      .state('dashboard.console', {
        url: '/console',
        templateUrl: 'dashboard/dashboard.html',
        controller: 'ConsoleCtrl',
        resolve: {
          loadMyFile: function($ocLazyLoad) {
            return $ocLazyLoad.load({
              name: 'console',
              files: [
                drupalSettings.userPath + '/scripts/controller/consoleController.js'
              ]
            })
          }
        }
      });
  }]);
})(angular, drupalSettings);
```

接下来将这个路由文件定义成库，保存为`cloud_user.libraries.yml`，

```yaml
cloud_user.user_route:
  version: VERSION
  js:
    scripts/user.route.js: {}
```

最后，回到`angular`的支撑模块`cloud_angular`，打开`cloud_angular.module`文件，在`hook_page`中引入用户模块的js库文件，就像下面这样：

```php
/**
 * Implements hook_preprocess_page().
 */
function cloud_angular_preprocess_page(&$variables) {
  $variables['#attached']['library'][] = 'cloud_angular/cloud_angular.angular';
  $variables['#attached']['library'][] = 'cloud_user/cloud_user.user_route';
}
```

启用`cloud_user`模块，尝试访问'profile/console'
