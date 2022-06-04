title: PHP Trait多重继承的实现
date: 2016-05-21 14:13:20
categories: PHP
tags: [php trait]
---

官方地址：
http://php.net/manual/en/language.oop5.traits.php

自 PHP 5.4.0 起，PHP 实现了代码复用的一个方法，称为 traits。

Traits 是一种为类似 PHP 的单继承语言而准备的代码复用机制。Trait 为了减少单继承语言的限制，使开发人员能够自由地在不同层次结构内独立的类中复用方法集。Traits 和类组合的语义是定义了一种方式来减少复杂性，避免传统多继承和混入类（Mixin）相关的典型问题。

Trait 和一个类相似，但仅仅旨在用细粒度和一致的方式来组合功能。Trait 不能通过它自身来实例化。它为传统继承增加了水平特性的组合；也就是说，应用类的成员不需要继承。

在我理解说白了就是在继承类链中隔离了子类继承父类的某些特性（就是子类“要用父类的特性的时候”，如果trait有，就优先调用trait的方法、属性等）。

```php
<?php
trait MyTrait
{
    protected $var = "MyTrait_var";
    protected $var1 = "MyTrait_var";
 
    function __construct()
    {
        echo $this->var.PHP_EOL;
    }
 
    function a()
    {
        echo "a".PHP_EOL;
    }
}
 
interface MyInterface
{
    function __construct();
    function b();
}
 
abstract class MyAbstract
{
    protected $var2 = "MyAbstract_var";
 
    use MyTrait;
 
    function b()
    {
        echo "b".PHP_EOL;
    }
}
 
class MyClass extends MyAbstract implements MyInterface
{
    protected $var3 = "MyClass_var";
 
    //也可以在这里引用，不区分继承关系
    //use MyTrait;
    function c()
    {
        echo "c".PHP_EOL;
    }
}
 
$class = new MyClass();
$class->a();
$class->b();
$class->c();
```

输出结果
MyTrait_var
a
b
c

总结：

从本质上说，trait和include文件的概念差不多
trait可以更加方便的实现代码复用，因为我们用继承关系实现的无法在父类中访问子类的private属性与方法，而trait就和把代码直接写在对象里效果一样。
使用trait时候应该坚决避免命名冲突，尤其是同时使用多个trait时。
如果产生了命名冲突，如果两者的可见性、初始值、static与否完全相同，则trait中的会覆盖掉对象中的，并抛出E_STRICT错误，否则会抛出E_COMPILE_ERROR错误，终止编译。