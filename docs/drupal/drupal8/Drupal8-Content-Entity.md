title: Drupal8 Content Entity
date: 2015-12-22 17:43:49
categories: Drupal8
tags: [entity]
---

这篇文章是[Drupal8-configuration-Entity](http://verynull.com/2015/12/21/Drupal8-configuration-Entity/)的姊妹篇。主要介绍如何创建内容实体，并在界面上管理。

content entity最大的优势在于可自定义字段。

本文github：https://github.com/RamboLau/drupal8-demos/tree/master/content_entity_example

###一、创建模块

```bash
mkdir content_entity_example

touch content_entity_example.info.yml
```

编辑内容为：

```bash
name: content_entity_example
type: module
description: Provides ContentEntityExampleOnlineMessage entity.
core: 8.x
package: Other
```

###二、content_entity_example.routing.yml

编辑内容为：

```bash

# OnlineMessage routing definition
entity.online_message.canonical:
  path: '/admin/structure/online_message/{online_message}'
  defaults:
    _entity_view: 'online_message'
    _title: 'Online message'
  requirements:
    _entity_access: 'online_message.view'
  options:
    _admin_route: TRUE

entity.online_message.collection:
  path: '/admin/structure/online_message'
  defaults:
    _entity_list: 'online_message'
    _title: 'Online message list'
  requirements:
    _permission: 'view online message entities'
  options:
    _admin_route: TRUE

entity.online_message.add_form:
  path: '/admin/structure/online_message/add'
  defaults:
    _entity_form: online_message.add
    _title: 'Add Online message'
  requirements:
    _permission: 'add online message entities'
  options:
    _admin_route: TRUE

entity.online_message.edit_form:
  path: '/admin/structure/online_message/{online_message}/edit'
  defaults:
    _entity_form: online_message.edit
    _title: 'Edit Online message'
  requirements:
    _permission: 'edit online message entities'
  options:
    _admin_route: TRUE

entity.online_message.delete_form:
  path: '/admin/structure/online_message/{online_message}/delete'
  defaults:
    _entity_form: online_message.delete
    _title: 'Delete Online message'
  requirements:
    _permission: 'delete online message entities'
  options:
    _admin_route: TRUE

online_message.settings:
  path: '/admin/structure/online_message/settings'
  defaults:
   _form: '\Drupal\content_entity_example\Form\OnlineMessageSettingsForm'
   _title: 'Online message settings'
  requirements:
    _permission: 'administer online message entities'
  options:
    _admin_route: TRUE

```

###三、content_entity_example.links.menu.yml

编辑内容为：

```bash
# Online message menu items definition
entity.online_message.collection:
  title: 'Online message list'
  route_name: entity.online_message.collection
  description: 'List Online message entities'
  parent: system.admin_structure
  weight: 100

online_message.admin.structure.settings:
  title: Online message settings
  description: 'Configure Online message entities'
  route_name: online_message.settings
  parent: system.admin_structure

```

![menu](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223140428_47031.png)

###四、content_entity_example.links.action.yml

编辑内容为：

```bash
entity.online_message.add_form:
  route_name: entity.online_message.add_form
  title: 'Add Online message'
  appears_on:
    - entity.online_message.collection
    - entity.online_message.canonical

```

![action](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223140610_19940.png)

###五、content_entity_example.links.task.yml

编辑内容为：

```bash
# Online message routing definition
online_message.settings_tab:
  route_name: online_message.settings
  title: 'Settings'
  base_route: online_message.settings
entity.online_message.canonical:
  route_name: entity.online_message.canonical
  base_route: entity.online_message.canonical
  title: 'View'

entity.online_message.edit_form:
  route_name: entity.online_message.edit_form
  base_route: entity.online_message.canonical
  title: Edit

entity.online_message.delete_form:
  route_name:  entity.online_message.delete_form
  base_route:  entity.online_message.canonical
  title: Delete
  weight: 10

```

###六、Entity类

####1、src/OnlineMessageInterface.php

注：通过EntityOwnerInterface接口可获取其他的一些函数，如

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\content_entity_example\OnlineMessageInterface.
 */

namespace Drupal\content_entity_example;

use Drupal\Core\Entity\ContentEntityInterface;
use Drupal\Core\Entity\EntityChangedInterface;
use Drupal\Core\Entity\EntityTypeInterface;
use Drupal\user\EntityOwnerInterface;

/**
 * Provides an interface for defining Online message entities.
 *
 * @ingroup content_entity_example
 */
interface OnlineMessageInterface extends ContentEntityInterface, EntityChangedInterface, EntityOwnerInterface {
  // Add get/set methods for your configuration properties here.

  /**
   * Gets the Online message name.
   *
   * @return string
   *   Name of the Online message.
   */
  public function getName();

  /**
   * Sets the Online message name.
   *
   * @param string $name
   *   The Online message name.
   *
   * @return \Drupal\content_entity_example\OnlineMessageInterface
   *   The called Online message entity.
   */
  public function setName($name);

  /**
   * Gets the Online message creation timestamp.
   *
   * @return int
   *   Creation timestamp of the Online message.
   */
  public function getCreatedTime();

  /**
   * Sets the Online message creation timestamp.
   *
   * @param int $timestamp
   *   The Online message creation timestamp.
   *
   * @return \Drupal\content_entity_example\OnlineMessageInterface
   *   The called Online message entity.
   */
  public function setCreatedTime($timestamp);

  /**
   * Returns the Online message published status indicator.
   *
   * Unpublished Online message are only visible to restricted users.
   *
   * @return bool
   *   TRUE if the Online message is published.
   */
  public function isPublished();

  /**
   * Sets the published status of a Online message.
   *
   * @param bool $published
   *   TRUE to set this Online message to published, FALSE to set it to unpublished.
   *
   * @return \Drupal\content_entity_example\OnlineMessageInterface
   *   The called Online message entity.
   */
  public function setPublished($published);

}

?>
```

####2、src/Entity/OnlineMessage.php

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\content_entity_example\Entity\OnlineMessage.
 */

namespace Drupal\content_entity_example\Entity;

use Drupal\Core\Entity\EntityStorageInterface;
use Drupal\Core\Field\BaseFieldDefinition;
use Drupal\Core\Entity\ContentEntityBase;
use Drupal\Core\Entity\EntityChangedTrait;
use Drupal\Core\Entity\EntityTypeInterface;
use Drupal\content_entity_example\OnlineMessageInterface;
use Drupal\user\UserInterface;

/**
 * Defines the Online message entity.
 *
 * @ingroup content_entity_example
 *
 * @ContentEntityType(
 *   id = "online_message",
 *   label = @Translation("Online message"),
 *   handlers = {
 *     "view_builder" = "Drupal\Core\Entity\EntityViewBuilder",
 *     "list_builder" = "Drupal\content_entity_example\OnlineMessageListBuilder",
 *     "views_data" = "Drupal\content_entity_example\Entity\OnlineMessageViewsData",
 *
 *     "form" = {
 *       "default" = "Drupal\content_entity_example\Form\OnlineMessageForm",
 *       "add" = "Drupal\content_entity_example\Form\OnlineMessageForm",
 *       "edit" = "Drupal\content_entity_example\Form\OnlineMessageForm",
 *       "delete" = "Drupal\content_entity_example\Form\OnlineMessageDeleteForm",
 *     },
 *     "access" = "Drupal\content_entity_example\OnlineMessageAccessControlHandler",
 *   },
 *   base_table = "online_message",
 *   admin_permission = "administer OnlineMessage entities",
 *   entity_keys = {
 *     "id" = "id",
 *     "label" = "name",
 *     "uuid" = "uuid",
 *     "uid" = "user_id",
 *     "langcode" = "langcode",
 *   },
 *   links = {
 *     "canonical" = "/admin/online_message/{online_message}",
 *     "edit-form" = "/admin/online_message/{online_message}/edit",
 *     "delete-form" = "/admin/online_message/{online_message}/delete"
 *   },
 *   field_ui_base_route = "online_message.settings"
 * )
 */
class OnlineMessage extends ContentEntityBase implements OnlineMessageInterface {
  use EntityChangedTrait;
  /**
   * {@inheritdoc}
   */
  public static function preCreate(EntityStorageInterface $storage_controller, array &$values) {
    parent::preCreate($storage_controller, $values);
    $values += array(
      'user_id' => \Drupal::currentUser()->id(),
    );
  }

  /**
   * {@inheritdoc}
   */
  public function getName() {
    return $this->get('name')->value;
  }

  /**
   * {@inheritdoc}
   */
  public function setName($name) {
    $this->set('name', $name);
    return $this;
  }

  /**
   * {@inheritdoc}
   */
  public function getCreatedTime() {
    return $this->get('created')->value;
  }

  /**
   * {@inheritdoc}
   */
  public function setCreatedTime($timestamp) {
    $this->set('created', $timestamp);
    return $this;
  }

  /**
   * {@inheritdoc}
   */
  public function getOwner() {
    return $this->get('user_id')->entity;
  }

  /**
   * {@inheritdoc}
   */
  public function getOwnerId() {
    return $this->get('user_id')->target_id;
  }

  /**
   * {@inheritdoc}
   */
  public function setOwnerId($uid) {
    $this->set('user_id', $uid);
    return $this;
  }

  /**
   * {@inheritdoc}
   */
  public function setOwner(UserInterface $account) {
    $this->set('user_id', $account->id());
    return $this;
  }

  /**
   * {@inheritdoc}
   */
  public function isPublished() {
    return (bool) $this->getEntityKey('status');
  }

  /**
   * {@inheritdoc}
   */
  public function setPublished($published) {
    $this->set('status', $published ? NODE_PUBLISHED : NODE_NOT_PUBLISHED);
    return $this;
  }

  /**
   * {@inheritdoc}
   */
  public static function baseFieldDefinitions(EntityTypeInterface $entity_type) {
    $fields['id'] = BaseFieldDefinition::create('integer')
      ->setLabel(t('ID'))
      ->setDescription(t('The ID of the Online message entity.'))
      ->setReadOnly(TRUE);
    $fields['uuid'] = BaseFieldDefinition::create('uuid')
      ->setLabel(t('UUID'))
      ->setDescription(t('The UUID of the Online message entity.'))
      ->setReadOnly(TRUE);

    $fields['user_id'] = BaseFieldDefinition::create('entity_reference')
      ->setLabel(t('Authored by'))
      ->setDescription(t('The user ID of author of the Online message entity.'))
      ->setRevisionable(TRUE)
      ->setSetting('target_type', 'user')
      ->setSetting('handler', 'default')
      ->setDefaultValueCallback('Drupal\node\Entity\Node::getCurrentUserId')
      ->setTranslatable(TRUE)
      ->setDisplayOptions('view', array(
        'label' => 'hidden',
        'type' => 'author',
        'weight' => 0,
      ))
      ->setDisplayOptions('form', array(
        'type' => 'entity_reference_autocomplete',
        'weight' => 5,
        'settings' => array(
          'match_operator' => 'CONTAINS',
          'size' => '60',
          'autocomplete_type' => 'tags',
          'placeholder' => '',
        ),
      ))
      ->setDisplayConfigurable('form', TRUE)
      ->setDisplayConfigurable('view', TRUE);

    $fields['name'] = BaseFieldDefinition::create('string')
      ->setLabel(t('Name'))
      ->setDescription(t('The name of the Online message entity.'))
      ->setSettings(array(
        'max_length' => 50,
        'text_processing' => 0,
      ))
      ->setDefaultValue('')
      ->setDisplayOptions('view', array(
        'label' => 'above',
        'type' => 'string',
        'weight' => -4,
      ))
      ->setDisplayOptions('form', array(
        'type' => 'string_textfield',
        'weight' => -4,
      ))
      ->setDisplayConfigurable('form', TRUE)
      ->setDisplayConfigurable('view', TRUE);

    $fields['status'] = BaseFieldDefinition::create('boolean')
      ->setLabel(t('Publishing status'))
      ->setDescription(t('A boolean indicating whether the Online message is published.'))
      ->setDefaultValue(TRUE);

    $fields['langcode'] = BaseFieldDefinition::create('language')
      ->setLabel(t('Language code'))
      ->setDescription(t('The language code for the Online message entity.'));

    $fields['created'] = BaseFieldDefinition::create('created')
      ->setLabel(t('Created'))
      ->setDescription(t('The time that the entity was created.'));

    $fields['changed'] = BaseFieldDefinition::create('changed')
      ->setLabel(t('Changed'))
      ->setDescription(t('The time that the entity was last edited.'));

    return $fields;
  }

}
?>
```

注解的相关名词解释：

1、id：实体类型唯一标识符，一般遵循'modulename_xyz'的命名规则。

2、label：易读的实体名称。

3、handlers：不同的任务所使用的处理程序，一般来说，标准的处理方法有：
	1)、view_builder：由routing.yml中的'_entity_view'所调用。

	2)、list_builder：继承entityListBuilder，展示在线留言的列表。

	3)、form：由routing.yml中的'_entity_form'所调用。

	4)、access：设定访问权限

