title: tpcc测试mysql性能
date: 2016-01-29 17:05:25
categories: mysql
tags: [sysbench]
---

TPC-C是专门针对联机交易处理系统（OLTP系统）的规范，一般情况下我们也把这类系统称为业务处理系统。
tpcc-mysql是percona基于TPC-C(下面简写成TPCC)衍生出来的产品，专用于MySQL基准测试。其源码放在launchpad上，用bazaar管理，项目地址：https://code.launchpad.net/~percona-dev/perconatools/tpcc-mysql。

也可以用叶金荣的版本，https://github.com/yejr/tpcc-mysql-autoinc-pk。

tpcc-mysql的业务逻辑及其相关的几个表作用如下：

New-Order：新订单，一次完整的订单事务，几乎涉及到全部表
Payment：支付，主要对应 orders、history 表
Order-Status：订单状态，主要对应 orders、order_line 表
Delivery：发货，主要对应 order_line 表
Stock-Level：库存，主要对应 stock 表

其他相关表：
客户：主要对应 customer 表
地区：主要对应 district 表
商品：主要对应 item 表
仓库：主要对应 warehouse 表

###一、编译安装
编译非常简单，只需要一个 make 即可。

```bash
cd /tmp/tpcc-mysql/src
make
```

如果 make 没有报错，就会在 /tmp/tpcc-mysql 下生成 tpcc 二进制命令行工具 tpcc_load、 tpcc_start

###二、TPCC测试前准备
初始化测试库环境

```bash
cd /tmp/tpcc-mysql
mysqladmin create tpcc
mysql -uroot -proot tpcc < create_table.sql
```

tpcc_load用法如下：

```bash
tpcc_load [server] [DB] [user] [pass] [warehouse]

OR

tpcc_load [server] [DB] [user] [pass] [warehouse] [part] [min_wh] [max_wh]
```

选项 warehouse 意为指定测试库下的仓库数量。

真实测试场景中，仓库数一般不建议少于100个，视服务器硬件配置而定，如果是配备了SSD或者PCIE SSD这种高IOPS设备的话，建议最少不低于1000个。

执行下面的命令，开始灌入测试数据：

```bash
cd /tmp/tpcc-mysql
./tpcc_load localhost tpcc tpcc_user "tpcc_password" 1000
```

在这里，需要注意的是 tpcc 默认会读取 /var/lib/mysql/mysql.sock 这个socket 文件。
因此，如果你的 socket 文件不在相应路径的话，可以做个软连接，或者通过TCP/IP的方式连接测试服务器，例如：

```bash
cd /tmp/tpcc-mysql
./tpcc_load 1.2.3.4:3306 tpcc1000 tpcc_user "tpcc_password" 1000
```

加载测试数据时长视仓库数量而定，若过程比较久需要稍加耐心等待。

tpcc_load其实是可以并行加载的，一方面是可以区分 ITEMS、WAREHOUSE、CUSTOMER、ORDERS 四个维度的数据并行加载。
另外，比如最终想加载1000个 warehouse的话，也可以分开成1000个并发并行加载的。看下 tpcc_load 工具的参数就知道了：

```bash
usage: tpcc_load [server] [DB] [user] [pass] [warehouse]
OR
tpcc_load [server] [DB] [user] [pass] [warehouse] [part] [min_wh] [max_wh]
```

