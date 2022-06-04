title: Drupal8 Configuration Entity
date: 2015-12-21 10:39:45
categories: Drupal8
tags: [entity]
---

Drupal8的实体由2部分组成，配置实体(configuration entities)和内容实体(content entities)。如图：
![Entity](https://static.verycloud.cn/sites/default/files/pic/image/20151222/20151222113456_92999.png)

本节源代码见 https://github.com/RamboLau/drupal8-demos/tree/master/config_entities_example

内容实体见：http://verynull.com/2015/12/22/drupal8-content-entity/

###一、配置实体接口(configuration entity interface)

在src目录下touch DemoInterface.php，DemoInterface继承并实现[ConfigEntityInterface](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Config%21Entity%21ConfigEntityInterface.php/interface/ConfigEntityInterface/8)。

```php
<?php

/**
 * @file
 * Contains \Drupal\demo\DemoInterface.
 */

namespace Drupal\demo;

use Drupal\Core\Config\Entity\ConfigEntityInterface;

/**
 * Provides an interface defining a demo entity type.
 */
interface DemoInterface extends ConfigEntityInterface {
  // Add get/set methods for your configuration properties here.
}

?>
```

###二、配置实体类(configuration entity class)
在src目录下创建Entity目录，并touch src/Entity/Demo.php，在这个文件中我们将实现DemoEntity。

```php
<?php

/**
 * @file
 * Contains \Drupal\demo\Entity\Demo.
 */

namespace Drupal\demo\Entity;

use Drupal\Core\Config\Entity\ConfigEntityBase;
use Drupal\demo\DemoInterface;

/**
 * Defines the Demo entity.
 *
 * @ConfigEntityType(
 *   id = "demo",
 *   label = @Translation("Demo"),
 *   fieldable = FALSE,
 *   handlers = {
 *     "list_builder" = "Drupal\demo\DemoListBuilder",
 *     "form" = {
 *       "add" = "Drupal\demo\Form\DemoForm",
 *       "edit" = "Drupal\demo\Form\DemoForm",
 *       "delete" = "Drupal\demo\Form\DemoDeleteForm"
 *     }
 *   },
 *   config_prefix = "demo",
 *   admin_permission = "administer site configuration",
 *   entity_keys = {
 *     "id" = "id",
 *     "label" = "name"
 *   },
 *   links = {
 *     "edit-form" = "/admin/structure/demos/{demo}",
 *     "delete-form" = "/admin/structure/demos/{demo}/delete"
 *   }
 * )
 */
class Demo extends ConfigEntityBase implements DemoInterface {

  /**
   * The ID of the demo.
   *
   * @var string
   */
  public $id;

  /**
   * The demo name.
   *
   * @var string
   */
  public $name;

  /**
   * The demo sex.
   *
   * @var string
   */
  public $sex;

}

?>
```

上面定义Entity的方式熟悉吧！对，就是注解annotation。

@ConfigEntityType告诉Drupal这是一个配置实体。参数如下：
1、lable：翻译系统的标签
2、fieldable: 附加字段，配置实体一般为FALSE，内容实体一般为TRUE
3、handlers：管理该实体所需要的类
4、list_builder：提供该实体的管理界面接口
5、config_prefix：配置的前缀，一般用于schema
6、admin_permission：管理员权限
7、entity_keys：主要实体属性的集合
8、links：编辑和删除的路径，在demo.routing.yml中定义

###三、实体表单(entity form)

接下来我们来实现在Entity中的定义的表单，如add, edit, delete。

####1、mkdir -p src/Form，并touch src/Form/DemoForm.php。
在DemoForm主要实现2个方法，form()和save()。

```php
<?php

/**
 * @file
 * Contains \Drupal\demo\Form\DemoForm.
 */

namespace Drupal\demo\Form;

use Drupal\Core\Entity\EntityForm;
use Drupal\Core\Entity\EntityInterface;
use Drupal\Core\Entity\EntityTypeInterface;
use Symfony\Component\DependencyInjection\ContainerInterface;
use Drupal\Core\Entity\Query\QueryFactory;
use Drupal\Core\Form\FormStateInterface;

/**
 * Class DemoForm
 *
 * Form class for adding/editing demo config entities.
 */
class DemoForm extends EntityForm {

  /**
   * @param \Drupal\Core\Entity\Query\QueryFactory $entity_query
   *   The entity query.
   */
  public function __construct(QueryFactory $entity_query) {
    $this->entityQuery = $entity_query;
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('entity.query')
    );
  }

  /**
   * {@inheritdoc}
   */
  public function form(array $form, FormStateInterface $form_state) {
    $form = parent::form($form, $form_state);
    $demo = $this->entity;

    // Change page title for the edit operation
    if ($this->operation == 'edit') {
      $form['#title'] = $this->t('Edit demo: @name', array('@name' => $demo->name));
    }

    // The demo name.
    $form['name'] = array(
      '#type' => 'textfield',
      '#title' => $this->t('Name'),
      '#maxlength' => 255,
      '#default_value' => $demo->name,
      '#description' => $this->t("demo name."),
      '#required' => TRUE,
    );

    // The unique machine name of the demo.
    $form['id'] = array(
      '#type' => 'machine_name',
      '#maxlength' => EntityTypeInterface::BUNDLE_MAX_LENGTH,
      '#default_value' => $demo->id,
      '#disabled' => !$demo->isNew(),
      '#machine_name' => array(
        'source' => array('name'),
        'exists' => 'demo_load'
      ),
    );

    // The sex.
    $form['sex'] = array(
      '#type' => 'select',
      '#options' => array(
        'Man' => 'Man',
        'Woman' => 'Woman',
      ),
      '#title' => $this->t('Sex'),
      '#maxlength' => 255,
      '#default_value' => $demo->sex,
      '#description' => $this->t("sex"),
      '#required' => TRUE,
    );
    return $form;
  }

  /**
   * {@inheritdoc}
   */
  public function save(array $form, FormStateInterface $form_state) {
    $demo = $this->entity;
    $status = $demo->save();

    if ($status) {
      // Setting the success message.
      drupal_set_message($this->t('Saved the demo: @label.', array(
        '@label' => $demo->name,
      )));
    }
    else {
      drupal_set_message($this->t('The @label demo was not saved.', array(
        '@label' => $demo->name,
      )));
    }

    $form_state->setRedirect('demo.list');
  }
} 
?>
```

如图：

![DemoForm](https://static.verycloud.cn/sites/default/files/pic/image/20151222/20151222212259_77460.png)

####2、touch src/Form/DemoFormDelete.php。

DemoFormDelete继承[EntityConfirmFormBase](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Core%21Entity%21EntityConfirmFormBase.php/class/EntityConfirmFormBase/8)。

```php
<?php

/**
 * @file
 * Contains \Drupal\demo\Form\DemoDeleteForm.
 */

namespace Drupal\demo\Form;

use Drupal\Core\Entity\EntityConfirmFormBase;
use Drupal\Core\Url;
use Drupal\Core\Form\FormStateInterface;


/**
 * Form that handles the removal of demo entities.
 */
class DemoDeleteForm extends EntityConfirmFormBase {

  /**
   * {@inheritdoc}
   */
  public function getQuestion() {
    return $this->t('Are you sure you want to delete this demo: @name?', array('@name' => $this->entity->name));
  }

  /**
   * {@inheritdoc}
   */
  public function getCancelUrl() {
    return new Url('demo.list');
  }

  /**
   * {@inheritdoc}
   */
  public function getConfirmText() {
    return $this->t('Delete');
  }

  /**
   * {@inheritdoc}
   */
  public function submitForm(array &$form, FormStateInterface $form_state) {
    // Delete and set message
    $this->entity->delete();
    drupal_set_message($this->t('The demo @name has been deleted.', array('@name' => $this->entity->name)));

    $form_state->setRedirectUrl($this->getCancelUrl());
  }
}
?>
```

如图：
![FormDelete](https://static.verycloud.cn/sites/default/files/pic/image/20151222/20151222212429_43428.png)


###四、list builder

接下来就该创建管理界面啦！

在src目录下，touch DemoListBuilder.php

DemoListBuilder继承[ConfigEntityListBuilder](https://api.drupal.org/api/drupal/core!lib!Drupal!Core!Config!Entity!ConfigEntityListBuilder.php/class/ConfigEntityListBuilder/8)，实现了三个方法。

1、buildHeader：创建表格header
2、buildRow：创建表格行
3、render：重写表格输出的内容，类似于Drupal7的hook


```php
<?php

/**
 * @file
 *
 * Contains Drupal\demo\DemoListBuilder
 */

namespace Drupal\demo;

use Drupal\Core\Config\Entity\ConfigEntityListBuilder;
use Drupal\Core\Entity\EntityInterface;

class DemoListBuilder extends ConfigEntityListBuilder {

  /**
   * {@inheritdoc}
   */
  public function buildHeader() {
    $header['label'] = $this->t('Name');
    $header['sex'] = $this->t('Sex');
    return $header + parent::buildHeader();
  }

  /**
   * {@inheritdoc}
   */
  public function buildRow(EntityInterface $entity) {
    // Label
    $row['label'] = $this->getLabel($entity);
    // Sex
    $row['sex'] = $entity->sex;
    return $row + parent::buildRow($entity);
  }

  /**
   * {@inheritdoc}
   */
  public function render() {
    $build = parent::render();
    $build['#empty'] = $this->t('There are no data available.');
    return $build;
  }

}
?>
```

如图：
![ListBuilder](https://static.verycloud.cn/sites/default/files/pic/image/20151222/20151222212547_35896.png)

###五、schema

```bash
mkdir config/schema
touch demo.schema.yml
```

```php
# Schema for the configuration files of the Demo module.

demo.demo.*:
  type: mapping
  label: 'Demo'
  mapping:
    id:
      type: string
      label: 'Demo identifier'
    uuid:
      type: string
      label: 'UUID'
    name:
      type: label
      label: 'Name'
    sex:
      type: string
      label: 'Sex'
      translatable: true
```

schema的命名规则：

```bash
(demo module).(demo configuration entity type).(all demo configuration entities).
```

uuid在这里也可以不用定义，drupal默认会添加。

###六、routing

定义一些路由：

```bash
demo.list:
  path: '/admin/structure/demos'
  defaults:
    _entity_list: 'demo'
    _title: 'Demos'
  requirements:
    _permission: 'administer site configuration'
demo.add:
  path: '/admin/structure/demos/add'
  defaults:
    _entity_form: 'demo.add'
    _title: 'Add a new demo'
  requirements:
    _permission: 'administer site configuration'
entity.demo.edit_form:
  path: '/admin/structure/demos/{demo}'
  defaults:
    _entity_form: 'demo.edit'
    _title: 'Edit demo'
  requirements:
    _permission: 'administer site configuration'
entity.demo.delete_form:
  path: '/admin/structure/demos/{demo}/delete'
  defaults:
    _entity_form: 'demo.delete'
    _title: 'Delete demo'
  requirements:
    _permission: 'administer site configuration'
```

###七、 links

####1、显示添加按钮在网页上，需要在demo.links.action.yml中定义

```bash
demo.add:
  route_name: 'demo.add'
  title: 'Add demo'
  appears_on:
    - demo.list
```

![demo.add](https://static.verycloud.cn/sites/default/files/pic/image/20151222/20151222212157_21914.png)

####2、在admin/structure页面显示demo，需要在demo.links.menu.yml中定义

```bash
demo.list:
  title: Demo
  description: 'Administrator the demo entities'
  parent: system.admin_structure
  route_name: demo.list
```

如图：
![demo.list](https://static.verycloud.cn/sites/default/files/pic/image/20151222/20151222212041_77165.png)


参考文章：
1、http://www.slideshare.net/andypost/d8-entity
2、http://www.sitepoint.com/drupal-8-version-entityfieldquery/
3、https://www.drupal.org/node/1809494