4、base_table：定义存储数据的表，必须唯一哦！schema由BaseFieldDefinitions决定，该表会在模块启用的时候自动创建。

5、fieldable：可以添加自定义字段，只有内容实体才有这个功能

6、entity_keys：访问fields的字段，如nid或uid

7、links：操作链接，如'edit-form'和'delete-form'。在列表页可以看到相应的操作按钮。

###七、控制器类

####1、src/Form/OnlineMessageForm.php

由routing中定义的'_entity_form'调用。

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\content_entity_example\Form\OnlineMessageForm.
 */

namespace Drupal\content_entity_example\Form;

use Drupal\Core\Entity\ContentEntityForm;
use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Language\Language;

/**
 * Form controller for Online message edit forms.
 *
 * @ingroup content_entity_example
 */
class OnlineMessageForm extends ContentEntityForm {
  /**
   * {@inheritdoc}
   */
  public function buildForm(array $form, FormStateInterface $form_state) {
    /* @var $entity \Drupal\content_entity_example\Entity\OnlineMessage */
    $form = parent::buildForm($form, $form_state);
    $entity = $this->entity;

    $form['langcode'] = array(
      '#title' => $this->t('Language'),
      '#type' => 'language_select',
      '#default_value' => $entity->langcode->value,
      '#languages' => Language::STATE_ALL,
    );

    return $form;
  }

