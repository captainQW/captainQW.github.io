title: Drush for Drupal8
date: 2015-11-02 13:54:20
categories: Drupal8
tags: [drush]
---

文档见http://docs.drush.org/en/master/install/，这里记录下笔者的安装流程。

##1、安装composer

作为最知名的php第三方库安装工具，drush也支持使用这种方式安装。
composer的安装比较简单。

```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
```

vim ~/.bashrc 并添加一条 export PATH="$HOME/.composer/vendor/bin:$PATH"。

##2、安装drush

```bash
# Create and/or navigate to a path for the single Composer Drush install.
mkdir --parents /opt/drush-8.x
cd /opt/drush-8.x
# Initialise a new Composer project that requires Drush.
composer init --require=drush/drush:8.* -n
# Configure the path Composer should use for the Drush vendor binaries.
composer config bin-dir /usr/local/bin
# Install Drush. 
composer install
```

安装完，如果有Unable to load autoload.php...的错误，编辑

```bash
vim /usr/local/drush/drush/drush/includes/preflight.inc
```

查找$global_vendor_path，修改为

```bash
$global_vendor_path = DRUSH_BASE_PATH . '/../vendor/autoload.php';
```

执行命令

```bash
drush version
```

如果显示

```bash
Drush Version:  8.1.2
```

则安装成功。

为什么要安装drush?因为drush是一款非常优秀的drupal管理工具，drupal离不开drush。
下文将讲解如何把drupal7的模块转化为drupal8的模块。