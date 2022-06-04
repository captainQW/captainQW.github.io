title: sysbench测试mysql性能
date: 2016-01-29 16:15:54
categories: mysql
tags: [sysbench]
---

sysbench是一个模块化的、跨平台、多线程基准测试工具，主要用于评估测试各种不同系统参数下的数据库负载情况。关于这个项目的详细介绍请看：https://launchpad.net/sysbench。
它主要包括以下几种方式的测试：
1、cpu性能
2、磁盘io性能
3、调度程序性能
4、内存分配及传输速度
5、POSIX线程性能
6、数据库性能(OLTP基准测试)
目前sysbench主要支持 MySQL,pgsql,oracle 这3种数据库。



###一、安装

环境：

centos 6.7
mysql  5.7.10

```bash
yum install automake gcc gcc-c++ libtool mysql-devel

git clone https://github.com/akopytov/sysbench.git

./autogen.sh
./configure
make && make install
```

如果想要让 sysbench 支持 pgsql/oracle 的话，就需要在编译的时候加上参数
--with-pgsql
或者
--with-oracle
这2个参数默认是关闭的，只有 MySQL 是默认支持的。


###二、测试

#####1、cpu性能测试

```bash
sysbench --test=cpu --cpu-max-prime=20000 run
```

cpu测试主要是进行素数的加法运算，在上面的例子中，指定了最大的素数为 20000，自己可以根据机器cpu的性能来适当调整数值。

#####2、线程测试

```bash
sysbench --test=threads --num-threads=64 --thread-yields=100 --thread-locks=2 run
```

#####3、磁盘IO性能测试

```bash
sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw prepare
sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw run
sysbench --test=fileio --num-threads=16 --file-total-size=3G --file-test-mode=rndrw cleanup
```

上述参数指定了最大创建16个线程，创建的文件总大小为3G，文件读写模式为随机读。

#####4、内存测试

```bash
sysbench --test=memory --memory-block-size=8k --memory-total-size=4G run
```

上述参数指定了本次测试整个过程是在内存中传输 4G 的数据量，每个 block 大小为 8K。

#####5、OLTP测试

```bash
sysbench --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root --mysql-password=root  --test=/root/sysbench/sysbench/tests/db/oltp.lua --oltp_tables_count=10 --oltp-table-size=100000 --rand-init=on prepare
```

关于这几个参数的解释：

--test=tests/db/oltp.lua 表示调用 tests/db/oltp.lua 脚本进行 oltp 模式测试
--oltp_tables_count=10 表示会生成 10 个测试表
--oltp-table-size=100000 表示每个测试表填充数据量为 100000 
--rand-init=on 表示每个测试表都是用随机数据来填充的
如果在本机，也可以使用 –mysql-socket 指定 socket 文件来连接。加载测试数据时长视数据量而定，若过程比较久需要稍加耐心等待。

真实测试场景中，数据表建议不低于10个，单表数据量不低于500万行，当然了，要视服务器硬件配置而定。如果是配备了SSD或者PCIE SSD这种高IOPS设备的话，则建议单表数据量最少不低于1亿行。

```bash
sysbench --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=root \
--mysql-password=root --test=/root/sysbench/sysbench/tests/db/oltp.lua --oltp_tables_count=10 \
--oltp-table-size=10000000 --num-threads=8 --oltp-read-only=off \
--report-interval=10 --rand-type=uniform --max-time=3600 \
 --max-requests=0 --percentile=99 run >> /tmp/sysbench_oltpX_8.log
```

参数解释：
--num-threads=8 表示发起 8个并发连接
--oltp-read-only=off 表示不要进行只读测试，也就是会采用读写混合模式测试
--report-interval=10 表示每10秒输出一次测试进度报告
--rand-type=uniform 表示随机类型为固定模式，其他几个可选随机模式：uniform(固定),gaussian(高斯),special(特定的),pareto(帕累托)
--max-time=120 表示最大执行时长为 120秒
--max-requests=0 表示总请求数为 0，因为上面已经定义了总执行时长，所以总请求数可以设定为 0；也可以只设定总请求数，不设定最大执行时长
--percentile=99 表示设定采样比例，默认是 95%，即丢弃1%的长请求，在剩余的99%里取最大值

即：模拟 对10个表并发OLTP测试，每个表1000万行记录，持续压测时间为 1小时。

真实测试场景中，建议持续压测时长不小于30分钟，否则测试数据可能不具参考意义。


#####6、测试结果解读

测试结果解读如下：

```bash
sysbench 0.5:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 8
Report intermediate results every 10 second(s)
Random number generator seed is 0 and will be ignored


Threads started!
-- 每10秒钟报告一次测试结果，tps、每秒读、每秒写、99%以上的响应时长统计
[  10s] threads: 8, tps: 1111.51, reads/s: 15568.42, writes/s: 4446.13, response time: 9.95ms (99%)
[  20s] threads: 8, tps: 1121.90, reads/s: 15709.62, writes/s: 4487.80, response time: 9.78ms (99%)
[  30s] threads: 8, tps: 1120.00, reads/s: 15679.10, writes/s: 4480.20, response time: 9.84ms (99%)
[  40s] threads: 8, tps: 1114.20, reads/s: 15599.39, writes/s: 4456.30, response time: 9.90ms (99%)
[  50s] threads: 8, tps: 1114.00, reads/s: 15593.60, writes/s: 4456.70, response time: 9.84ms (99%)
[  60s] threads: 8, tps: 1119.30, reads/s: 15671.60, writes/s: 4476.50, response time: 9.99ms (99%)
OLTP test statistics:
    queries performed:
        read:                            938224    -- 读总数
        write:                           268064    -- 写总数
        other:                           134032    -- 其他操作总数(SELECT、INSERT、UPDATE、DELETE之外的操作，例如COMMIT等)
        total:                           1340320    -- 全部总数
    transactions:                        67016  (1116.83 per sec.)    -- 总事务数(每秒事务数)
    deadlocks:                           0      (0.00 per sec.)    -- 发生死锁总数
    read/write requests:                 1206288 (20103.01 per sec.)    -- 读写总数(每秒读写次数)
    other operations:                    134032 (2233.67 per sec.)    -- 其他操作总数(每秒其他操作次数)

General statistics:    -- 一些统计结果
    total time:                          60.0053s    -- 总耗时
    total number of events:              67016    -- 共发生多少事务数
    total time taken by event execution: 479.8171s    -- 所有事务耗时相加(不考虑并行因素)
    response time:    -- 响应时长统计
         min:                                  4.27ms    -- 最小耗时
         avg:                                  7.16ms    -- 平均耗时
         max:                                 13.80ms    -- 最长耗时
         approx.  99 percentile:               9.88ms    -- 超过99%平均耗时

Threads fairness:
    events (avg/stddev):           8377.0000/44.33
    execution time (avg/stddev):   59.9771/0.00
```

参考文章：
1、http://imysql.com/2014/10/17/sysbench-full-user-manual.shtml
2、http://imysql.com/2015/07/28/mysql-benchmark-reference.shtml