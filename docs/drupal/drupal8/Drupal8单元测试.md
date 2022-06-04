title: Drupal8单元测试
date: 2016-06-02 16:01:57
categories: Drupal8
tags: [测试]
---

单元测试在编程中是非常重要的，而我更倾向于去对那些有意义的错误做测试。
在Drupal8中，可以借助phpstorm来帮助进行单元测试。

##一、phpstorm配置

###1）、针对项目进行设置：
> 1、选中 PhpStorm > Preferences > Languages & Frameworks > PHP > PHPUnit > Local
> Window 操作系统是：File > Settings …
> 2、选择 PHPUnit Library > Use custom autoloader
> 3、导航到 /vendor/autoload.php
> 4、选择 Test Runner > Default configuration file
> 5、导航到 /core/phpunit.xml.dist
> 6、点击 Save

在右上方看到 For current project。接下来完成下面的步骤：

> 1、运行： Run > Edit Configurations
> 2、点击 + 图标，选择 PHPUnit
> 3、提供一个名称，例如 Drupal 8: All Tests
> 4、在配置文件内选择 Test Runner > Defined
> 5、保存

在右上角，选择你的运行配置，按下绿色箭头。也可以操作 Run > Run Drupal 8: All Tests 。

说明：
> 如果提示“Interpreter is not specified or invalid”，
> 要去 PhpStorm > Preferences > Languages & Frameworks > PHP 设置下 PHP 解释器。

当你运行测试时，你会在 PhpStorm 内看到一个新窗口，列出了测试结果。默认情况，通过的测试不会显示出来。你可以操作窗口左上角的 Hide passed 图标显示通过的测试。


###2)、测试一个指定区域

PHPUnit 将运行很多测试。要把测试区域限制为一个文件夹（我们的模块）可以象下面这样做：

> 1、运行： Run > Edit Configurations
> 2、点击 + 图标选择 PHPUnit
> 3、提供个名称，例如 Drupal 8: Test Example
> 4、选择 Test Runner > Test Scope > Directory
> 5、浏览 test_example 模块
> 6、点击 Save

##二、添加单元测试

要进行单元测试，必须在[module_name]/tests/srs/Unit/下创建一个以Test.php结尾的文件。一般都是继承UnitTestCase这个类来创建测试，这个测试类的namespace必须为Drupal\Test\[module_name]\Unit。

接下来，我们来书写代码。

```bash
vim tests/src/Unit/TestExampleConversionsTest.php
```

```php
<?php

/**
 * @file
 *
 * Contains \Drupal\Tests\test_example\Unit\TestExampleConversionsTest.
 */

namespace Drupal\Tests\test_example\Unit;

use Drupal\Tests\UnitTestCase;

/**
 * Demonstrates how to write tests.
 *
 * @group test_example
 */
class TestExampleConversionsTest extends UnitTestCase {
  /**
   * {@inheritdoc}
   */
  public function setUp() {
    parent::setUp();
  }

  /**
   * Convert Celsius to Fahrenheit
   *
   * @param $temp
   *
   * @return int
   */
  public function celsiusToFahrenheit($temp) {
    return ($temp * (9/5)) + 32;
  }

  /**
   * Convert centimeter to inches.
   *
   * @param $length
   *
   * @return int
   */
  public function centimeterToInch($length) {
    return $length / 2.54;
  }

  /**
   * A simple test that tests our celsiusToFahrenheit() function.
   */
  public function testOneConversion() {
    // Confirm that 0C = 31F.
    $this->assertEquals(31, $this->celsiusToFahrenheit(0));
  }

}
```

- 测试方法
每个测试方法必须以test开头。
- 断言
断言决定测试是否通过，比较常用的一些断言函数如assertTrue、assertFalse、assertEquals等。
文档见 https://phpunit.de/manual/current/zh_cn/appendixes.assertions.html
- setUp()
每次测试之前都会执行setUp方法，我们可以创建一个变成或实例。

##三、执行单元测试

输出如下：
```bash
Failed asserting that 32.0 matches expected 31.
Expected :31
Actual   :32
```

##四、较复杂的单元测试
如果要做比较复杂的单元测试，一个简单的办法是用setUp方法，另一个方案就是使用data provider.

###1)、data provider

PHPUnit中的data provider是为测试方法提供配置的方法，并返回一个测试用例数组。

本例中，我们创建一个data provider叫 providerCentimetersToInches。

```php
/**
* Provides data for the testCentimetersToInches method.
*
* @return array
*/
public function providerCentimetersToInches() {
  return [
    [2.545,1],
    [254,100],
    [0,0],
    [-2.54,-1],
  ];
}

/**
* Tests centimetersToInches method.
*
* @dataProvider providerCentimetersToInches
*/
public function testCentimetersToInches($length, $expectedValue) {
  $this->assertEquals($expectedValue, $this->centimeterToInch($length));
}
```

- Data Provider注解
在测试方法上使用 @dataProvider 注解来连接这个data provider。

- Data Provider参数
在测试方法中可以添加参数，在上面的测试方法中，我们在数组内传递了两个值，那么在上面的测试方法中有两个对应的参数。

执行run，结果如下
```bash
Failed asserting that 1.0019685039370079 matches expected 1.
Expected :1
Actual   :1.001968503937
```