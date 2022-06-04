title: Drupal7表单提交时出现非法选项解决方案
date: 2014-01-20 20:44:22
categories: Drupal
tags: [drupal7表单]
---

有的使用ajax操作表单时，提交时会出现“在请选择xxx元素中的非法选项1。(An illegal choice has been detected ajax call!)”，解决办法很简单，如下

```php
form['my_dynamic_select'] = array(
  … 
  '#type' => 'select',
  '#validated' => TRUE, // 表单元素添加该行即可。
  …
)
```

为什么要加'#validated' => TRUE呢？
由于我们使用ajax取得的值，页面加载时并不存在，Drupal会认为这个值时非法的。所以我们要重置表单验证项永远为真，即可完美解决此问题。