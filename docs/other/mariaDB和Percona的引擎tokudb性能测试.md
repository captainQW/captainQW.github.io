title: MariaDB和Percona的引擎tokudb性能对比测试
date: 2015-11-08 20:30:12
categories: mysql
tags: [tokudb]
---

MariaDB和Mysql对比（配置4核4G）:

```php
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.25-73.1 Percona Server (GPL), Release 73.1, Revision 07b797f
```

```php
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.0.14-MariaDB MariaDB Server
```

###1、MariaDB导入数据：
```php
MariaDB [billing]> load data infile '/home/user_logs.txt' into table cloud_innodb FIELDS TERMINATED BY '\t';
Query OK, 641098 rows affected, 65535 warnings (1 min 18.02 sec)
Records: 641098  Deleted: 0  Skipped: 0  Warnings: 641098

MariaDB [billing]> load data infile '/home/user_logs.txt' into table cloud_tokudb FIELDS TERMINATED BY '\t';
Query OK, 641098 rows affected, 65535 warnings (36.83 sec)                                 
Records: 641098  Deleted: 0  Skipped: 0  Warnings: 641098
```

Mysql导入数据：
```php
mysql> load data infile '/home/user_logs.txt' into table cloud_innodb FIELDS TERMINATED BY '\t';
Query OK, 641098 rows affected, 65535 warnings (1 min 10.75 sec)
Records: 641098  Deleted: 0  Skipped: 0  Warnings: 641098

mysql> load data infile '/home/user_logs.txt' into table cloud_tokudb FIELDS TERMINATED BY '\t';
Query OK, 641098 rows affected, 65535 warnings (33.35 sec)
Records: 641098  Deleted: 0  Skipped: 0  Warnings: 641098
```

总结：
1）、MariaDB导入数据比Mysql要慢一点
2）、TokuDB导入数据比InnoDB速度快一倍
3）、TokuDB采用多CPU并发导入


###2、MariaDB存储空间
```php
[root@localhost billing]# ll -h
-rw-rw----. 1 mysql mysql 1.5K Nov  4 14:16 cloud_innodb.frm
-rw-rw----. 1 mysql mysql 568M Nov  4 14:40 cloud_innodb.ibd

[root@localhost mysqldata]# ll -h
-rw-rw----. 1 mysql mysql  26M Nov  4 14:31 _billing_cloud_tokudb_main_172d_1_1b_B_0.tokudb
-rw-rw----. 1 mysql mysql  64K Nov  4 14:31 _billing_cloud_tokudb_status_1729_1_1b.tokudb
```

Mysql存储空间
```php
[root@localhost billing]# ll -h 
-rw-rw----. 1 mysql mysql 8.9K Nov  4 14:17 cloud_innodb.frm
-rw-rw----. 1 mysql mysql 568M Nov  4 14:36 cloud_innodb.ibd


[root@localhost mysqldata]# ll -h
-rw-rw----. 1 mysql mysql  26M Nov  4 14:37 _billing_cloud_tokudb_main_8be1_1_1c_B_0.tokudb
-rw-rw----. 1 mysql mysql  64K Nov  4 14:37 _billing_cloud_tokudb_status_8bde_1_1c.tokudb
```

总结：
1）、 MariaDB和Mysql在采用相同引擎的时候存储空间相同
2）、 TokuDB的压缩有很多种，上面采用tokudb_lzma,压缩比例最大，但非常消耗CPU，内存
3）、 TokuDB和InnoDB从上面的例子来看压缩比率是1：20左右，如果都是整型数据，压缩比率在1：4左右。

###3、MariaDB查询速度
重启第一次查询
```php
MariaDB [billing]> select count(*) from cloud_innodb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (0.58 sec)

MariaDB [billing]> select count(*) from cloud_tokudb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (16.55 sec)
```

