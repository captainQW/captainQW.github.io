title: rsync主备代码同步
date: 2015-07-04 10:00:54
categories: linux
tags: [rsync]
---

同步服务器192.168.1.100
接收服务器192.168.1.200
即把192.168.1.100的代码备份到192.168.1.200。

# 192.168.1.100上的配置如下：
```bash
yum install rsync inotify-tools

crontab -e
*/59 * * * * /bin/sh /usr/local/scripts/rsync.sh >> /var/log/rsync.log
即每小时的第59分钟同步代码
```

# /usr/local/scripts/rsync.sh脚本内容如下

```bash
#!/bin/sh 
SRC=/var/www/html/ #记得在最后面加/不然RYNC会自动增加一层目录 
DES=rsync_data #192.168.1.200认证的模块名，后面会说明
IP=192.168.1.200
USER=backup 
INWT=/usr/bin/inotifywait 
RSYNC=/usr/bin/rsync 
$RSYNC -auzviL --delete --progress --password-file=/root/rsync.pwd $SRC $USER@$IP::$DES
```

> rsync 选项说明
>  -a 称为归档模式，执行以下操作
> 递归模式
> 保留符号链接
> 保留权限
> 保留时间标记
> 保留用户名及组名
> -z 开启压缩
> -v 详情输出
> -u 选项可以排除被修改过的目的文件
> -L 把软连接文件夹同步过来
> 查看每个文件的传输进程,用 –-progress 参数
> 删除在目的文件夹中创建的文件,用 –-delete 参数

# 192.168.1.200上的配置如下：

```bash
yum install rsync

vim /etc/rsyncd.conf
uid=root 
gid=root 
max connections=36000 
use chroot=no 
log file=/var/log/rsyncd.log 
pid file=/var/run/rsyncd.pid 
lock file=/var/run/rsyncd.lock 
[rsync_data] 
path=/data/192.168.1.100/ #192.168.1.100是一个必须存在的目录
comment = 192.168.1.100
ignore errors = yes
read only = no 
hosts allow = 192.168.1.100
```

rsync启动脚本
```bash
#! /bin/sh
#
# chkconfig:   2345 50 50
# description: The rsync daemon

# source function library
 . /etc/rc.d/init.d/functions

PROG='/usr/bin/rsync'
BASE=${0##*/}

# Adapt the --config parameter to point to your rsync daemon configuration
# The config file must contain following line:
#  pid file = /var/run/<filename>.pid
# Where <filename> is the filename of the init script (= this file)
OPTIONS="--daemon --config=/etc/rsyncd.conf"

case "$1" in
  start)
    echo -n $"Starting $BASE: "
    daemon --check $BASE $PROG $OPTIONS
    RETVAL=$?
    [ $RETVAL -eq 0 ] && touch /var/lock/subsys/$BASE
    echo
    ;;
  stop)
    echo -n $"Shutting down $BASE: "
    killproc $BASE
    RETVAL=$?
    [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$BASE
    echo
    ;;
  restart|force-reload)
    $0 stop
    sleep 1
    $0 start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|force-reload}" >&2
    exit 1
    ;;
esac

exit 0
```

至此，同步就算完成。可以tailf /var/log/rsync.log查看同步日志。