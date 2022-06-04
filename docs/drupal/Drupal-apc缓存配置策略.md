title: Drupal apc缓存配置策略
date: 2013-12-11 17:38:28
categories: Drupal
tags: [apc]
---

今天发现在如果开起来 apc缓存。并开启缓存页面（不从数据库拿）,匿名用户会返回500。。。。
配置代码如下：

```php
$conf['page_cache_without_database'] = TRUE;
```

通过追踪代码，发现drupal针对匿名用户并没有设置缓存。那用apc缓存了当然会有问题，修改代码如下：

```php
if (1) {
  $conf['cache_backends'] = array('sites/all/modules/apc/drupal_apc_cache.inc');
  if (1) {
    // Use APC by default (cache everything.)
    $conf['cache_default_class'] = 'DrupalAPCCache';
    if (1) {
      // Page cache without database and module hooks (approx twice as fast.)
      $conf['page_cache_invoke_hooks'] = FALSE;
      $conf['page_cache_without_database'] = TRUE;
    }
  }
  else {
    // Use APC for "cache" and "bootstrap" only.
    $conf['cache_class_cache'] = 'DrupalAPCCache';
    $conf['cache_class_cache_bootstrap'] = 'DrupalAPCCache';
  }
} 
```
问题解决. 