* [part]: 1=ITEMS 2=WAREHOUSE 3=CUSTOMER 4=ORDERS
本来想自己写个并行加载脚本的，后来发现万能的github上已经有人做好了，我就直接拿来用了，这是项目链接 [tpcc_load_parallel.sh](https://gist.github.com/sh2/3458844)，加载效率至少提升10倍以上。

二、进行TPCC测试
tpcc_start 工具用于tpcc压测，其用法如下：

tpcc_start -h server_host -P port -d database_name -u mysql_user \
 -p mysql_password -w warehouses -c connections -r warmup_time \
 -l running_time -i report_interval -f report_file


几个选项稍微解释下：
-w 指定仓库数量
-c 指定并发连接数
-r 指定开始测试前进行warmup的时间，进行预热后，测试效果更好
-l 指定测试持续时间
-i  指定生成报告间隔时长
-f 指定生成的报告文件名

现在我们来开启一个测试案例：

```bash
tpcc_start -hlocalhost -d tpcc1000 -u tpcc_user -p "tpcc_password" \
 -w 1000 -c 32 -r 120 -l 3600 \
 -f tpcc_mysql_20140921.log >> tpcc_caseX_20140921.log 2>&1
```

即：模拟 1000个仓库规模，并发 16个线程进行测试，热身时间为 60秒, 压测时间为 1小时。

真实测试场景中，建议预热时间不小于5分钟，持续压测时长不小于30分钟，否则测试数据可能不具参考意义。


###三、TPCC测试结果解读：

发起测试

```bash
./tpcc_start -h 1.2.3.4 -P 3306 -d tpcc10 -u tpcc -p tpcc \
 -w 10 -c 64 -r 30 -l 120 \
 -f tpcclog_201409211538_64_THREADS.log >> tpcc_noaid_2_20140921_64.log 2>&1
```

测试结果输出如下：

```bash
-- 本轮tpcc压测的一些基本信息
***************************************
*** ###easy### TPC-C Load Generator ***
***************************************
option h with value '1.2.3.4'   -- 主机
option P with value '3306'             -- 端口
option d with value 'tpcc10'         -- 数据库
option u with value 'tpcc'             -- 账号
option p with value 'tpcc'             -- 密码
option w with value '10'                 -- 仓库数
option c with value '64'                 -- 并发线程数
option r with value '30'                 -- 数据预热时长
option l with value '120'               -- 压测时长
option f with value 'tpcclog_20140921_64_THREADS.res'  -- 输出报告日志文件

     [server]: 1.2.3.4
     [port]: 3306
     [DBname]: tpcc10
       [user]: tpcc
       [pass]: tpcc
  [warehouse]: 10
 [connection]: 64
     [rampup]: 30 (sec.)
    [measure]: 120 (sec.)

RAMP-UP TIME.(30 sec.)

-- 预热结束，开始进行压测
MEASURING START.

-- 每10秒钟输出一次压测数据
  10, 8376(0):2.744|3.211, 8374(0):0.523|1.626, 838(0):0.250|0.305, 837(0):3.241|3.518, 839(0):9.086|10.676
  20, 8294(0):2.175|2.327, 8292(0):0.420|0.495, 829(0):0.206|0.243, 827(0):2.489|2.593, 827(0):7.214|7.646
…
 110, 8800(0):2.149|2.458, 8792(0):0.424|0.710, 879(0):0.207|0.244, 878(0):2.461|2.556, 878(0):7.042|7.341
 120, 8819(0):2.147|2.327, 8820(0):0.424|0.568, 882(0):0.208|0.237, 881(0):2.483|2.561, 883(0):7.025|7.405
-- 以逗号分隔，共6列
-- 第一列，第N次10秒
-- 第二列，新订单成功执行压测的次数(推迟执行压测的次数):90%事务的响应时间|本轮测试最大响应时间，新订单事务数也被认为是总有效事务数的指标
-- 第三列，支付业务成功执行次数(推迟执行次数):90%事务的响应时间|本轮测试最大响应时间
-- 第四列，订单状态业务的结果，后面几个的意义同上
-- 第五列，物流发货业务的结果，后面几个的意义同上
-- 第六列，库存仓储业务的结果，后面几个的意义同上

-- 压测结束
STOPPING THREADS................................................................

   -- 第一次结果统计
  [0] sc:100589  lt:0  rt:0  fl:0    -- New-Order，新订单业务成功(success,简写sc)次数，延迟(late,简写lt)次数，重试(retry,简写rt)次数，失败(failure,简写fl)次数
  [1] sc:100552  lt:0  rt:0  fl:0    -- Payment，支付业务统计，其他同上
  [2] sc:10059  lt:0  rt:0  fl:0    -- Order-Status，订单状态业务统计，其他同上
  [3] sc:10057  lt:0  rt:0  fl:0    -- Delivery，发货业务统计，其他同上
  [4] sc:10058  lt:0  rt:0  fl:0    -- Stock-Level，库存业务统计，其他同上
 in 120 sec.

    -- 第二次统计结果，其他同上
  [0] sc:100590  lt:0  rt:0  fl:0 
  [1] sc:100582  lt:0  rt:0  fl:0 
  [2] sc:10059  lt:0  rt:0  fl:0 
  [3] sc:10057  lt:0  rt:0  fl:0 
  [4] sc:10059  lt:0  rt:0  fl:0 

 (all must be [OK])       -- 下面所有业务逻辑结果都必须为 OK 才行
 [transaction percentage]
        Payment: 43.47% (>=43.0%) [OK]      -- 支付成功次数(上述统计结果中 sc + lt)必须大于43.0%，否则结果为NG，而不是OK
   Order-Status: 4.35% (>= 4.0%) [OK]       -- 订单状态，其他同上
       Delivery: 4.35% (>= 4.0%) [OK]       -- 发货，其他同上
    Stock-Level: 4.35% (>= 4.0%) [OK]       -- 库存，其他同上
 [response time (at least 90% passed)]      -- 响应耗时指标必须超过90%通过才行
      New-Order: 100.00%  [OK]              -- 下面几个响应耗时指标全部 100% 通过
        Payment: 100.00%  [OK]
   Order-Status: 100.00%  [OK]
       Delivery: 100.00%  [OK]
    Stock-Level: 100.00%  [OK]


                 50294.500 TpmC                      -- TpmC结果值（每分钟事务数，该值是第一次统计结果中的新订单事务数除以总耗时分钟数，例如本例中是：100589/2 = 50294.500）

```

script目录下的一些脚本主要是一些性能数据采集以及分析的，可以自行摸索下怎么用。

参考文章：
1、http://imysql.com/2014/10/10/tpcc-mysql-full-user-manual.shtml
2、https://github.com/yejr/tpcc-mysql-autoinc-pk