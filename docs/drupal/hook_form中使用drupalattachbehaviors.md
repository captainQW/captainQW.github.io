title: hook_form中使用Drupal.attachBehaviors
date: 2014-04-04 09:46:23
categories: Drupal
tags: [hook_form]
---

如果form中有ajax事件，可能需要重新绑定事件，可以这样解决，代码如下

```php
/**
 * Implements hook_form_alter().
 */
function xxx_form_alter(&$commands, &$form, $form_state) {
  //用于Drupal.attachBehaviors(document)
  $commands[] =  ajax_command_invoke(NULL, 'xxx_form_alter_attach_behaviors');
}

(function($) {

Drupal.behaviors.xxx_form = {
  attach: function (context, settings) {
    //js here

  }
}

$.fn.xxx_form_alter_attach_behaviors = function() {
  Drupal.attachBehaviors(document);
}

})(jQuery);
```