  /**
   * {@inheritdoc}
   */
  public function submit(array $form, FormStateInterface $form_state) {
    // Build the entity object from the submitted values.
    $entity = parent::submit($form, $form_state);

    return $entity;
  }

  /**
   * {@inheritdoc}
   */
  public function save(array $form, FormStateInterface $form_state) {
    $entity = $this->entity;
    $status = $entity->save();

    switch ($status) {
      case SAVED_NEW:
        drupal_set_message($this->t('Created the %label Online message.', [
          '%label' => $entity->label(),
        ]));
        break;

      default:
        drupal_set_message($this->t('Saved the %label Online message.', [
          '%label' => $entity->label(),
        ]));
    }
    $form_state->setRedirect('entity.online_message.edit_form', ['online_message' => $entity->id()]);
  }

}

?>
```

![add_form](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223143411_52761.png)

####2、src/Form/OnlineMessageDeleteForm.php

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\content_entity_example\Form\OnlineMessageDeleteForm.
 */

namespace Drupal\content_entity_example\Form;

use Drupal\Core\Entity\ContentEntityConfirmFormBase;
use Drupal\Core\Form\FormStateInterface;
use Drupal\Core\Url;

/**
 * Provides a form for deleting Online message entities.
 *
 * @ingroup content_entity_example
 */
class OnlineMessageDeleteForm extends ContentEntityConfirmFormBase {
  /**
   * {@inheritdoc}
   */
  public function getQuestion() {
    return $this->t('Are you sure you want to delete entity %name?', array('%name' => $this->entity->label()));
  }

  /**
   * {@inheritdoc}
   */
  public function getCancelUrl() {
    return new Url('entity.online_message.collection');
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
    $this->entity->delete();

    drupal_set_message(
      $this->t('content @type: deleted @label.',
        [
          '@type' => $this->entity->bundle(),
          '@label' => $this->entity->label()
        ]
        )
    );

    $form_state->setRedirectUrl($this->getCancelUrl());
  }

}

?>
```

