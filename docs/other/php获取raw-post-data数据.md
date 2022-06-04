title: PHP获取Raw POST Data数据
date: 2013-12-11 16:05:05
categories: PHP
tags: [Raw POST]
---

通常在做网站开发的时候，接收POST数组通过$_POST即可获取，但是如果把一段 XML 文本直接作为POST DATA提交到服务器时，$_POST是一个空数组，如：
```php
curl -X POST http://www.example.com -H "Accept:application/json" -d '{"ip":"8.8.4.4"}'
```

如何解决这个问题呢，PHP提供了解析Raw POST data的方法，见http://www.php.net/manual/en/reserved.variables.httprawpostdata.php，用法如下：

```php
$postdata = file_get_contents("php://input");
```

解释一下，php://input 是一个资源标识，用 file_get_contents 从这个资源中获取的内容，就是原始的 Raw POST Data。
