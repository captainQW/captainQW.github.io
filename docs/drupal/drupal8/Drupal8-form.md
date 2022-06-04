title: Drupal8表单(form)
date: 2015-11-12 17:15:23
categories: Drupal8
tags: [form]
---

这里简单做下Entity Form的实现，包括调用模板输出form.
详情请访问俺的github，链接地址： https://github.com/aboutblankchina/drupal8_test


1、首先创建相关文件
[$install_path]/modules/hello_world/lib/Drupal/hello_world/Entity/Hello.php
注：该文件定义实例（Entity）

如下：
```php
<?php

/**
 * @file
 * Definition of Drupal\hello_world\Entity\Hello.
 */

namespace Drupal\hello_world\Entity;

use Drupal\Core\Entity\ContentEntityBase;
use Drupal\Core\Entity\EntityStorageControllerInterface;
use Drupal\Core\Field\FieldDefinition;
use Drupal\Core\Language\Language;
use Drupal\Core\Session\AccountInterface;

/**
 * Defines the hello entity class.
 *
 * @EntityType(
 *   id = "hello",
 *   label = @Translation("Hello"),
 *   controllers = {
 *     "storage" = "Drupal\Core\Entity\FieldableDatabaseStorageController",
 *     "view_builder" = "Drupal\Core\Entity\EntityViewBuilder",
 *     "form" = {
 *       "default" = "Drupal\hello_world\HelloWorldFormController",
 *     },
 *   },
 *   base_table = "node",
 *   entity_keys = {
 *     "id" = "nid",
 *     "label" = "title",
 *     "uuid" = "uuid"
 *   },
 *   links = {
 *     "edit-form" = "hello.edit",
 *   }
 * )
 */
class Hello extends ContentEntityBase {

}
```

需要说明的是，上面的一部分注释(Annotation)是必须使用的，这是symfony的标准。
Symfony2框架中大量使用了Annotation：从缓存的定义到路由的配置，甚至到表结构的定义，处处都使用了Annotation功能。你甚至可以根据规范编写自己的Annotation。所以在使用Symfony2开发程序时，复杂的逻辑会变成一行行清晰的注释，程序的流程控制将变得非常简单。
详细见 https://api.drupal.org/api/drupal/core!modules!system!system.api.php/group/annotation/8 。

2、在hello_world.routing.yml中增加一行

```php
hello.edit:
  path: '/hello/edit'
  defaults:
    _entity_form: 'hello.default'
  requirements:
    _permission: 'access hello world'
```

3、创建文件 lib/Drupal/hello_world/HelloWorldFormController.php

```php
<?php

/**
 * @file
 * Definition of Drupal\hello_world\HelloWorldFormController.
 */

namespace Drupal\hello_world;

use Drupal\Component\Utility\NestedArray;
use Drupal\Core\Datetime\DrupalDateTime;
use Drupal\Core\Entity\ContentEntityFormController;
use Drupal\Core\Language\Language;
use Drupal\Component\Utility\String;

/**
 * Form controller for the node edit forms.
 */
class HelloWorldFormController extends ContentEntityFormController {
  
  public function form(array $form, array &$form_state) {
    $form = parent::form($form, $form_state); $entity = $this->entity;

    $form['#title'] = '这里写入网页标题';

    $form['title'] = array(
      '#type' => 'textfield',
      '#title' => t('Title'),
      '#default_value' => $entity->title->value,
    );

    // 如果要用模板输出。则这样调用
    $form['#theme'] = 'hello_world_form';
    return $form;
  }

  public function save(array $form, array &$form_state) { 
    $entity = $this->entity;
    $entity->save();
    drupal_set_message("hello saved");
  }

  public function delete(array $form, array &$form_state) { 
    $entity = $this->entity;
    $entity->delete();
    drupal_set_message("hello has been deleted."); 
    $form_sate['redirect_route']['route_name'] = '<front>';
  }

}

```

4、如果要使用模板输出，则要定义hook_theme

```php
/**
 * Implements hook_theme().
 */
function hello_world_theme() {
  return array(
    'hello_world_form' => array(
      'render element' => 'form',
      'template' => 'hello-world-form',
    ),
  );
}
```

访问http://localhost/drupal8/hello/edit就可以看到表单了。