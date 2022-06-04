title: infiniDB与clickhouse对比
date: 2016-08-22 13:55:37
categories: mysql
tags: [clickhouse]
---

机器配置：

测试机：	dell R510
cpu： 	xeon E5620*2
RAID卡：	H700 
系统	：ubuntu 14.04
内核：	Linux ubuntu 4.2.0-27-generic
raid级别：	raid0
文件系统：	ext4
硬盘：	 SSD 240G*2 
IP：	192.168.112.73

一、安装
1、infiniDB的安装见 http://verynull.com/2015/06/24/infiniDB安装配置/

2、clickhouse安装
只支持ubutu14.04,16.04，12.04，但是我在16.04上没有安装起来。
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E0C56BD4# optional

sudo mkdir -p /etc/apt/sources.list.d
echo "deb http://repo.yandex.ru/clickhouse/trusty stable main" |
sudo tee /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install clickhouse-server-common clickhouse-client

sudo service clickhouse-server start
clickhouse-client
```

3、就安装过程而言，clickhouse方便很多，infiniDB要编译老半天。

二、性能测试

1、创建表

infiniDB:
```bash
CREATE TABLE `billing` (
`did` int(11) NOT NULL DEFAULT '0' COMMENT '',
`hit` bigint(20) NOT NULL DEFAULT '0' COMMENT '',
`reqs` bigint(20) NOT NULL DEFAULT '0' COMMENT '',
`rx` bigint(20) NOT NULL DEFAULT '0' COMMENT '',
`tx` bigint(20) NOT NULL DEFAULT '0' COMMENT '',
`logtime` bigint(20) NOT NULL DEFAULT '0' COMMENT '',
`tpr` bigint(20) NOT NULL DEFAULT '0' COMMENT ''
) ENGINE=infiniDB DEFAULT CHARSET=latin1
```

clickhouse:
```bash
 CREATE TABLE billing (id Int32,did Int32,hit Int64,reqs Int64,rx Int64,tx Int64,logtime Int64,tpr Int64) ENGINE=Log;
```


2、导入csv数据

csv格式如下：
```bash
106,9,10,123,11858014,1467133200,330,0
122,0,2,506,621,1467133200,150,0
124,112,249,1131297,1899996,1467133200,49,0
127,3,12,64257,88503,1467133200,66,0
```

数据量9000万,大小4.5G。

infiniDB:
```bash
sudo /data/mysql/bin/mysql --defaults-file=/data/mysql/my.cnf --local-infile -u root monitor -e "load data local infile '/root/1.txt' into table billing FIELDS TERMINATED BY ',';"
```

耗时：
```bash
0.60user 
2.50system 
4:14.61elapsed 1%CPU (0avgtext+0avgdata 3640maxresident)k
0inputs+0outputs (0major+227minor)pagefaults 0swaps
```

clickhouse:
```bash
cat 1.txt | clickhouse-client --query="INSERT INTO monitor.monitor_data FORMAT CSV";
```

耗时：
```bash
real1m6.809s
user1m26.720s
sys 0m6.724s
```

clickhouse导入速度明显快很多。


3、查询速度

SQL:
```bash
SELECT logtime, SUM(hit) as hit, SUM(reqs) as reqs, SUM(rx) as rx,SUM(tx) as tx, round(avg(tpr)) as tpr FROM billingWHERE did > 0AND logtime >= 1467302400 AND logtime <= 1469980799 GROUP BY logtime ORDER BY l
ogtime;
```

infiniDB:
```bash
9663 rows in set (1.26 sec)
```

clickhouse:
```bash
8924 rows in set. Elapsed: 2.676 sec. Processed 91.80 million rows, 4.77 GB (34.30 million rows/s., 1.78 GB/s.) 
```

infiniDB速度较快。

4、2亿数据对比

SQL同上

infiniDB:
```bash
9663 rows in set (2.24 sec)
```

clickhouse:
```bash
8924 rows in set. Elapsed: 4.215 sec. Processed 183.61 million rows, 9.55 GB (43.56 million rows/s., 2.26 GB/s.) 
```

infiniDB较快。

5、10亿数据对比

SQL同上

infiniDB:
```bash
9663 rows in set (9.58 sec)
```

clickhouse:
```bash
8924 rows in set. Elapsed: 17.040 sec. Processed 1.01 billion rows, 52.51 GB (56.67 million rows/s., 2.95 GB/s.) 
```

infiniDB更快。

6、count对比
infiniDB:
```bash
+------------+
| count(*) |
+------------+
| 1009844583 |
+------------+
1 row in set (6.14 sec)
```

clickhouse:
```bash
SELECT count(*)
FROM billing 

┌────count()─┐
│ 1009844583 │
└────────────┘

1 rows in set. Elapsed: 1.120 sec. Processed 1.01 billion rows, 4.04 GB (901.82 million rows/s., 3.61 GB/s.) 
```

7、压缩比
infiniDB:
```bash
22G billing
```

clickhouse:
```bash
24G billing
```

压缩比相差不多。


结果：
1、infiniDB会使用所有CPU，而clickhouse最多使用10个。
2、clickhouse在大数据量的时候，处理速度还是比较令人满意的。
3、clickhouse在导入数据的时候，查询会被卡住，而infiniDB不会。
4、infiniDB性能更好。

附对比图：
这里clickhouse是Log引擎。
![对比图](https://static.verycloud.cn/sites/default/files/pic/image/20160822/20160822222132_99553.png)

--------------------------------------------------------------------------------------------
经一位俄罗斯网友提醒,MergeTree引擎比Log引擎更高效，因为Log引擎会全表扫描，而MergeTree的结构是这样的：

```bash
xxxx@ubuntu:/opt/clickhouse$ ls data/monitor/billing_mergetree
20150829_20150831_10964_10994_2
20160101_20160131_10_11360_30 
20160401_20160430_12_9964_31
20160609_20160613_11302_11350_4
20160808_20160820_10972_11250_2
20150325_20150331_4_9822_23 
20150901_20150930_1030_11316_50
20160201_20160229_244_10606_27
20160401_20160430_9982_11362_10 
20160701_20160731_308_8668_14 
20160820_20160820_11260_11260_0
20150401_20150430_16_9932_57
20151001_20151031_8_11352_39 
20160220_20160229_10618_10788_10
20160501_20160531_530_11042_23
20160701_20160731_8678_10786_12
20160820_20160820_11266_11266_0
20150501_20150531_102_10032_75
20151009_20151011_11356_11356_0
20160229_20160229_10804_10812_1 
20160528_20160531_11056_11124_6 
20160731_20160731_10798_10798_0
detached
```

这种存储结构就是把每个月的数据单独放置在一个文件夹中，这样就避免了全表扫描。

附对比图:
![对比图](https://static.verycloud.cn/sites/default/files/pic/image/20160823/20160823113203_30109.png)

结论：
clickhouse速度更快！