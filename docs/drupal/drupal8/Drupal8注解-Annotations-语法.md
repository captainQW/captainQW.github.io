title: Drupal8注解(Annotations)语法
date: 2015-12-12 10:35:46
categories: Drupal8
tags: [annotation]
---

写过java代码的同学，相信大家对注解都不陌生。有些PHP框架也是支持注解的，如Symfony2使用注解定义路由规则，Doctrine使用注解来添加ORM元数据。PHP注解RFC见[rfc-annotations](https://wiki.php.net/rfc/annotations "rfc")。

Drupal8的注解依照[Doctrine](http://docs.doctrine-project.org/projects/doctrine-common/en/latest/reference/annotations.html)的规范。

本文主要对Drupal8中所用到的注解做分析，欢迎拍砖交流！

###一、注解的争议
目前对注解的争议主要有这么几点。
####1、如何区分注解和注释？
####2、为什么要把业务逻辑放在注释中？难道他们不是一个核心语言语义的一部分？
####3、注解和注释代码容易混淆，如果开发人员忘记定义了注解，程序并不会报错，但是可能不会像预期的工作。
####4、注解不好测试、调试。

###二、注解的优势
####1、不用写很多代码就可以实现注入。如：

我们要把weapon对象注入到soldier实例中。

```php
class Weapon {  
    public function shoot() {
        print "... shooting ...";
    }
}

class Soldier {  
    private $weapon;

    public function setWeapon($weapon) {
        $this->weapon = $weapon;
    }

    public function fight() {
        $this->weapon->shoot();
    }
}
```

要实现这个功能，要这样写代码：

```php
$weapon = new Weapon();

$soldier = new Soldier();
$soldier->setWeapon($weapon); 
$soldier->fight();
```

如果我们用注解，就变得简单多了：

```php
class Soldier {  
    ...

    /**
     * @inject $weapon Weapon
     */
    public function setWeapon($weapon) {
        $this->weapon = $weapon;
    }
}

// 只需要2行代码就可以了
$soldier = new Soldier();
$soldier->fight();
```

####2、Drupal8执行时，内存使用大大减少。在Drupal7中，通过hook定义元数据，这样就导致每个请求都需要把所有的模块加载到内存中，而Drupal8仅解析注解，并加载进内存，这样就可以占用很小的内存。

###三、语法
注解的数据结构是类似json key-value。
让我们来看text模块的TextDefaultFormatter.php文件中的定义。

```php
/**
 * Plugin implementation of the 'text_default' formatter.
 *
 * @FieldFormatter(
 *   id = "text_default",
 *   label = @Translation("Default"),
 *   field_types = {
 *     "text",
 *     "text_long",
 *     "text_with_summary"
 *   },
 *   quickedit = {
 *     "editor" = "plain_text"
 *   }
 * )
 */
class TextDefaultFormatter extends FormatterBase {  
...
```

以下是一些规范：

####1、必须以一个唯一的id开始，类似Drupal7的machine name
####2、顶级的Keys可以使用双引号
####3、下级的Keys必须使用双引号
####4、必须使用双引号，单引号会造成语法错误
####5、value允许的数据类型
		> 字符串(string)：必须使用双引号,如"foo"，使用一对双引号包含双引号字符，如"The ""On"" value "
		> 数字(number)：不能使用引号，如果使用引号，会被解析成字符串
		> 布尔型(bool)：不能使用引号，如果使用引号，会被解析成字符串
		> list：必须使用花括号，末尾不能使用逗号。如
			
```php
field_types = {
	"text",
	"text_long",
	"text_with_summary"
}
```
		> map: 必须使用花括号，key-value用＝号分割，默认不能使用逗号。如：	
			
```php
quickedit = {
	"editor" = "plain_text"
}
```
		> 允许定义常量
		
###四、自定义注解类
Drupal8中的一些基础plugin如entity,views等会被自动加载。当然，你可以在自定义的类中给plugin提供文档和参数。

我们还是以text模块举例说明，在text/src/Plugin/field/formatter/TextPlainFormatter.php中定义了plaintext，并且自定义了一个叫FieldFormatter的注解类。

注意：所有的注解类都是继承自 \Drupal\Component\Annotation\Plugin。

```php
<?php
/**
 * Plugin implementation of the 'text_plain' formatter.
 *
 * @FieldFormatter(
 *   id = "text_plain",
 *   label = @Translation("Plain text"),
 *   field_types = {
 *     "text",
 *     "text_long",
 *     "text_with_summary"
 *   },
 *   edit = {
 *     "editor" = "direct"
 *   }
 * )
 */
class TextPlainFormatter {
?>
```

###五、自定义插件类型中使用注解

如果你想在自定义plugin类型中使用注解，可以用AnnotatedClassDiscovery类。

AnnotatedClassDiscovery构造函数的第一个参数是这个plugin类型所存放的目录结构。

如下面的代码，plugin在目录$module/src/Plugin/field/formatter中。

```php
<?php
use Drupal\Core\Plugin\Discovery\AnnotatedClassDiscovery;

class FormatterPluginManager extends PluginManagerBase {

  /**
   * Constructs a FormatterPluginManager object.
   *
   * @param array $namespaces
   *   An array of paths keyed by their corresponding namespaces.
   */
  public function __construct(array $namespaces) {
    // This is the essential line you have to use in your manager.
    $this->discovery = new AnnotatedClassDiscovery('Plugin/field/formatter', $namespaces);
    // Every other line is a good practice.
    $this->discovery = new ProcessDecorator($this->discovery, [$this, 'processDefinition']);
    $this->discovery = new AlterDecorator($this->discovery, 'field_formatter_info');
    $this->discovery = new CacheDecorator($this->discovery, 'field_formatter_types', 'field');
  }

}
?>
```

注入的命名空间来自于依赖注入容器，如FieldBundle：

```php
<?php
/**
 * @file
 * Contains Drupal\field\FieldBundle.
 */

namespace Drupal\field;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\Bundle\Bundle;

/**
 * Field dependency injection container.
 */
class FieldBundle extends Bundle {

  /**
   * Overrides Symfony\Component\HttpKernel\Bundle\Bundle::build().
   */
  public function build(ContainerBuilder $container) {
    // Register the plugin managers for our plugin types with the dependency injection container.
    $container->register('plugin.manager.field.widget', 'Drupal\field\Plugin\Type\Widget\WidgetPluginManager')
      ->addArgument('%container.namespaces%');
    $container->register('plugin.manager.field.formatter', 'Drupal\field\Plugin\Type\Formatter\FormatterPluginManager')
      ->addArgument('%container.namespaces%');
  }

}
?>
```

要调用自定义的plugin管理器，需要在构造函数调用的时候注入命名空间。

```php
<?php
 $type = new CustomPluginManager(\Drupal::getContainer()->getParameter('container.namespaces')); 
?>
```

###六、Entity注解

Entity类型不管是配置还是内容实体都必须用注解定义。

如：core/modules/user/src/Entity/User.php:

```php
...
/**
 * Defines the user entity class.
 *
 * The base table name here is plural, despite Drupal table naming standards,
 * because "user" is a reserved word in many databases.
 *
 * @ContentEntityType(
 *   id = "user",
 *   label = @Translation("User"),
 *   handlers = {
 *     "storage" = "Drupal\user\UserStorage",
 *     "storage_schema" = "Drupal\user\UserStorageSchema",
 *     "access" = "Drupal\user\UserAccessControlHandler",
 *     "list_builder" = "Drupal\user\UserListBuilder",
 *     "view_builder" = "Drupal\Core\Entity\EntityViewBuilder",
 *     "views_data" = "Drupal\user\UserViewsData",
 *     "route_provider" = {
 *       "html" = "Drupal\user\Entity\UserRouteProvider",
 *     },
 *     "form" = {
 *       "default" = "Drupal\user\ProfileForm",
 *       "cancel" = "Drupal\user\Form\UserCancelForm",
 *       "register" = "Drupal\user\RegisterForm"
 *     },
 *     "translation" = "Drupal\user\ProfileTranslationHandler"
 *   },
 *   admin_permission = "administer user",
 *   base_table = "users",
 *   data_table = "users_field_data",
 *   label_callback = "user_format_name",
 *   translatable = TRUE,
 *   entity_keys = {
 *     "id" = "uid",
 *     "langcode" = "langcode",
 *     "uuid" = "uuid"
 *   },
 *   links = {
 *     "canonical" = "/user/{user}",
 *     "edit-form" = "/user/{user}/edit",
 *     "cancel-form" = "/user/{user}/cancel",
 *     "collection" = "/admin/people",
 *   },
 *   field_ui_base_route = "entity.user.admin_form",
 * )
 */
class User extends ContentEntityBase implements UserInterface {
...
```

@ConfigEntityType继承自Drupal\Core\Config\Entity\ConfigEntityBase类。


参考文章：
1、https://www.drupal.org/node/1882526
2、https://www.drupal.org/node/2207559
3、https://www.drupal.org/node/1638040
4、http://lakshminp-lakshminp.rhcloud.com/annotations-in-drupal-8/
5、https://docs.phalconphp.com/zh/latest/reference/annotations.html