清缓存（reset query cache）
```php
MariaDB [billing]> reset query cache;
Query OK, 0 rows affected (0.02 sec)

MariaDB [billing]> select count(*) from cloud_innodb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (0.42 sec)

MariaDB [billing]> select count(*) from cloud_tokudb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (0.43 sec)
```

Mysql查询速度
```php
mysql> select count(*) from cloud_innodb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (0.66 sec)

mysql> select count(*) from cloud_tokudb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (13.74 sec)
```

清缓存（reset query cache）
```php
mysql> reset query cache;
Query OK, 0 rows affected (0.03 sec)

mysql> select count(*) from cloud_innodb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (0.38 sec)

mysql> select count(*) from cloud_tokudb;
+----------+
| count(*) |
+----------+
|   641098 |
+----------+
1 row in set (0.49 sec)
```

总结：
1）、 查询速度MariaDB和Mysql速度差不多
2）、 第一次查询的时候InnoDB和TokuDB速度相差很大
3）、 清缓存后查询差不多

###4、并发查询
MariaDB：
```php
[root@localhost mysqldata]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select count(content) from cloud_innodb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 0.509 seconds
	Minimum number of seconds to run all queries: 0.509 seconds
	Maximum number of seconds to run all queries: 0.509 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0

[root@localhost mysqldata]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select count(content) from cloud_tokudb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 0.600 seconds
	Minimum number of seconds to run all queries: 0.600 seconds
	Maximum number of seconds to run all queries: 0.600 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0

[root@localhost mysqldata]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select content from cloud_innodb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 0.901 seconds
	Minimum number of seconds to run all queries: 0.901 seconds
	Maximum number of seconds to run all queries: 0.901 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0

[root@localhost mysqldata]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select content from cloud_tokudb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 1.028 seconds
	Minimum number of seconds to run all queries: 1.028 seconds
	Maximum number of seconds to run all queries: 1.028 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0
```

Mysql：
```php
[root@localhost billing]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select count(content) from cloud_innodb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 0.496 seconds
	Minimum number of seconds to run all queries: 0.496 seconds
	Maximum number of seconds to run all queries: 0.496 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0

[root@localhost billing]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select count(content) from cloud_tokudb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 0.483 seconds
	Minimum number of seconds to run all queries: 0.483 seconds
	Maximum number of seconds to run all queries: 0.483 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0

[root@localhost billing]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select content from cloud_innodb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 1.231 seconds
	Minimum number of seconds to run all queries: 1.231 seconds
	Maximum number of seconds to run all queries: 1.231 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0

[root@localhost billing]# mysqlslap --concurrency=1000 --iterations=1 --create-schema='billing' --query='select content from cloud_tokudb where id >9950 and id < 9999;' --number-of-queries=10
Benchmark
	Average number of seconds to run all queries: 0.739 seconds
	Minimum number of seconds to run all queries: 0.739 seconds
	Maximum number of seconds to run all queries: 0.739 seconds
	Number of clients running queries: 1000
	Average number of queries per client: 0
```

总结
1）、 并发查询两数据相差不是很大
2）、 Tokudb查询速度在MariaDB中比InnoDB引擎慢一点，但在Mysql中比InnoDB引擎速度还快（网上说的TokuDB查询速度不行，没有验证出来，不知道真正在线速度怎么样，按理说压缩后的数据应该查询要慢于没有压缩的数据）

###5、 批量更新

MariaDB 
```php
[billing]> update  cloud_innodb set content="ddddddddddddddddddddsssssss" where id >8900 and id < 90000;
Query OK, 81099 rows affected (5.71 sec)
Rows matched: 81099  Changed: 81099  Warnings: 0
```

MariaDB 
```php
[billing]> update  cloud_tokudb set content="ddddddddddddddddddddsssssss" where id >8900 and id < 90000;
Query OK, 81099 rows affected (3.46 sec)
```

