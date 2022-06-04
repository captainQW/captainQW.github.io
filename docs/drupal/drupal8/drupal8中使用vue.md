title: Drupal8中使用vue
date: 2016-11-27 21:52:07
tags:
  - drupal
  - vue
categories:
  - drupal
---

## 一、什么是vue

[点击这里](https://cn.vuejs.org/)跳转vue官网

Vue.js（读音 /vjuː/, 类似于 view） 是一套构建用户界面的 渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。

Vue 的核心库只关注视图层，并且非常容易学习，非常容易与其它库或已有项目整合。

## 二、drupal融合vue

OK，新建一个模块，`cloud_system`，初始化一个`vue`在`drupal`中的根模版，如下：

```html
<div id="main-content">
  <router-view></router-view>
</div>
```

这样，所有的路由都输出这个模版，之后前端路由解析填充`router-view`

## 三、vue的模块化开发

详见**vue在drupal中的模块化开发**
