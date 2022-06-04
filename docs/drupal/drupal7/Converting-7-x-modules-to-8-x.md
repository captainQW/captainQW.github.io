title: Converting 7.x modules to 8.x
date: 2015-11-02 14:10:20
categories: Drupal8
tags: [drupal8]
---

本节主要介绍如何把drupal7的模块转化为drupal8。参考资料https://www.drupal.org/update/modules/7/8

先介绍一个模块drupalmoduleupgrader，他会分析drupal7的模块，并提供升级的指导说明，另外还可以直接升级为drupal8。安装方式也比较简单，如下：
```bash
drush dl drupalmoduleupgrader
cd modules/drupalmoduleupgrader
composer install               #安装依赖
drush en drupalmoduleupgrader  #启用此模块
```

###如何分析？
cp一个drupal7的模块到drupal8的modules目录下，执行以下命令分析:
```bash
drush dmu-analyze hello_world
```
会释出一个upgrade-info.html的文件，非常详细的标明哪些API需要修改。

![upgrade-info.html](https://static.verycloud.cn/sites/default/files/pic/image/20151102/20151102143425_76787.png)

###如何升级？
执行命令
```bash
drush dmu-upgrade hello_world
```
会自动释出一个drupal8的版本。

###转换原理？
使用了pharborist做代码转换，见https://github.com/grom358/pharborist

怎么样？灰常简单吧！