title: Drupal8数据库(database)
date: 2016-07-20 22:20:15
categories: Drupal8
tags: [database]
---

Drupal8的数据库操作函数跟Drupal7没什么变化，但是，在Drupal9(还没开始开发，但是代码注释中已经有)中会把一些函数移除，比如db_add, db_insert, db_query等，见 [Drupal8 database](https://api.drupal.org/api/drupal/core%21includes%21database.inc/8 "D8 database")。

##Select

###Get single value:

```php
$query = \Drupal::database()->select('node_field_data', 'nfd');
$query->addField('nfd', 'nid');
$query->condition('nfd.title', 'Potato');
$query->range(0, 1);
$nid = $query->execute()->fetchField();
```

###Get single row:

```php
$query = \Drupal::database()->select('node_field_data', 'nfd');
$query->fields('nfd', ['nid', 'title']);
$query->condition('nfd.type', 'vegetable');
$query->range(0, 1);
$vegetable = $query->execute()->fetchAssoc();
```

###Using db like:

```php
$query = \Drupal::database()->select('node_field_data', 'nfd');
$query->fields('nfd', ['nid', 'title']);
$query->condition('nfd.type', 'vegetable');
$query->condition('nfd.title', $query->escapeLike('ca') . '%', 'LIKE');
$vegetable = $query->execute()->fetchAllKeyed();
```

###Get several rows with JOIN:

```php
$query = \Drupal::database()->select('node_field_data', 'nfd');
$query->fields('nfd', ['nid', 'title']);
$query->addField('ufd', 'name');
$query->join('users_field_data', 'ufd', 'ufd.uid = nfd.uid');
$query->condition('nfd.type', 'vegetable');
$vegetable = $query->execute()->fetchAllAssoc('nid');
```

##Insert

```php
$query = \Drupal::database()->insert('flood');
$query->fields([
  'event',
  'identifier'
]);
$query->values([
  'My event',
  'My indentifier'
]);
$query->execute();
```

注：values可以调用多次，如
```php
$query->values([
  'My event',
  'My indentifier'
])
->values([
  'My event1',
  'My indentifier1'
]);
```

##Update

```php
$query = \Drupal::database()->update('flood');
$query->fields([
  'identifier' => 'My new identifier'
]);
$query->condition('event', 'My event');
$query->execute();
```

##Upsert

```php
$query = \Drupal::database()->upsert('flood');
$query->fields([
  'fid',
  'identifier',
]);
$query->values([
  1,
  'My indentifier for upsert'
]);
$query->key('fid');
$query->execute();
```

注：upsert，顾名思义就是update+insert的作用。
根据条件判断有无记录，有的话就更新记录，没有的话就插入一条记录。

key()用来定义一个已经存在的row, 必须是unique类型，不然永远执行Insert操作.

###Delete

```php
$query = \Drupal::database()->delete('flood');
$query->condition('event', 'My event');
$query->execute();
```
