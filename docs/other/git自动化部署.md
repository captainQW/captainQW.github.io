title: git自动化部属
date: 2016-12-21 07:47:36
categories: linux
tags: [git]
---

gitlab和github都支持通过webhook自动部署，本文使用gitlab+webhook的方式实现自动化部署项目。

1、填写回调URL
随便在gitlab上找一个项目，填写回调URL，如果有Push请求，gitlab会自动回调你填写的这个地址。
![回调地址](https://static.verycloud.cn/sites/default/files/pic/image/20161221/2016122175313_33517.png)

2、开发服务器配置
一般php是通过php-fpm来运行，而php-fpm一般由apache用户来运行。确保clone下来的仓库权限比如为apache.
可通过如下命令进行修改

```bash
# chown apache:apache you_project * -R
```

创建apache用户的.ssh文件
```bash
sudo -u apache ssh-keygen -t rsa
```

3、测试
在本地commit一个提交.

```bash
# git ci -am 'test webhook'
# git push origin dev
```

在开发服务器上查看日志
```bash
# tail -f /var/log/php-fpm/www-error.log
```

可以看到
```bash
[20-Dec-2016 23:00:59 Asia/Chongqing] Array
(
    [0] => From git.xxx.cn:your_project/project_name
    [1] =>  * branch            master     -> FETCH_HEAD
    [2] => Updating c02949b..f8c52c5
    [3] => Fast-forward
    [4] =>  webhook.php |   42 ++++++++++++++++++++++++++++++++++++++++++
    [5] =>  1 files changed, 42 insertions(+), 0 deletions(-)
    [6] =>  create mode 100644 webhook.php
)
```

搞定！

附、webhook.php源码

```php
<?php
$event = $_SERVER['HTTP_X_GITLAB_EVENT'];
$client_ip = $_SERVER['REMOTE_ADDR'];
$access_ip = array('192.168.112.151');
// access ip
if (!in_array($client_ip, $access_ip)) {
  echo "Invalid ip [{$client_ip}]" . PHP_EOL;
  exit(0);
}
if ($event == 'Push Hook') {
  // get json data
  $input = file_get_contents("php://input");
  $json = json_decode($input, true);
  if ($json['project']['name'] != 'portal') {
     exit(0);
  }
  $branch = $json['ref'];
  switch ($branch) {
    case 'refs/heads/dev':
      exec("cd /var/www/html/voss/portal/;/usr/bin/git pull origin dev 2>&1", $output, $result);
      error_log(print_r('pull dev', true));
      error_log(print_r($output, true));
      error_log(print_r($result, true));
      break;
      
    default:
      exec("cd /var/www/html/voss/portal/;/usr/bin/git pull origin master 2>&1", $output, $result);
      error_log(print_r('pull master', true));
      error_log(print_r($output, true));
      error_log(print_r($result, true));
      break;
  }
}
```
