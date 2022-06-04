title: centos6.6安装mariadb10 Tokudb
date: 2015-11-20 20:17:18
categories: mysql
tags: [tokudb]
---

ps: 如果想在mariadb10上使用tokudb就只能自己编译了。。。。

###一、环境准备：
tokudb的编译需要cmake 2.8.9+ 和 gcc 4.7+

本文选用cmake 2.8.9 和 gcc4.8.1。
这2个软件都只能使用源码编译安装。

cmake的很简单，直接下载
wget http://www.cmake.org/files/v2.8/cmake-2.8.9.tar.gz
然后解压进行编译
```bash
./configure
make & make install
```

###二、安装gcc 4.8.1
####1、下载gcc 4.8.1源码包：
http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-4.8.1/gcc-4.8.1.tar.bz2
我是虚拟机里面装的Linux，我嫌wget太慢，所以自己在Windows上用迅雷下好，然后共享到Linux中。

####2、解压：
```bash
tar -jxvf gcc-4.8.1.tar.bz2
```

####3、下载编译所需的依赖包：
这个步骤有两种方式完成：
a) 如果Linux有网络连接，直接这样：
```bash
cd gcc-4.8.1
./contrib/download_prerequisites
cd ..
```

b) 如果Linux没有网络连接（我主机和虚拟机是Host-only，不能联网，所以另外想办法），则用Windows上网下载这几个包：
ftp://ftp.gnu.org/gnu/gmp/gmp-4.3.2.tar.bz2
http://www.mpfr.org/mpfr-2.4.2/mpfr-2.4.2.tar.bz2
http://www.multiprecision.org/mpc/download/mpc-0.8.1.tar.gz
有人问，一定要下载几个版本吗？下载最新的版本行不行？我没试过，也不知道，我是按照gcc-4.8.1/contrib/download_prerequisites脚本里面的版本下载的。既然里面已经说了这几个版本，那我就严格按照它的要求来做。
然后解压并移动到gcc-4.8.1下面：
```bash
tar -xjf gmp-4.3.2.tar.bz2
tar -xjf mpfr-2.4.2.tar.bz2
tar -xzf mpc-0.8.1.tar.gz
mv gmp-4.3.2 gcc-4.8.1/gmp
mv mpfr-2.4.2 gcc-4.8.1/mpfr
mv mpc-0.8.1 gcc-4.8.1/mpc
```
这样的做法好处是，不用单独编译gmp、mpfr和mpc三个包，放在gcc源码下面一起编译（事实上这也是gcc-4.8.1/contrib/download_prerequisites脚本的做法，个人感觉更简洁些）。

####4、新建目录用于存放编译结果：
```bash
mkdir gcc-build-4.8.1
```

####5、进入新目录，并执行configure命令，产生makefile：
```bash
cd gcc-build-4.8.1
../gcc-4.8.1/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
```
具体选项不多解释，大家可以自己查看，我只用到c和c++，所以只编译这两种语言的编译器。