![delete_form](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223143307_47107.png)

####3、src/OnlineMessageListBuilder.php

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\content_entity_example\OnlineMessageListBuilder.
 */

namespace Drupal\content_entity_example;

use Drupal\Core\Entity\EntityInterface;
use Drupal\Core\Entity\EntityListBuilder;
use Drupal\Core\Routing\LinkGeneratorTrait;
use Drupal\Core\Url;

/**
 * Defines a class to build a listing of Online message entities.
 *
 * @ingroup content_entity_example
 */
class OnlineMessageListBuilder extends EntityListBuilder {
  use LinkGeneratorTrait;
  /**
   * {@inheritdoc}
   */
  public function buildHeader() {
    $header['id'] = $this->t('Online message ID');
    $header['name'] = $this->t('Name');
    return $header + parent::buildHeader();
  }

  /**
   * {@inheritdoc}
   */
  public function buildRow(EntityInterface $entity) {
    /* @var $entity \Drupal\content_entity_example\Entity\OnlineMessage */
    $row['id'] = $entity->id();
    $row['name'] = $this->l(
      $entity->label(),
      new Url(
        'entity.online_message.edit_form', array(
          'online_message' => $entity->id(),
        )
      )
    );
    return $row + parent::buildRow($entity);
  }

}

?>
```

![list_builder](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223143200_33896.png)

###八、Field配置

####1、src/Form/OnlineMessageSettingsForm.php

编辑内容为：

```php
<?php

