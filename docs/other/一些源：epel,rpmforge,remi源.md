title: 一些源：epel,RPMForge,Remi源
date: 2015-02-03 09:31:50
categories: linux
tags: [yum]
---


为了保证官方源的顺序要高于第三方源的顺序:
yum install yum-priorities

加载epel源
CentOS and Red Hat Enterprise Linux 6.x
```bash
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm 
```

CentOS and Red Hat Enterprise Linux 7.x
```bash
https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

加载RPMForge源
rpm -Uvh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm

加载Remi源
CentOS and Red Hat Enterprise Linux 6.x
```bash
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```

CentOS and Red Hat Enterprise Linux 7.x
```bash
rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

由于CentOS官方说RPMFusion源里面的软件稳定性不如rpmforge，可以不加RPMFusion源.