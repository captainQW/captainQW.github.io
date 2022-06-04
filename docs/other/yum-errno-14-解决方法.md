title: yum Errno 14 解决方法
date: 2015-03-26 10:51:49
categories: linux
tags: [yum]
---

今天在安装supervisor的时候，报如下错误

```php
[root@admin yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
Determining fastest mirrors  
http://balabala/repodata/repomd.xml: [Errno 14] problem making ssl connection
Trying other mirror.
http://balabala/repodata/repomd.xml: [Errno 14] problem making ssl connection
Trying other mirror.
```

排查应该是ssl证书问题有关，网上查了资料发现需要安装 ca-certificates 才行。

```php
yum install ca-certificates -y
```