title: Drupal7生产环境的一些配置
date: 2015-03-21 15:09:59
categories: Drupal
tags: [drupal7配置]
---

由于经常需要配置，这里主要是做一个梳理和整理，免得以后忘记

1、缓存设置，见文章
	
	```php
	http://176code.com/tags/apc/
	```

2、如果站点启用了https，必须在settings.php里做如下设置，不然会导致session问题
	
	```php 
	$conf['https'] = TRUE;
	```

3、如果需要去除所有的报错，则在settings.php里做如下设置 
	
	```php
	$conf['error_level'] = 0;
	```

4、php.ini设置 date.timezone = Asia/Chongqing

5、解决跨域问题,在.htaccess里添加如下代码
	```php
	Header set Access-Control-Allow-Origin "*"
	```