title: centos6.6安装Percona Tokudb
date: 2015-11-21 20:17:57
categories: mysql
tags: [tokudb]
---

在percona官网https://www.percona.com/
下载带有Tokudb的二进制包，也可以选择源码编译。tokudb有单独的包. 
我这里下载的是最新的包，5.6.25-73
Percona-Server-5.6.25-rel73.1-Linux.x86_64.ssl101.tar.gz
Percona-Server-5.6.25-rel73.1-TokuDB.Linux.x86_64.ssl101.tar.gz

###前期账号创建
```php
shell> groupadd mysql
shell> useradd -g mysql mysql
```

###解压软件包
```php
shell> cd /usr/local
shell> tar -zxvf Percona-Server-5.6.25-rel73.1-Linux.x86_64.ssl101.tar.gz
shell> tar -zxvf Percona-Server-5.6.25-rel73.1-TokuDB.Linux.x86_64.ssl101.tar.gz
shell> ln -s Percona-Server-5.6.25-rel73.1-Linux.x86_64.ssl101 mysql
```

###设置权限
```php
shell> cd mysql
shell> chown -R mysql .
shell> chgrp -R mysql .
```

###创建配置文件/etc/my.cnf
创建存放数据源的目录

```php 
Shell> mkdir -p /var/lib/mysql
Shell> mkdir -p /data/mysqldata /data/mysqllog
Shell> chown -R mysql:mysql /var/lib/mysql /data/mysqldata /data/mysqllog
```

在附件中给出，请参考附配置文件内容

###初始化DB

```php
Shell> scripts/mysql_install_db --user=mysql --defaults-file=/etc/my.cnf
shell> chown -R root .
```

###修改系统参数

```php
echo never > /sys/kernel/mm/redhat_transparent_hugepage/defrag
echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```
建议写到 /etc/rc.local 中，重启后也可生效

###启动数据库

```php
shell> bin/mysqld_safe --user=mysql &
```

###把启动文件放去自启动中

```php
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
service mysqld start
service mysqld stop
```

然后启动。

```php
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                                    | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Percona-XtraDB, Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| CSV                | YES     | CSV storage engine                                                         | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                                      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears)             | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables                  | NO           | NO   | NO         |
| TokuDB             | YES     | Tokutek TokuDB Storage Engine with Fractal Tree(tm) Technology             | YES          | YES  | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                                         | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                                     | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                                      | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                             | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
10 rows in set (0.00 sec)
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

#tokudb
plugin_dir =  /usr/local/mysql/lib/mysql/plugin/
plugin_load=ha_tokudb.so
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
malloc_lib = /usr/local/mysql/lib/mysql/libjemalloc.so #只能放在mysqld_safe中，不得报错
```