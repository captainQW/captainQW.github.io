title: Drupal8面向对象(OOP)
date: 2015-11-16 15:16:30
categories: Drupal8
tags: [oop]
---

这是学习Drupal8的系列的开篇文章，在这里我们将介绍和探讨Drupal OOP。
OOP的最大优势就是解耦，Drupal8中使用大量的OOP(interface, class, etc...)，这里仅对Drupal中常见的一些概念做一下介绍。

###一、数据类型
什么是对象？我们先来看字符串，一个字符串就是一个数据类型，可以改变或获取字符串的一些信息，如要获取长度，我们一般是用strlen()函数。
但是对于其他的数据类型，如整形(int)等，这种方式就不适用。

而OOP的优势在于，它允许你自定义数据类型。这些数据类型也称之为"类"，在类中包含一些内部结构，如:

```php
class Person
{
  // 下面是人的成员属性
  protected $name; // 人的名子
  protected $sex; // 人的性别
  protected $age; // 人的年龄

  // 构造函数
  public function __construct($name, $sex, $age) {
  	$this->name = $name;
  	$this->sex = $sex;
  	$this->age = $age;
  }

  public function getPerson() {
  	return $this->name . '，性别：' . $this->sex . '，年龄：' . $this->age;
  }
}
```
这段代码定义了一个名叫Person的类，包含三个属性$name, $sex, $age，也包含2个方法。实例化如下：

```php
$name = '王鹏';
$sex = '男';
$age = 30;
$myperson = new Person($name, $sex, $age);
```
$myperson现在就是一个对象实例。对于Drupal来说，类和对象的关系就类似内容类型和内容之间的关系。

###二、类的结构(class)
类的概念：类是具有相同属性和服务的一组对象的集合。它为属于该类的所有对象提供了统一的抽象描述，其内部包括属性和服务两个主要部分。在面向对象的编程语言中，类是一个独立的程序单位，它应该有一个类名并包括属性说明和服务说明两个主要部分。

这段文字是摘抄自google，比较晦涩难懂。
简单说，方法是可以在对象上执行的操作。比如在Person类，我们有一个方法叫getPerson()，我们可以通过引用对象调用TA。

```php
$person = $myperson->getPerson();
```
当然，我们不能用$myperson->name或$myperson->sex来引用这些属性。我们先来看一些关键词public、protected，他们定义了方法或者属性的可见性。public方法或者属性可以外部访问调用，如getPerson()。但是protected仅限于在类或子类的内部调用。而private就只能在类自己内部调用，而不能在它的子类中。

Drupal中一般声明属性protected,方法声明为public或protected，private一般不赞成使用。

###三、接口(interface)
接口是一种特殊的抽象类，抽象类又是一种特殊的类，所以接口也是一种特殊的类，为 什么说接口是一种特殊的抽象类呢？如果一个抽象类里面的所有的方法都是抽象方法，那么我们就换一种声明方法使用“接口”；也就是说接口里面所有的方法必须 都是声明为抽象方法，另外接口里面不能声明变量(但可声明常量constant)，而且接口里面所有的成员都是public权限的。所以子类在实现的时候 也一定要使用public权限实限。

在类中，Person定义的对象大抵是这样：

```php
class Person
{
	public function getPerson() {...}
}
```

那如何定义接口呢？
```php
interface PersonInterface 
{
	public function getPerson();
}

class Person implements PersonInterface
{
	public __construct($name, $sex, $age);
	public function getPerson() {
		/*..........*/
	}
}
```

这就像用户界面定义了人和程序如何交互，而接口定义了方法和对象的交互过程。当然你不能创建一个接口对象，而必须一个接口类。

说简单点。就是---接口定义了方法，类实现了方法。

###四、构造方法(constructor)
我们再来看Person类中的方法__construct()，它是用两个下划线作为前缀。这是PHP5中引入的。当创建一个对象的时候，会自动调用构造函数，也就是用new这个关键字实例化对象的时候自动调用构造函数。
在一个类中只能声明一个构造方法，而是只有在每次创建对象的时候都会去调用一次构造方法，不能主动的调用这个方法，所以通常用它执行一些有用的初始化任务。比如对成属性在创建对象的时候赋初值。如我们上面的代码：

```php
$myperson = new Person($name, $sex, $age);
```
它是如何执行的呢？
1、在内存中创建一个Person对象
2、用new关键字调用__construct方法
3、把Person对象复制给$this属性
4、创建一个名为$myperson的变量，他并不是一个对象，他用来处理对象
5、把$myperson指向内存中的Person对象

注意：不要在接口中定义构造方法，这样会导致代码逻辑混乱！

###五、继承(inherit)
继承是OOP的一个重要的特性，继承是子类自动共享父类数据结构和方法的机制，这是类之间的一种关系。如：

```php
class Student extends Person {
  public function __construct($name, $sex, $age) {
    $this->name = $name;
    $this->sex = $sex;
    $this->age = $age;
  }
}

$mystudent = new Student($name, $sex, $age);
```
这里声明了Student类是Person的子类，我们这里没有定义getPerson方法，这意味着这个方法继承自Person类，如$mystudent->getPerson()和$mypeson->getPerson()执行的结果是一样的。我们这里也没有定义$name, $sex, $age这些属性，他们也是继承自Person父类。

在C++语言中，一个派生类可以从一个基类派生，也可以从多个基类派生。从一个基类派生的继承称为单继承；从多个基类派生的继承称为多继承。

但是在PHP和Java语言里面没有多继承，只有单继承，也就是说，一个类只能直接从一个类中继承数据， 这就是我们所说的单继承。

















































































































































