####6、编译：
```bash
make -j4
```
我是i5四核，所以开4个线程同时编译，要是有8核就更爽了~我在虚拟机里面花了30分钟不到的时间，不算太慢了。
####7、安装：
```bash
sudo make install
```
####8、大功告成，检查版本：
```bash
g++ --version
g++ (GCC) 4.8.1
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

###三、编译安装
下载mariadb源码mariadb-10.0.14.tar.gz
 https://downloads.mariadb.org/mariadb/10.0.14/

首先需要修改tokudb的cmake编译文件，去掉fuse-linker-plugin相关内容
修改mariadb-10.0.14/storage/tokudb/ft-index/cmake_modules/TokuSetupCompiler.cmake
去掉所有的 -fuse-linker-plugin ：
```bash
#  set(CMAKE_C_FLAGS_RELWITHDEBINFO "-flto -fuse-linker-plugin ${CMAKE_C_FLAGS_RELWITHDEBINFO} -g -O3 -UNDEBUG")
#  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-flto -fuse-linker-plugin ${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -g -O3 -UNDEBUG")
#  set(CMAKE_C_FLAGS_RELEASE "-g -O3 -flto -fuse-linker-plugin ${CMAKE_C_FLAGS_RELEASE} -UNDEBUG")
#  set(CMAKE_CXX_FLAGS_RELEASE "-g -O3 -flto -fuse-linker-plugin ${CMAKE_CXX_FLAGS_RELEASE} -UNDEBUG")
#  set(CMAKE_EXE_LINKER_FLAGS "-g -fuse-linker-plugin ${CMAKE_EXE_LINKER_FLAGS}")
#  set(CMAKE_SHARED_LINKER_FLAGS "-g -fuse-linker-plugin ${CMAKE_SHARED_LINKER_FLAGS}")

  set(CMAKE_C_FLAGS_RELWITHDEBINFO "-flto  ${CMAKE_C_FLAGS_RELWITHDEBINFO} -g -O3 -UNDEBUG")
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-flto ${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -g -O3 -UNDEBUG")
  set(CMAKE_C_FLAGS_RELEASE "-g -O3 -flto  ${CMAKE_C_FLAGS_RELEASE} -UNDEBUG")
  set(CMAKE_CXX_FLAGS_RELEASE "-g -O3 -flto  ${CMAKE_CXX_FLAGS_RELEASE} -UNDEBUG")
  set(CMAKE_EXE_LINKER_FLAGS "-g  ${CMAKE_EXE_LINKER_FLAGS}")
  set(CMAKE_SHARED_LINKER_FLAGS "-g  ${CMAKE_SHARED_LINKER_FLAGS}")
```

然后是/home/gao-compile/mariadb-10.0.14/storage/tokudb/CMakeLists.txt
去掉所有的 -fuse-linker-plugin ：
```bash
#SET(CMAKE_MODULE_LINKER_FLAGS_RELEASE "${CMAKE_MODULE_LINKER_FLAGS_RELEASE} -flto -fuse-linker-plugin")
#SET(CMAKE_MODULE_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_MODULE_LINKER_FLAGS_RELWITHDEBINFO} -flto -fuse-linker-plugin")

SET(CMAKE_MODULE_LINKER_FLAGS_RELEASE "${CMAKE_MODULE_LINKER_FLAGS_RELEASE} -flto")
SET(CMAKE_MODULE_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_MODULE_LINKER_FLAGS_RELWITHDEBINFO} -flto")
```

然后进行编译：
```bash
cd mariadb-10.0.14
mkdir release
./BUILD/compile-pentium64-max --prefix=/home/gao-compile/mariadb-10.0.14/release
make install
```

然后在/home/gao-compile/mariadb-10.0.14/release/lib/plugin 目录中找到 ha_tokudb.so动态库。
这个就是我们要的东西了！

注：其实可以直接在mariadb-10.0.14/storage/tokudb目录中直接进行编译，而不用去编译整个mariadb，因为编译出来的也无法使用， mysqld可执行程序依赖GLIBCXX_3.4.15
```bash
[root@GXX release]# ldd bin/mysqld
bin/mysqld: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.15' not found (required by bin/mysqld)
```

在mariadb官网https://downloads.mariadb.org/mariadb/10.0.14/下载带有mariadb的二进制包。从mariadb官网下载10.0.14的不带tokudb的可执行程序tar包，然后将我们编译好的tokudb.so拷贝到
[maradb10.0.14]/lib/plugin/下
我这里下载的包，10.0.14
mariadb-10.0.14-linux-x86_64.tar.gz 

###四、初始化
####1、前期账号创建
```bash
shell> groupadd mysql
shell> useradd -g mysql mysql
```

####2、解压软件包
```bash
shell> cd /usr/local
shell> tar -zxvf mariadb-10.0.14-linux-x86_64.tar.gz
shell> ln -s mariadb-10.0.14-linux-x86_64 mysql
```

####3、设置权限
```bash
shell> cd mysql
shell> chown -R mysql .
shell> chgrp -R mysql .
```

####4、创建配置文件/etc/my.cnf
创建存放数据源的目录
```php
Shell> mkdir -p /var/lib/mysql
Shell> mkdir -p /data/mysqldata /data/mysqllog
Shell> chown -R mysql:mysql /var/lib/mysql /data/mysqldata /data/mysqllog
```

在附件中给出，请参考附配置文件内容


####5、初始化DB
```php
Shell> scripts/mysql_install_db --user=mysql --defaults-file=/etc/my.cnf
shell> chown -R root .
```

####6、修改系统参数
```php
echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag 
```
建议写到 /etc/rc.local 中，重启后也可生效

####7、启动数据库
```php
shell> bin/mysqld_safe --user=mysql &
```

####8、把启动文件放去自启动中
```php
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
service mysqld start
service mysqld stop
```
####9、安装tokudb支持
```bash
INSTALL SONAME 'ha_tokudb';
```
查看是否支持
```bash
MariaDB [(none)]> show engines
    -> ;
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                                    | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| MyISAM             | YES     | MyISAM storage engine                                                      | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                                         | NO           | NO   | NO         |
| MRG_MyISAM         | YES     | Collection of identical MyISAM tables                                      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears)             | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables                  | NO           | NO   | NO         |
| TokuDB             | YES     | Tokutek TokuDB Storage Engine with Fractal Tree(tm) Technology             | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                                         | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                                     | NO           | NO   | NO         |
| FEDERATED          | YES     | FederatedX pluggable storage engine                                        | YES          | NO   | YES        |
| InnoDB             | DEFAULT | Percona-XtraDB, Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| Aria               | YES     | Crash-safe tables with MyISAM heritage                                     | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
11 rows in set (0.11 sec)
```
看到上面一行红字就代表数据库支持TokuDB的存储引擎了。

tokudb安装顺利完成，后附配置文件
```bash
[root@localhost mysql]# cat /etc/my.cnf 
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
#public
max_connections=3000
max_connect_errors=6000
key_buffer_size = 384M
low_priority_updates = 1
back_log = 1500
query_cache_type = 1
query_cache_size = 64M
query_cache_limit = 4M
query_cache_min_res_unit = 2k
tmp_table_size = 256M
read_buffer_size=1M
read_rnd_buffer_size = 16M
bulk_insert_buffer_size = 64M
max_allowed_packet = 64M
thread_cache_size = 300
#file
innodb_file_per_table
#buffer
innodb_buffer_pool_size = 2500M
innodb_log_buffer_size = 64M
join_buffer_size = 16M
sort_buffer_size = 16M
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120
innodb_thread_concurrency = 16
innodb_flush_log_at_trx_commit = 2
character_set_server=utf8
init_connect='SET NAMES utf8'

#把TokuDB datadir以及logdir和MySQL的datadir分开，美观点，也可以不分开，注释掉本行以及下面2行即可
tokudb_data-dir = /data/mysqldata
tokudb_log-dir = /data/mysqllog
#TokuDB的行模式，建议用 FAST 就足够了，如果磁盘空间很紧张，建议用 SMALL
tokudb_row_format = tokudb_small
tokudb_row_format = tokudb_fast
tokudb_cache_size = 2G
#其他大部分配置其实可以不用修改的，只需要几个关键配置即可
tokudb_commit_sync = 0
tokudb_directio = 1
tokudb_read_block_size = 128K
tokudb_read_buf_size = 128K

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```