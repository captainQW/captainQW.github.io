title: Drush的安装配置和应用
date: 2013-12-11 16:50:14
categories: Drupal
tags: [drush]
---

###安装方法

```php
pear channel-update pear.php.net
pear channel-discover pear.drush.org
pear install drush/drush
```

用drush help即可查看相关命令。

###经常使用的几个命令

```php
drush cc all: 清除缓存
drush updb: 更新数据库.
drush up或者 drush pm-update更新核心代码或者模块.

drush up的备份代码默认在 ~/drush-backups 
cd  ~/drush-backups 即可看到原来的代码
```

###打印Drupal函数
```php
drush ev "print_r(module_implements('menu'))"
```

###执行php代码
```php
drush php-eval 'print_r(file("/xxxx.log"));' 
```