```php
mysql> update  cloud_innodb set content="ddddddddddddddddddddsssssss" where id >8900 and id < 90000;
Query OK, 80952 rows affected (5.32 sec)
Rows matched: 81099  Changed: 80952  Warnings: 0

mysql> update  cloud_tokudb set content="ddddddddddddddddddddsssssss" where id >8900 and id < 90000;
Query OK, 80952 rows affected (3.40 sec)
Rows matched: 81099  Changed: 80952  Warnings: 0
```

总结
1）、两数据库性能差不多
2）、批量更新感觉TokuDB速度比InnoDB速度快（和网上说的又不一样）

###6、DDL操作对比
添加索引
```php
MariaDB [billing]> create index idx_uid on cloud_innodb(uid);
Query OK, 0 rows affected (12.23 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [billing]> create index idx_uid on cloud_tokudb(uid);
Query OK, 0 rows affected (29.51 sec)                                   
Records: 0  Duplicates: 0  Warnings: 0

mysql> create index idx_uid on cloud_innodb(uid);
Query OK, 0 rows affected (15.48 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> create index idx_uid on cloud_tokudb(uid);
Query OK, 0 rows affected (31.10 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除索引
```php
MariaDB [billing]> drop index idx_uid on cloud_innodb;
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [billing]> drop index idx_uid on cloud_tokudb;
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> drop index idx_uid on cloud_innodb;
Query OK, 0 rows affected (0.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> drop index idx_uid on cloud_tokudb;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

添加字段
```php
MariaDB [billing]> alter table cloud_innodb add column test_flag varchar(20);
Query OK, 0 rows affected (1 min 20.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [billing]> alter table cloud_tokudb add column test_flag varchar(20);
Query OK, 0 rows affected (0.16 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table cloud_innodb add column test_flag varchar(20);
Query OK, 0 rows affected (1 min 31.07 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table cloud_tokudb add column test_flag varchar(20);
Query OK, 0 rows affected (0.23 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除字段
```php
MariaDB [billing]> alter table cloud_innodb drop column test_flag;
Query OK, 0 rows affected (1 min 19.48 sec)
Records: 0  Duplicates: 0  Warnings: 0

MariaDB [billing]> alter table cloud_tokudb drop column test_flag;
Query OK, 0 rows affected (0.23 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table cloud_innodb drop column test_flag;
Query OK, 0 rows affected (1 min 55.31 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table cloud_tokudb drop column test_flag;
Query OK, 0 rows affected (1.28 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
总结
1、MariaDB的DDL操作比Mysql的快
2、InnoDB的添加索引比TokuDB的快
3、删除索引两个引擎都很快
4、添加字段TokuDB非常快，InnoDB很慢
5、删除字段TokuDB非常快，InnoDB很慢


网上的一些总结
======================================================
总结：
TokuDB优点
1，online ddl 非常给力，特别是对字段的修改非常快
2，压缩比非常高通常都能达到7,8倍的压缩比
3，完全支持ACID事物的四大特性
TokuDB缺点
1，响应时间相对较长
2，online ddl 对text,blob等类型的字段不适用
3，没有合适的备份工具，只能通过mysqldump进行逻辑备份
建议适用场景：
1，访问频率不高的数据或历史数据归档
2，表非常大并且时不时还需要进行ddl操作

特点：
1.插入性能快20~80倍；
2.压缩数据减少存储空间；
3.数据量可以扩展到几个TB；
4.不会产生索引碎片；
5.支持hot column addition ， hot indexing， mvcc；
如何考虑使用：
1.如果要存储blob，不要使用tokuDB，因为他的记录不能太大；
2.如果记录数过亿，使用tokuDB；
3.如果注重update的性能，不要使用tokuDB，他没有innodb快；
4.如果要存储旧的记录，使用tokuDB；
5.如果要缩小数据占用的存储空间，使用tokuDB；

总结： 
TokuDB的优点：1、高压缩比 2、高insert性能 3、增删字段秒级。
TokuDB的缺点：1、cpu usr态消耗高 2、响应时间变长。