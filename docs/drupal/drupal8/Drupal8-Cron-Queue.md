title: Drupal8 Cron Queue
date: 2015-12-23 17:26:43
categories: Drupal8
tags: [queue]
---

Drupal中的Queue(队列)用于处理比较耗时的任务，一般是通过系统的Cron来触发。

本文github：https://github.com/RamboLau/drupal8-demos/tree/master/queue_example

###一、Drupal7 Queue

Drupal7一般是通过hook_cron_queue_info()这个钩子来实现，如：

```php
<?php
/**
 * Implements hook_cron().
 */
function queue_example_cron() {
  $queue = DrupalQueue::get('example_queue');

  foreach(example_data() as $item) {
    $queue->createItem($item);
  }
}

/**
 * Implements hook_cron_queue_info().
 */
function queue_example_cron_queue_info() {
  $queues = array();

  $queues['example_queue'] = array(
    'worker callback' => 'example_queue_worker', // queue调用的函数
    'time' => 60, //每次运行时，可供消耗的最长时间，单位是秒
  );

  return $queues;
}
?>
```

###二、Drupal8 Queue

Drupal8的Queue(队列)是通过QueueWorker这个插件来实现的。

####1、src/Plugin/QueueWorker/ExampleQueue.php

```php
<?php

/**
 * @file
 * Contains Drupal\queue_example\Plugin\QueueWorker\ExampleQueue
 */

namespace Drupal\queue_example\Plugin\QueueWorker;

use Drupal\Core\Queue\QueueWorkerBase;

/**
 * @QueueWorker(
 *   id = "example_queue",
 *   title = @Translation("example queue"),
 *   cron = {"time" = 10}
 * )
 */
class ExampleQueue extends QueueWorkerBase {
  /**
   * {@inheritdoc}
   */
  public function processItem($data) {
    // Process data here.
    throw new \Exception('Process data ' . json_encode($data));
  }
}

?>
```

####2、queue_example.module

要执行queue，必选实现hook_cron这个钩子。

```php
/**
 * Implements hook_cron().
 *
 * @see \Drupal\queue_example\Plugin\QueueWorker\ExampleQueue
 */
function queue_example_cron() {
  $queue = \Drupal::queue('example_queue', TRUE);
  $item = [ 'entity_type' => 'node', 'id' => 1 ];
  $queue->createItem($item);
}
```

####3、结果

1）、访问http://127.0.0.1/drupal8/admin/config/system/cron执行cron
2）、打开phpmyadmin，查看queue表

![queue](https://static.verycloud.cn/sites/default/files/pic/image/20151225/20151225142932_68268.png)

3）、访问http://127.0.0.1/drupal8/admin/reports/dblog，就可以看到我们在processItem方法中抛出的异常了。

![dblog](https://static.verycloud.cn/sites/default/files/pic/image/20151225/20151225143322_46779.png)

![dblog1](https://static.verycloud.cn/sites/default/files/pic/image/20151225/20151225143400_63877.png)

参考文章：
1、http://www.vdmi.nl/blog/creating-drupal-cron-queue-worker-drupal-8
2、http://www.sitepoint.com/drupal-8-queue-api-powerful-manual-and-cron-queueing/