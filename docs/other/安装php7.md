title: 安装php7
date: 2015-03-20 20:23:11
categories: php
tags: php7
---

推荐用阿里云主机，访问国外飞速。

```bash
git clone https://git.php.net/repository/php-src.git
```

```bash
yum install -y gcc gcc-c++  make zlib zlib-devel pcre pcre-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers curl-devel libXpm-devel libc-client-devel re2c bison bison-devel libmcrypt libmcrypt-devel libcurl libcurl-devel 
```
```bash
./buildconf
```

```bash
./configure --prefix=/usr/local/php7 --with-config-file-path=/usr/local/php7/etc --with-mcrypt=/usr/include --with-mysql-sock=/var/lib/mysql/mysql.sock --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-gd --with-iconv --with-zlib --enable-xml --enable-mysqlnd --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-curl --with-jpeg-dir --with-freetype-dir --enable-opcache --enable-debug
```

```bash
make && make install
```

> 软连接 ln -s /usr/local/php7/bin/php /usr/bin/php

> 在安装目录下有两个php的配置项，php.ini-development和php.ini-production,一个是开发环境用的，一个线上环境用的，改名php.ini并移到/usr/local/php/etc目录。

> php -v
	
	```php

	PHP 7.0.0-dev (cli) (built: Mar 20 2015 20:04:04) 
	Copyright (c) 1997-2015 The PHP Group
	Zend Engine v3.0.0-dev, Copyright (c) 1998-2015 Zend Technologies
	
	```