/**
 * @file
 * Contains \Drupal\content_entity_example\Form\OnlineMessageSettingsForm.
 */

namespace Drupal\content_entity_example\Form;

use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;

/**
 * Class OnlineMessageSettingsForm.
 *
 * @package Drupal\content_entity_example\Form
 *
 * @ingroup content_entity_example
 */
class OnlineMessageSettingsForm extends FormBase {
  /**
   * Returns a unique string identifying the form.
   *
   * @return string
   *   The unique string identifying the form.
   */
  public function getFormId() {
    return 'OnlineMessage_settings';
  }

  /**
   * Form submission handler.
   *
   * @param array $form
   *   An associative array containing the structure of the form.
   * @param \Drupal\Core\Form\FormStateInterface $form_state
   *   The current state of the form.
   */
  public function submitForm(array &$form, FormStateInterface $form_state) {
    // Empty implementation of the abstract submit class.
  }


  /**
   * Defines the settings form for Online message entities.
   *
   * @param array $form
   *   An associative array containing the structure of the form.
   * @param \Drupal\Core\Form\FormStateInterface $form_state
   *   The current state of the form.
   *
   * @return array
   *   Form definition array.
   */
  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['OnlineMessage_settings']['#markup'] = 'Settings form for Online message entities. Manage field settings here.';
    return $form;
  }

}

?>
```

![设置](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223142648_96014.png)

![管理字段](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223142758_83626.png)

![管理表单显示](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223142828_19080.png)

![管理显示](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223142848_49971.png)

###九、注册entity

启用模块，会自动根据Entity中的配置创建online_message表。

![online_message_table](https://static.verycloud.cn/sites/default/files/pic/image/20151223/20151223140008_92664.png)

参考文章：
1、https://www.drupal.org/node/2192175
2、http://www.slideshare.net/andypost/d8-entity