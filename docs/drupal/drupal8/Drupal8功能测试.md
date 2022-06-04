title: Drupal8功能测试
date: 2016-06-03 10:52:56
categories: Drupal8
tags: [测试]
---

Drupal8的功能测试必须要有一个正在运行的站点，并且安装了simpleTest核心模块。
本文的测试用例实现的功能为：
向user实体增加一个字段，然后检查用户里是否存在这个字段。

##一、PHPUnit和SimpleTest功能测试(BrowserTestBase / WebTestBase)
以后的功能测试都是使用phpUnit和BrowserTestBase，Drupal8中很多模块还在用WebTestBase.

BrowserTestBase的主要限制是不能使用JavaScript和AJAX。
如果需要测试它们，建议先用WebTestBase。

###1、定义字段

```bash
vim config/install/field.field.user.user.test_status.yml
```

编辑文件内容为：
```php
langcode: en
status: true
dependencies:
  config:
    - field.storage.user.test_status
id: user.user.test_status
field_name: test_status
entity_type: user
bundle: user
label: test_status
description: ''
required: false
translatable: false
default_value:
  -
    value: 0
default_value_callback: ''
settings:
  on_label: 'On'
  off_label: 'Off'
field_type: boolean
```

```bash
vim config/install/field.storage.user.test_status.yml
```
编辑内容为：
```php
langcode: en
status: true
dependencies:
  module:
    - user
id: user.test_status
field_name: test_status
entity_type: user
type: boolean
settings: {  }
module: test_example
locked: false
cardinality: 1
translatable: true
indexes: {  }
persist_with_no_fields: false
```

###2、创建功能测试
必须在tests/src/Functional/内创建一个以 Test.php 结尾的文件。
创建一个继承 BrowserTestBase 的类创建测试。
我们也使用 Drupal\Tests\[module_name]\Functional 作为所有测试类的namespace。

编辑src/Functional/TestExampleUserTest.php

```php
<?php

/**
 * @file
 *
 * Contains \Drupal\Tests\test_example\Functional\TestExampleUserTest.
 */

namespace Drupal\Tests\test_example\Functional;

use Drupal\simpletest\BrowserTestBase;

/**
 * Check if our user field works.
 *
 * @group test_example
 * @runTestsInSeparateProcesses
 * @preserveGlobalState disabled
 */
class TestExampleUserTest extends BrowserTestBase {

  /**
   * @var \Drupal\user\Entity\User.
   */
  protected $user;

  /**
   * Enabled modules
   */
  public static $modules = ['test_example'];

  /**
   * {@inheritdoc}
   */
  function setUp() {
    parent::setUp();

    $this->user = $this->drupalCreateUser();
  }

  /**
   * Test that the user has a test_status field.
   */
  public function testUserHasTestStatusField() {
    $this->assertTrue(in_array('test_status', array_keys($this->user->getFieldDefinitions())));
  }

}
```

- 注解
所有使用PhpUnit的功能测试必须要有两个注解，@runTestsInSeparateProcesses 和 @preserveGlobalState disabled。
- 激活模块
因为功能测试运行在没有完全构建的Drupal系统内，所以我们需要激活需测试的模块。
- setUp
必须调用parent::setUp方法，在这个测试用例里，我们调用了drupalCreateUser方法，这样我们可以访问到用户实体。
- 测试方法
和单元测试一样，方法必须以test开头。
- 断言语句
和单元测试一样。在上面这个测试用例，我们检查用户实体里是否存在test_status这个字段。

###3、执行功能测试
首先设置一下phpstrom
* 打开 PhpStorm
* 在 Command Line 部分，编辑 Environment  变量。
* 设置SIMPLETEST_BASE_URL 和值（站点URL)。
![PhpStorm 功能测试](https://static.verycloud.cn/sites/default/files/pic/image/20160603/20160603111934_95639.jpg)

##二、Ajax功能测试-SimpleTest

phpUnit和simpleTest的主要区别是：

* 文件命名和名字空间

> 要创建SimpleTest功能测试，你必须在src/Tests/内创建一个以Test.php结尾的文件。
> 一般通过继承基类WebTestBase或KernelTestBase创建测试。
> 使用Drupal\[module_name]\Tests作为所有测试类的namespace。

phpUnit功能测试的文件位于tests/src/Functional内。
simpleTest位于srs/Tests内。

下面是一个simpleTest:
src/Tests/TestExampleUserTest.php

```php
<?php

/**
 * @file
 *
 * Contains \Drupal\test_example\Tests\TestExampleUserTest.
 */

namespace Drupal\test_example\Tests;

use Drupal\simpletest\WebTestBase;

/**
 * Check if our user field works.
 *
 * @group test_example
 */
class TestExampleUserTest extends WebTestBase {

  /**
   * @var \Drupal\user\Entity\User.
   */
  protected $user;

  /**
   * Enabled modules
   */
  public static $modules = ['test_example'];

  /**
   * {@inheritdoc}
   */
  function setUp() {
    parent::setUp();

    $this->user = $this->drupalCreateUser();
  }

  /**
   * Test that the user has a test_status field.
   */
  public function testUserHasTestStatusField() {
    $this->assertTrue(in_array('test_status', array_keys($this->user->getFieldDefinitions())));
  }

}
```

Drupal8的核心模块有很多测试用例。有想学习的可以多看看。