title: 使用gdb调试php-fpm
date: 2015-11-27 16:17:10
categories: linux
tags: [gdb]
---

有很多php开发人员经常问我，如果执行php代码的时候出现严重错误的时候，如何快速定位问题。我一般推荐他们用gdb去调试。

下面用php-fpm的一个段错误来举例说明，如：

```bash
WARNING: [pool www] child 15109 exited on signal 11 (SIGSEGV) after 654.485221 seconds from start
```

###1、安装gdb

```bash
yum install gdb php-dbg
```

有的时候可能需要安装debuginfo相关的库，如:

```bash
debuginfo-install php-fpm
```

###2、设置coredump

```bash
echo '/tmp/core-%e.%p' > /proc/sys/kernel/core_pattern
echo 0 > /proc/sys/kernel/core_uses_pid
ulimit -c unlimited
```

当然,coredump的文件格式可以设置为其他的，这里是详细的设置项说明

```bash
%%  a single % character
%c  core file size soft resource limit of crashing process (since
    Linux 2.6.24)
%d  dump mode—same as value returned by prctl(2) PR_GET_DUMPABLE
    (since Linux 3.7)
%e  executable filename (without path prefix)
%E  pathname of executable, with slashes ('/') replaced by
    exclamation marks ('!') (since Linux 3.0).
%g  (numeric) real GID of dumped process
%h  hostname (same as nodename returned by uname(2))
%p  PID of dumped process, as seen in the PID namespace in which
    the process resides
%P  PID of dumped process, as seen in the initial PID namespace
    (since Linux 3.12)
%s  number of signal causing dump
%t  time of dump, expressed as seconds since the Epoch,
    1970-01-01 00:00:00 +0000 (UTC)
%u  (numeric) real UID of dumped process
```

###3、设置php-fpm

```bash
vim /etc/php-fpm.d/www.conf
```

找到关键字 rlimit_core，并修改为

```bash
rlimit_core = unlimited
```

然后执行

```bash
/usr/sbin/php-fpm --daemonize
```

这时就会在/tmp下看到有coredump文件生成了。

```bash
[root@localhost tmp]# ls
core-php-fpm.7279
```
###4、追踪
执行命令

```bash
gdb /usr/sbin/php-fpm /tmp/core-php-fpm.7279
```

执行bt命令就可以查看详细的错误日志了

```bash
(gdb) bt
#0  _zend_hash_init (ht=0x2b4d1a4f8100, nSize=174762, pDestructor=0, persistent=1 '\001') at /usr/src/debug/php-7.0.0RC8/Zend/zend_hash.c:172
#1  0x00002b4d0d0e87ac in zend_accel_init_shm (extension=<value optimized out>) at /usr/src/debug/php-7.0.0RC8/ext/opcache/ZendAccelerator.c:2387
#2  accel_startup (extension=<value optimized out>) at /usr/src/debug/php-7.0.0RC8/ext/opcache/ZendAccelerator.c:2640
#3  0x00000000005e8d21 in zend_extension_startup (extension=0x168c810) at /usr/src/debug/php-7.0.0RC8/Zend/zend_extensions.c:176
#4  0x00000000005d2293 in zend_llist_apply_with_del (l=0x9dd420, func=0x5e8d10 <zend_extension_startup>) at /usr/src/debug/php-7.0.0RC8/Zend/zend_llist.c:171
#5  0x00000000005e8d07 in zend_startup_extensions () at /usr/src/debug/php-7.0.0RC8/Zend/zend_extensions.c:197
#6  0x0000000000580985 in php_module_startup (sf=<value optimized out>, additional_modules=<value optimized out>, num_additional_modules=<value optimized out>) at /usr/src/debug/php-7.0.0RC8/main/main.c:2197
#7  0x000000000067aec5 in php_cgi_startup (sapi_module=<value optimized out>) at /usr/src/debug/php-7.0.0RC8/sapi/fpm/fpm/fpm_main.c:837
#8  0x000000000067bcfe in main (argc=2, argv=0x7fff1a037498) at /usr/src/debug/php-7.0.0RC8/sapi/fpm/fpm/fpm_main.c:1788
```
至此，整个追踪过程结束。
