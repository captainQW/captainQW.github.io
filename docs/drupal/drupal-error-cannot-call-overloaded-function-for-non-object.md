title: Drupal error - Cannot call overloaded function for non-object
date: 2013-12-31 13:35:34
categories: Drupal
tags: [drupal]
---

今天在测试API的时候，apache总是会莫名其妙的报
```php
PHP Fatal error: Cannot call overloaded function for non-object in /var/www/html/includes/database/query.inc on line 331
```
找了很久的资料，发现可以配置apache的CoreDumpDirectory命令，其实就是把apache的内核目录放到其他目录下。如/tmp/apache_core，并给apache_core文件给予chown apache:apache /tmp/apache_core的权限。这样可以解决问题。

但是需要注意：
该命令会导致tmp文件越来越大。

使用以下命里进行调试
gdb /usr/bin/httpd /tmp/apache_core/core