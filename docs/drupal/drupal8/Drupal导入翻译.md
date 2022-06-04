title: Drupal导入翻译
date: 2013-12-11 17:19:06
categories: Drupal
tags: [drush]
---

有的时候po文件太大，导入的时候会出现503错误。这是因为php请求超时了。还好，有drush,用drush 导入文件非常简单。
```php
drush dl drush_language
```

Usage

```php
drush language-add <langcode>
> Add and import a new language definition.
drush language-enable <langcode>
> Enable an existing language.
drush language-disable <langcode>
> Disable an enabled language.
drush language-default <langcode>
> Assign an enabled language as default.
drush language-import <langcode> <file.po> [--replace] [--group=<text-group>]
> Import a .po file to a language.
drush language-export <langcode> <file.po> [--group=<text-group>]
> Export a language to a .po file.
```

比如导入一个翻译文件，命令如下：

```php
drush language-import zh-hans xxxx/drupal-7.22.zh-hans.po 
```