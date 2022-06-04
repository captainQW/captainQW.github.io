title: Drupal 获取客户端真实IP
date: 2013-12-11 18:01:12
categories: Drupal
tags: [cdn_real_ip]
---


如果网站走CDN。必须在settings.php设置
```php
$conf['reverse_proxy'] = TRUE;
```
这样才可以获取到客户端的真实IP。