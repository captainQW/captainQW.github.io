title: Mariadb ColumnStore数据库测试
date: 2016-12-20 15:06:53
categories: mysql
tags: [ColumnStore]
---

顾名思义，就是Mariadb为了大数据而开发的列式数据库。后来发现，这货就是InfiniDB的翻版，InfiniDB被MariaDB收购了以后重新开发的新版本。

```bash
MariaDB [(none)]> show engines;
+--------------------+---------+--------------------------------------------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                                                          | Transactions | XA   | Savepoints |
+--------------------+---------+--------------------------------------------------------------------------------------------------+--------------+------+------------+
| Columnstore        | YES     | Columnstore storage engine                                                                       | YES          | NO   | NO         |
| MRG_MyISAM         | YES     | Collection of identical MyISAM tables                                                            | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                                                               | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                                                            | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables                                        | NO           | NO   | NO         |
| InfiniDB           | YES     | Columnstore storage engine (deprecated: use columnstore)                                         | YES          | NO   | NO         |
| InnoDB             | DEFAULT | Percona-XtraDB, Supports transactions, row-level locking, foreign keys and encryption for tables | YES          | YES  | YES        |
| SEQUENCE           | YES     | Generated tables filled with sequential values                                                   | YES          | NO   | YES        |
| Aria               | YES     | Crash-safe tables with MyISAM heritage                                                           | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                                                               | NO           | NO   | NO         |
+--------------------+---------+--------------------------------------------------------------------------------------------------+--------------+------+------------+
10 rows in set (0.00 sec)
```

还能看到InfiniDB引擎的影子。

测试环境：
centos 6.6 8核8G
ssd 100G
mariadb columnstore 1.0.6 GA

1、安装

安装依赖
```bash
# yum -y install expect perl perl-DBI openssl zlib perl-DBD-MySQL cmake
```

必须要boost1.50以上版本。

```bash
# wget https://static.verycloud.cn/rpm/boost_1_55_0.tgz
# wget https://static.verycloud.cn/rpm/mariadb-columnstore-1.0.6-1-centos6.x86_64.rpm.tar.gz
```

安装boost.
```bash
# tar -zxvf boost_1_55_0.tgz
# tar -zxvf mariadb-columnstore-1.0.6-1-centos6.x86_64.rpm.tar.gz
# cd boost_1_55_0 
# ./bootstrap.sh --with-libraries=atomic,date_time,exception,filesystem,iostreams,locale,program_options,regex,signals,system,test,thread,timer,log --prefix=/usr 
# ./b2 install
```

安装mariadb columnstore.
```bash
# rpm -ivh mariadb-columnstore-1.0.6-1-x86_64-centos6-*
```

2、初始化
```bash
# /usr/local/mariadb/columnstore/bin/post-install
```

这时会提示
```bash
The next step is:

/usr/local/mariadb/columnstore/bin/postConfigure
```

执行，完成安装，选择single server
```bash
# /usr/local/mariadb/columnstore/bin/postConfigure
```

生成别名
```bash
# . /data/mariadb/columnstore/bin/columnstoreAlias
```

执行 mcsmysql，会看到提示
```bash
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.1.19-MariaDB Columnstore 1.0.6-1

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

说明安装成功了！

3、数据导入

执行
```bash
# mcsmysql --local-infile
# LOAD DATA LOCAL INFILE '/root/data.txt' INTO TABLE your_table FIELDS TERMINATED BY '\t';
```

```bash
Query OK, 95981370 rows affected (6 min 8.62 sec)    
Records: 95981370  Deleted: 0  Skipped: 0  Warnings: 0
```
导入1亿条数据要6分钟。