title: 高性能mysql主从架构及MHA高可用负载均衡
date: 2015-11-28 14:35:42
categories: mysql
tags: [mysql MHA]
---

实验系统：CentOS 6.7_x86_64（4.2.3）

实验前提：防火墙和selinux都关闭

实验说明：本实验共有3台主机，拓扑图如下所示

![拓扑图](http://images.cnitblog.com/i/609710/201404/192216579327066.png)

主机说明：
manager: 172.16.2.99
master: 172.16.115.101
备用master: 172.16.115.102
也可以加别的slave。

实验软件：

mha4mysql-manager-0.56-0.el6.noarch.rpm

mha4mysql-node-0.56-0.el6.noarch.rpm

mysql(mysql-5.5.46-1.el6.remi.x86_64)

###1、安装软件
三台机器都需要安装mha4mysql-node, master和slave还需要安装mysql, manager可以不装mysql. 当然manager也可以用slave机器来做。

```bash
yum install mysql mysql-server epel-release -y

rpm -Uvh mha4mysql-node-0.56-0.el6.noarch.rpm
```

manager还需要安装mha4mysql-manager.

```bash
yum localinstall mha4mysql-manager-0.56-0.el6.noarch.rpm -y
```
用localinstall会自动安装依赖。

###2、mysql主从配置

#### master上编辑/etc/my.cnf，添加配置

```bash
# master config
server-id = 1
log_bin = mysql-bin
log-slave-updates
binlog_do_db = drupal
```

#### slave上编辑/etc/my.cnf,添加配置

```bash
# slave config
server-id = 2
log_bin = mysql-bin
log-slave-updates
binlog_do_db = drupal
```

#### 创建具有复制权限的用户
```bash
GRANT ALL PRIVILEGES ON *.* TO 'replicate'@'%' IDENTIFIED BY '123456';

FLUSH PRIVILEGES;
```

#### 设置slave

```bash
mysql -uroot -p123456

mysql> change master to master_host='172.16.115.101',master_port=3306,master_user='repl',master_password='123456',master_log_file='mysql-bin.000001',MASTER_LOG_POS=0;

mysql> start slave;
```
#### 查看master/slave配置
```bash
mysql> show master status;

+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      867 | drupal       |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

```bash
mysql> show slave status\G;

*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.16.115.101
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 867
               Relay_Log_File: mysqld-relay-bin.000002
                Relay_Log_Pos: 1013
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: drupal
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 867
              Relay_Log_Space: 1170
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.00 sec)
```

如果Slave_IO_Running和Slave_SQL_Running都是YES，说明slave服务器配置OK。

###3、MHA作用
1）从宕机崩溃的master保存二进制日志事件（binlog events）;

2）识别含有最新更新的slave；

3）应用差异的中继日志（relay log）到其他的slave；

4）应用从master保存的二进制日志事件（binlog events）；

5）提升一个slave为新的master；

6）使其他的slave连接新的master进行复制；

MHA软件由两部分组成，Manager工具包和Node工具包，具体的说明如下。

Manager工具包主要包括以下几个工具：

```bash
masterha_check_ssh              检查MHA的SSH配置状况
masterha_check_repl             检查MySQL复制状况
masterha_manger                 启动MHA
masterha_check_status           检测当前MHA运行状态
masterha_master_monitor         检测master是否宕机
masterha_master_switch          控制故障转移（自动或者手动）
masterha_conf_host              添加或删除配置的server信息
```

Node工具包（这些工具通常由MHA Manager的脚本触发，无需人为操作）主要包括以下几个工具：

```bash
save_binary_logs                保存和复制master的二进制日志
apply_diff_relay_logs           识别差异的中继日志事件并将其差异的事件应用于其他的slave
filter_mysqlbinlog              去除不必要的ROLLBACK事件（MHA已不再使用这个工具）
purge_relay_logs                清除中继日志（不会阻塞SQL线程）
```

###4、配置MHA
####创建MHA的工作目录，并且创建相关配置文件（在软件包解压后的目录里面有样例配置文件）

```bash
[root@172.16.2.99 ~]# vim /etc/masterha/app1.cnf 

[server default]
manager_log=/var/log/mha/app1/manager.log //设置manager的日志
manager_workdir=/var/log/mha/app1.log //设置manager的工作目录
master_binlog_dir=/var/lib/mysql //设置master 保存binlog的位置，以便MHA可以找到master的日志，我这里的也就是mysql的数据目录

user=repl  //设置监控用户root
password=123456 //设置mysql中root用户的密码，这个密码是前文中创建监控用户的那个密码
ssh_user=root //设置ssh的登录用户名
repl_user=repl //设置复制环境中的复制用户名
repl_password=123456 //设置复制用户的密码
ping_interval=10 //设置监控主库，发送ping包的时间间隔，默认是3秒，尝试三次没有回应的时候自动进行railover

shutdown_script="" //设置故障发生后关闭故障主机脚本（该脚本的主要作用是关闭主机放在发生脑裂,这里没有使用）
master_ip_online_change_script="" //设置手动切换时候的切换脚本
report_script="/usr/bin/masterha_report_script" //设置发生切换后发送的报警的脚本
master_ip_failover_script="/usr/bin/masterha_ip_failover" //设置自动failover时候的切换脚本

[server1]
hostname=172.16.115.101

[server2]
hostname=172.16.115.102
candidate_master=1 //设置为候选master，如果设置该参数以后，发生主从切换以后将会将此从库提升为主库，即使这个主库不是集群中时间最新的slave
check_repl_delay=0   //默认情况下如果一个slave落后master 100M的relay logs的话，MHA将不会选择该slave作为一个新的master，因为对于这个slave的恢复需要花费很长时间，通过设置check_repl_delay=0,MHA触发切换在选择一个新的master的时候将会忽略复制延时，这个参数对于设置了candidate_master=1的主机非常有用，因为这个候选主在切换的过程中一定是新的master

```

####设置relay log的清除方式（在每个slave节点上）：

```bash
[root@192.168.115.102 ~]# mysql -e 'set global relay_log_purge=0'
```

注意：

MHA在发生切换的过程中，从库的恢复过程中依赖于relay log的相关信息，所以这里要将relay log的自动清除设置为OFF，采用手动清除relay log的方式。在默认情况下，从服务器上的中继日志会在SQL线程执行完毕后被自动删除。但是在MHA环境中，这些中继日志在恢复其他从服务器时可能会被用到，因此需要禁用中继日志的自动删除功能。定期清除中继日志需要考虑到复制延时的问题。在ext3的文件系统下，删除大的文件需要一定的时间，会导致严重的复制延时。为了避免复制延时，需要暂时为中继日志创建硬链接，因为在linux系统中通过硬链接删除大文件速度会很快。（在mysql数据库中，删除大表时，通常也采用建立硬链接的方式）

MHA节点中包含了pure_relay_logs命令工具，它可以为中继日志创建硬链接，执行SET GLOBAL relay_log_purge=1,等待几秒钟以便SQL线程切换到新的中继日志，再执行SET GLOBAL relay_log_purge=0。

pure_relay_logs脚本参数如下所示：

```bash
--user mysql                      用户名
--password mysql                  密码
--port                            端口号
--workdir                         指定创建relay log的硬链接的位置，默认是/var/tmp，由于系统不同分区创建硬链接文件会失败，故需要执行硬链接具体位置，成功执行脚本后，硬链接的中继日志文件被删除
--disable_relay_log_purge         默认情况下，如果relay_log_purge=1，脚本会什么都不清理，自动退出，通过设定这个参数，当relay_log_purge=1的情况下会将relay_log_purge设置为0。清理relay log之后，最后将参数设置为OFF。
```

####设置定期清理relay脚本（两台slave服务器）

```bash
[root@192.168.115.102 ~]# cat purge_relay_log.sh 
#!/bin/bash
user=root
passwd=123456
port=3306
log_dir='/var/log/masterha'
work_dir='/var/lib/mysql'
purge='/usr/local/bin/purge_relay_logs'

if [ ! -d $log_dir ]
then
   mkdir $log_dir -p
fi

$purge --user=$user --password=$passwd --disable_relay_log_purge --port=$port --workdir=$work_dir >> $log_dir/purge_relay_logs.log 2>&1

```
添加到crontab定期执行
```bash
[root@192.168.115.102 ~]# crontab -l
0 4 * * * /bin/bash /root/purge_relay_log.sh
```

####purge_relay_logs脚本删除中继日志不会阻塞SQL线程。下面我们手动执行看看什么情况。

```bash
[root@192.168.115.102 ~]# purge_relay_logs --user=root --password=123456 --port=3306 -disable_relay_log_purge --workdir=/data/
2014-04-20 15:47:24: purge_relay_logs script started.
 Found relay_log.info: /data/mysql/relay-log.info
 Removing hard linked relay log files server03-relay-bin* under /data/.. done.
 Current relay log file: /data/mysql/server03-relay-bin.000002
 Archiving unused relay log files (up to /data/mysql/server03-relay-bin.000001) ...
 Creating hard link for /data/mysql/server03-relay-bin.000001 under /data//server03-relay-bin.000001 .. ok.
 Creating hard links for unused relay log files completed.
 Executing SET GLOBAL relay_log_purge=1; FLUSH LOGS; sleeping a few seconds so that SQL thread can delete older relay log files (if it keeps up); SET GLOBAL relay_log_purge=0; .. ok.
 Removing hard linked relay log files server03-relay-bin* under /data/.. done.
2014-04-20 15:47:27: All relay log purging operations succeeded.
```

###5、SSH配置
要想MHA顺利执行各项任务，还得打通每台机器的SSH。

```bash
[root@172.16.2.99 app1]# ssh-keygen -t rsa
[root@172.16.2.99 app1]# ssh-copy-id -i .ssh/id_rsa.pub root@172.16.115.101
.....
```
记住，每台机器都要跟别的机器打通SSH。

检查MHA Manger到所有MHA Node的SSH连接状态：

```bash
[root@172.16.2.99 app1]# masterha_check_ssh --conf=/etc/masterha/app1.cnf  
Wed Dec  2 14:42:39 2015 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Wed Dec  2 14:42:39 2015 - [info] Reading application default configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:42:39 2015 - [info] Reading server configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:42:39 2015 - [info] Starting SSH connection tests..
Wed Dec  2 14:42:39 2015 - [debug] 
Wed Dec  2 14:42:39 2015 - [debug]  Connecting via SSH from root@172.16.115.101(172.16.115.101:22) to root@172.16.115.102(172.16.115.102:22)..
Wed Dec  2 14:42:39 2015 - [debug]   ok.
Wed Dec  2 14:42:40 2015 - [debug] 
Wed Dec  2 14:42:39 2015 - [debug]  Connecting via SSH from root@172.16.115.102(172.16.115.102:22) to root@172.16.115.101(172.16.115.101:22)..
Wed Dec  2 14:42:39 2015 - [debug]   ok.
Wed Dec  2 14:42:40 2015 - [info] All SSH connection tests passed successfully.
```
可以看见各个节点ssh验证都是ok的。

###6、检查整个复制环境状况
通过masterha_check_repl脚本查看整个集群的状态

```bash
[root@172.16.2.99 app1]# masterha_check_ssh --conf=/etc/masterha/app1.cnf  
Wed Dec  2 14:42:39 2015 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Wed Dec  2 14:42:39 2015 - [info] Reading application default configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:42:39 2015 - [info] Reading server configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:42:39 2015 - [info] Starting SSH connection tests..
Wed Dec  2 14:42:39 2015 - [debug] 
Wed Dec  2 14:42:39 2015 - [debug]  Connecting via SSH from root@172.16.115.101(172.16.115.101:22) to root@172.16.115.102(172.16.115.102:22)..
Wed Dec  2 14:42:39 2015 - [debug]   ok.
Wed Dec  2 14:42:40 2015 - [debug] 
Wed Dec  2 14:42:39 2015 - [debug]  Connecting via SSH from root@172.16.115.102(172.16.115.102:22) to root@172.16.115.101(172.16.115.101:22)..
Wed Dec  2 14:42:39 2015 - [debug]   ok.
Wed Dec  2 14:42:40 2015 - [info] All SSH connection tests passed successfully.
[root@utn-cz-1-1-s16h2 app1.log]# masterha_check_repl --conf=/etc/masterha/app1.cnf  
Wed Dec  2 14:43:49 2015 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Wed Dec  2 14:43:49 2015 - [info] Reading application default configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:43:49 2015 - [info] Reading server configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:43:49 2015 - [info] MHA::MasterMonitor version 0.56.
Wed Dec  2 14:43:50 2015 - [info] GTID failover mode = 0
Wed Dec  2 14:43:50 2015 - [info] Dead Servers:
Wed Dec  2 14:43:50 2015 - [info] Alive Servers:
Wed Dec  2 14:43:50 2015 - [info]   172.16.115.101(172.16.115.101:3306)
Wed Dec  2 14:43:50 2015 - [info]   172.16.115.102(172.16.115.102:3306)
Wed Dec  2 14:43:50 2015 - [info] Alive Slaves:
Wed Dec  2 14:43:50 2015 - [info]   172.16.115.102(172.16.115.102:3306)  Version=5.5.46-log (oldest major version between slaves) log-bin:enabled
Wed Dec  2 14:43:50 2015 - [info]     Replicating from 172.16.115.101(172.16.115.101:3306)
Wed Dec  2 14:43:50 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Dec  2 14:43:50 2015 - [info] Current Alive Master: 172.16.115.101(172.16.115.101:3306)
Wed Dec  2 14:43:50 2015 - [info] Checking slave configurations..
Wed Dec  2 14:43:50 2015 - [info]  read_only=1 is not set on slave 172.16.115.102(172.16.115.102:3306).
Wed Dec  2 14:43:50 2015 - [warning]  relay_log_purge=0 is not set on slave 172.16.115.102(172.16.115.102:3306).
Wed Dec  2 14:43:50 2015 - [info] Checking replication filtering settings..
Wed Dec  2 14:43:50 2015 - [info]  binlog_do_db= drupal, binlog_ignore_db= 
Wed Dec  2 14:43:50 2015 - [info]  Replication filtering check ok.
Wed Dec  2 14:43:50 2015 - [info] GTID (with auto-pos) is not supported
Wed Dec  2 14:43:50 2015 - [info] Starting SSH connection tests..
Wed Dec  2 14:43:51 2015 - [info] All SSH connection tests passed successfully.
Wed Dec  2 14:43:51 2015 - [info] Checking MHA Node version..
Wed Dec  2 14:43:51 2015 - [info]  Version check ok.
Wed Dec  2 14:43:51 2015 - [info] Checking SSH publickey authentication settings on the current master..
Wed Dec  2 14:43:51 2015 - [info] HealthCheck: SSH to 172.16.115.101 is reachable.
Wed Dec  2 14:43:51 2015 - [info] Master MHA Node version is 0.56.
Wed Dec  2 14:43:51 2015 - [info] Checking recovery script configurations on 172.16.115.101(172.16.115.101:3306)..
Wed Dec  2 14:43:51 2015 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/var/tmp/save_binary_logs_test --manager_version=0.56 --start_file=mysql-bin.000021 
Wed Dec  2 14:43:51 2015 - [info]   Connecting to root@172.16.115.101(172.16.115.101:22).. 
  Creating /var/tmp if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin.000021
Wed Dec  2 14:43:52 2015 - [info] Binlog setting check done.
Wed Dec  2 14:43:52 2015 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Wed Dec  2 14:43:52 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='repl' --slave_host=172.16.115.102 --slave_ip=172.16.115.102 --slave_port=3306 --workdir=/var/tmp --target_version=5.5.46-log --manager_version=0.56 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Wed Dec  2 14:43:52 2015 - [info]   Connecting to root@172.16.115.102(172.16.115.102:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to mysqld-relay-bin.000022
    Temporary relay log file is /var/lib/mysql/mysqld-relay-bin.000022
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Wed Dec  2 14:43:52 2015 - [info] Slaves settings check done.
Wed Dec  2 14:43:52 2015 - [info] 
172.16.115.101(172.16.115.101:3306) (current master)
 +--172.16.115.102(172.16.115.102:3306)

Wed Dec  2 14:43:52 2015 - [info] Checking replication health on 172.16.115.102..
Wed Dec  2 14:43:52 2015 - [info]  ok.
Wed Dec  2 14:43:52 2015 - [info] Checking master_ip_failover_script status:
Wed Dec  2 14:43:52 2015 - [info]   /usr/bin/masterha_ip_failover --command=status --ssh_user=root --orig_master_host=172.16.115.101 --orig_master_ip=172.16.115.101 --orig_master_port=3306 


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 172.16.115.200/24===

Checking the Status of the script.. OK 
Wed Dec  2 14:43:52 2015 - [info]  OK.
Wed Dec  2 14:43:52 2015 - [warning] shutdown_script is not defined.
Wed Dec  2 14:43:52 2015 - [info] Got exit code 0 (Not master dead).

MySQL Replication Health is OK.
```

###7、检查MHA Manager的状态
通过master_check_status脚本查看Manager的状态：

```bash
[root@172.16.2.99 ~]# masterha_check_status --conf=/etc/masterha/app1.cnf
app1 is stopped(2:NOT_RUNNING).
```
注意：如果正常，会显示"PING_OK"，否则会显示"NOT_RUNNING"，这代表MHA监控没有开启。


###8、开启MHA Manager监控

```bash
[root@172.16.2.99 ~]# nohup masterha_manager --conf=/etc/masterha/app1.cnf --ignore_last_failover > /var/log/mha/app1/manager.log < /dev/null 2>&1 &
[1] 7303
```

启动参数介绍：

```bash
--remove_dead_master_conf      该参数代表当发生主从切换后，老的主库的ip将会从配置文件中移除。
--manger_log                            日志存放位置
--ignore_last_failover                 在缺省情况下，如果MHA检测到连续发生宕机，且两次宕机间隔不足8小时的话，则不会进行Failover，之所以这样限制是为了避免ping-pong效应。该参数代表忽略上次MHA触发切换产生的文件，默认情况下，MHA发生切换后会在日志目录，也就是上面我设置的/data产生app1.failover.complete文件，下次再次切换的时候如果发现该目录下存在该文件将不允许触发切换，除非在第一次切换后收到删除该文件，为了方便，这里设置为--ignore_last_failover。
```

查看MHA Manager监控是否正常：

```bash
[root@172.16.2.99 ~]# masterha_check_status --conf=/etc/masterha/app1.cnf
app1 (pid:20386) is running(0:PING_OK), master:172.16.115.101
```

可以看见已经在监控了，而且master的主机为172.16.115.101


###9、查看启动日志

```bash
[root@172.16.2.99 ~]#  cat /var/log/mha/app1/manager.log
Wed Dec  2 14:46:37 2015 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Wed Dec  2 14:46:37 2015 - [info] Reading application default configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:46:37 2015 - [info] Reading server configuration from /etc/masterha/app1.cnf..
Wed Dec  2 14:46:37 2015 - [info] MHA::MasterMonitor version 0.56.
Wed Dec  2 14:46:38 2015 - [info] GTID failover mode = 0
Wed Dec  2 14:46:38 2015 - [info] Dead Servers:
Wed Dec  2 14:46:38 2015 - [info] Alive Servers:
Wed Dec  2 14:46:38 2015 - [info]   172.16.115.101(172.16.115.101:3306)
Wed Dec  2 14:46:38 2015 - [info]   172.16.115.102(172.16.115.102:3306)
Wed Dec  2 14:46:38 2015 - [info] Alive Slaves:
Wed Dec  2 14:46:38 2015 - [info]   172.16.115.102(172.16.115.102:3306)  Version=5.5.46-log (oldest major version between slaves) log-bin:enabled
Wed Dec  2 14:46:38 2015 - [info]     Replicating from 172.16.115.101(172.16.115.101:3306)
Wed Dec  2 14:46:38 2015 - [info]     Primary candidate for the new Master (candidate_master is set)
Wed Dec  2 14:46:38 2015 - [info] Current Alive Master: 172.16.115.101(172.16.115.101:3306)
Wed Dec  2 14:46:38 2015 - [info] Checking slave configurations..
Wed Dec  2 14:46:38 2015 - [info]  read_only=1 is not set on slave 172.16.115.102(172.16.115.102:3306).
Wed Dec  2 14:46:38 2015 - [warning]  relay_log_purge=0 is not set on slave 172.16.115.102(172.16.115.102:3306).
Wed Dec  2 14:46:38 2015 - [info] Checking replication filtering settings..
Wed Dec  2 14:46:38 2015 - [info]  binlog_do_db= drupal, binlog_ignore_db= 
Wed Dec  2 14:46:38 2015 - [info]  Replication filtering check ok.
Wed Dec  2 14:46:38 2015 - [info] GTID (with auto-pos) is not supported
Wed Dec  2 14:46:38 2015 - [info] Starting SSH connection tests..
Wed Dec  2 14:46:39 2015 - [info] All SSH connection tests passed successfully.
Wed Dec  2 14:46:39 2015 - [info] Checking MHA Node version..
Wed Dec  2 14:46:39 2015 - [info]  Version check ok.
Wed Dec  2 14:46:39 2015 - [info] Checking SSH publickey authentication settings on the current master..
Wed Dec  2 14:46:39 2015 - [info] HealthCheck: SSH to 172.16.115.101 is reachable.
Wed Dec  2 14:46:39 2015 - [info] Master MHA Node version is 0.56.
Wed Dec  2 14:46:39 2015 - [info] Checking recovery script configurations on 172.16.115.101(172.16.115.101:3306)..
Wed Dec  2 14:46:39 2015 - [info]   Executing command: save_binary_logs --command=test --start_pos=4 --binlog_dir=/var/lib/mysql --output_file=/var/tmp/save_binary_logs_test --manager_version=0.56 --start_file=mysql-bin.000021 
Wed Dec  2 14:46:39 2015 - [info]   Connecting to root@172.16.115.101(172.16.115.101:22).. 
  Creating /var/tmp if not exists..    ok.
  Checking output directory is accessible or not..
   ok.
  Binlog found at /var/lib/mysql, up to mysql-bin.000021
Wed Dec  2 14:46:39 2015 - [info] Binlog setting check done.
Wed Dec  2 14:46:39 2015 - [info] Checking SSH publickey authentication and checking recovery script configurations on all alive slave servers..
Wed Dec  2 14:46:39 2015 - [info]   Executing command : apply_diff_relay_logs --command=test --slave_user='repl' --slave_host=172.16.115.102 --slave_ip=172.16.115.102 --slave_port=3306 --workdir=/var/tmp --target_version=5.5.46-log --manager_version=0.56 --relay_log_info=/var/lib/mysql/relay-log.info  --relay_dir=/var/lib/mysql/  --slave_pass=xxx
Wed Dec  2 14:46:39 2015 - [info]   Connecting to root@172.16.115.102(172.16.115.102:22).. 
  Checking slave recovery environment settings..
    Opening /var/lib/mysql/relay-log.info ... ok.
    Relay log found at /var/lib/mysql, up to mysqld-relay-bin.000022
    Temporary relay log file is /var/lib/mysql/mysqld-relay-bin.000022
    Testing mysql connection and privileges.. done.
    Testing mysqlbinlog output.. done.
    Cleaning up test file(s).. done.
Wed Dec  2 14:46:40 2015 - [info] Slaves settings check done.
Wed Dec  2 14:46:40 2015 - [info] 
172.16.115.101(172.16.115.101:3306) (current master)
 +--172.16.115.102(172.16.115.102:3306)

Wed Dec  2 14:46:40 2015 - [info] Checking master_ip_failover_script status:
Wed Dec  2 14:46:40 2015 - [info]   /usr/bin/masterha_ip_failover --command=status --ssh_user=root --orig_master_host=172.16.115.101 --orig_master_ip=172.16.115.101 --orig_master_port=3306 


IN SCRIPT TEST====/sbin/ifconfig eth0:0 down==/sbin/ifconfig eth0:0 172.16.115.200/24===

Checking the Status of the script.. OK 
Wed Dec  2 14:46:40 2015 - [info]  OK.
Wed Dec  2 14:46:40 2015 - [warning] shutdown_script is not defined.
Wed Dec  2 14:46:40 2015 - [info] Set master ping interval 1 seconds.
Wed Dec  2 14:46:40 2015 - [warning] secondary_check_script is not defined. It is highly recommended setting it to check master reachability from two or more routes.
Wed Dec  2 14:46:40 2015 - [info] Starting ping health check on 172.16.115.101(172.16.115.101:3306)..
Wed Dec  2 14:46:40 2015 - [info] Ping(SELECT) succeeded, waiting until MySQL doesn't respond..
```

###10、配置VIP

我们采用脚本的方式去配置vip

```bash
[root@172.16.2.99 ~]#  vim /usr/bin/masterha_ip_failover

#!/usr/bin/env perl

use strict;
use warnings FATAL => 'all';

use Getopt::Long;

my (
    $command,          $ssh_user,        $orig_master_host, $orig_master_ip,
    $orig_master_port, $new_master_host, $new_master_ip,    $new_master_port
);

my $vip = '172.16.115.200/24';
my $key = '0';
my $ssh_start_vip = "/sbin/ifconfig eth0:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig eth0:$key down";

GetOptions(
    'command=s'          => \$command,
    'ssh_user=s'         => \$ssh_user,
    'orig_master_host=s' => \$orig_master_host,
    'orig_master_ip=s'   => \$orig_master_ip,
    'orig_master_port=i' => \$orig_master_port,
    'new_master_host=s'  => \$new_master_host,
    'new_master_ip=s'    => \$new_master_ip,
    'new_master_port=i'  => \$new_master_port,
);

exit &main();

sub main {

    print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";

    if ( $command eq "stop" || $command eq "stopssh" ) {

        my $exit_code = 1;
        eval {
            print "Disabling the VIP on old master: $orig_master_host \n";
            &stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "start" ) {

        my $exit_code = 10;
        eval {
            print "Enabling the VIP - $vip on the new master - $new_master_host \n";
            &start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        exit 0;
    }
    else {
        &usage();
        exit 1;
    }
}

sub start_vip() {
    `ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
     return 0  unless  ($ssh_user);
    `ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}

sub usage {
    print
    "Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip 
            --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
```

###11、邮件发送脚本send_report 

```bash
#!/usr/bin/perl

#  Copyright (C) 2011 DeNA Co.,Ltd.
#
#  This program is free software; you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 2 of the License, or
#  (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#   along with this program; if not, write to the Free Software
#  Foundation, Inc.,
#  51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA

## Note: This is a sample script and is not complete. Modify the script based on your environment.

use strict;
use warnings FATAL => 'all';
use Mail::Sender;
use Getopt::Long;

#new_master_host and new_slave_hosts are set only when recovering master succeeded
my ( $dead_master_host, $new_master_host, $new_slave_hosts, $subject, $body );
my $smtp='smtp.163.com';
my $mail_from='services@163.com';
my $mail_user='services@163.com';
my $mail_pass='123456';
my $mail_to=['xxx@xxx.cn'];
GetOptions(
  'orig_master_host=s' => \$dead_master_host,
  'new_master_host=s'  => \$new_master_host,
  'new_slave_hosts=s'  => \$new_slave_hosts,
  'subject=s'          => \$subject,
  'body=s'             => \$body,
);

mailToContacts($smtp,$mail_from,$mail_user,$mail_pass,$mail_to,$subject,$body);

sub mailToContacts {
    my ( $smtp, $mail_from, $user, $passwd, $mail_to, $subject, $msg ) = @_;
    open my $DEBUG, "> /tmp/monitormail.log"
        or die "Can't open the debug      file:$!\n";
    my $sender = new Mail::Sender {
        ctype       => 'text/plain; charset=utf-8',
        encoding    => 'utf-8',
        smtp        => $smtp,
        from        => $mail_from,
        auth        => 'LOGIN',
        TLS_allowed => '0',
        authid      => $user,
        authpwd     => $passwd,
        to          => $mail_to,
        subject     => $subject,
        debug       => $DEBUG
    };

    $sender->MailMsg(
        {   msg   => $msg,
            debug => $DEBUG
        }
    ) or print $Mail::Sender::Error;
    return 1;
}



# Do whatever you want here

exit 0;
```


###12、测试  
1.主库断电    
2.主库断网  
3.主库重启  
4.主库关机  
以上情况都测试过都能自动切换~  
请看日志分析切换过程  
  
###13、主库模拟故障后的恢复  

```bash
[root@172.16.2.99 app1]# grep -i "All other slaves should start" manager.log 
Mon Apr 21 22:28:33 2014 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='172.16.115.102', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000022', MASTER_LOG_POS=506716, MASTER_USER='repl', MASTER_PASSWORD='xxx';
[root@192.168.0.20 app1]# 
```
   
然后再就得主库里执行下,再start slave ,然后就得主库就变为了新的主库的从库了。  
记得看下show slave status\G;  

###14、报错记录总结
摘抄自google.

####报错记录1：

```bash
[root@data01 ~]# masterha_check_repl--conf=/etc/masterha/app1.cnf
Tue Apr 7 22:31:06 2015 - [warning] Global configuration file/etc/masterha_default.cnf not found. Skipping.
Tue Apr 7 22:31:07 2015 - [info] Reading application default configuration from/etc/masterha/app1.cnf..
Tue Apr 7 22:31:07 2015 - [info] Reading server configuration from/etc/masterha/app1.cnf..
Tue Apr 7 22:31:07 2015 - [info] MHA::MasterMonitor version 0.56.
Tue Apr 7 22:31:07 2015 - [error][/usr/local/share/perl5/MHA/Server.pm,ln303]  Getting relay log directory orcurrent relay logfile from replication table failed on192.168.52.130(192.168.52.130:3306)!
Tue Apr 7 22:31:07 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln424] Error happened on checking configurations.  at /usr/local/share/perl5/MHA/ServerManager.pmline 315
Tue Apr 7 22:31:07 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln523] Error happened on monitoring servers.
Tue Apr 7 22:31:07 2015 - [info] Got exit code 1 (Not master dead).
 
MySQL Replication Health is NOT OK!
[root@data01 ~]#
```

解决办法：在192.168.52.130上面，vim /etc/my.cnf，在里面添加

```bash
relay-log=/home/data/mysql/binlog/mysql-relay-bin
```

然后重启mysql，再去重新设置slave连接。

```bash
STOP SLAVE;
RESET SLAVE;
CHANGE MASTER TOMASTER_HOST='192.168.52.129',MASTER_USER='repl',MASTER_PASSWORD='repl_1234',MASTER_LOG_FILE='mysql-bin.000178',MASTER_LOG_POS=459;
START SLAVE;
```

Ok，搞定了。
 
####报错记录2：

```bash
[root@data01 perl]# masterha_check_repl--conf=/etc/masterha/app1.cnf
Thu Apr 9 00:54:32 2015 - [warning] Global configuration file/etc/masterha_default.cnf not found. Skipping.
Thu Apr 9 00:54:32 2015 - [info] Reading application default configuration from/etc/masterha/app1.cnf..
Thu Apr 9 00:54:32 2015 - [info] Reading server configuration from/etc/masterha/app1.cnf..
Thu Apr 9 00:54:32 2015 - [info] MHA::MasterMonitor version 0.56.
Thu Apr 9 00:54:32 2015 - [error][/usr/local/share/perl5/MHA/Server.pm,ln306]  Getting relay log directory orcurrent relay logfile from replication table failed on 192.168.52.130(192.168.52.130:3306)!
Thu Apr 9 00:54:32 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln424] Error happened on checking configurations.  at/usr/local/share/perl5/MHA/ServerManager.pm line 315
Thu Apr 9 00:54:32 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln523] Error happened on monitoring servers.
Thu Apr 9 00:54:32 2015 - [info] Got exit code 1 (Not master dead).
 
MySQL Replication Health is NOT OK!
[root@data01 perl]#
```

解决方法：
/etc/masterha/app1.cnf文件里面的参数配置，user和repl_user都是mysql账号，需要创建好，这里是只创建了repl_user而没有创建好user账号：

```bash
user=manager
password=manager_1234
repl_user=repl
repl_password=repl_1234
```

在mysql节点上，建立允许manager 访问数据库的“ manager manager ”账户，主要用于SHOW SLAVESTATUS,RESET SLAVE; 所以需要执行如下命令：

```bash
GRANT SUPER,RELOAD,REPLICATIONCLIENT,SELECT ON *.* TO manager@'192.168.52.%' IDENTIFIED BY 'manager_1234';
```

####错误记录3：

```bash
[root@oraclem1 ~]#  masterha_check_repl--conf=/etc/masterha/app1.cnf
Thu Apr 9 23:09:05 2015 - [warning] Global configuration file/etc/masterha_default.cnf not found. Skipping.
Thu Apr 9 23:09:05 2015 - [info] Reading application default configuration from/etc/masterha/app1.cnf..
Thu Apr 9 23:09:05 2015 - [info] Reading server configuration from/etc/masterha/app1.cnf..
Thu Apr 9 23:09:05 2015 - [info] MHA::MasterMonitor version 0.56.
Thu Apr 9 23:09:05 2015 - [error][/usr/local/share/perl5/MHA/ServerManager.pm,ln781] Multi-master configuration is detected, but two or more masters areeither writable (read-only is not set) or dead! Check configurations fordetails. Master configurations are as below:
Master 192.168.52.130(192.168.52.130:3306),replicating from 192.168.52.129(192.168.52.129:3306)
Master 192.168.52.129(192.168.52.129:3306),replicating from 192.168.52.130(192.168.52.130:3306)
 
Thu Apr 9 23:09:05 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln424] Error happened on checking configurations.  at/usr/local/share/perl5/MHA/MasterMonitor.pm line 326
Thu Apr 9 23:09:05 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln523] Error happened on monitoring servers.
Thu Apr 9 23:09:05 2015 - [info] Got exit code 1 (Not master dead).
 
MySQL Replication Health is NOT OK!
[root@oraclem1 ~]#
```

解决办法：

```bash
mysql> set global read_only=1;
Query OK, 0 rows affected (0.00 sec)

mysql>
```
 
####报错记录4：

```bash
Thu Apr 9 23:54:32 2015 - [info] Checking SSH publickey authentication andchecking recovery script configurations on all alive slave servers..
Thu Apr 9 23:54:32 2015 - [info]  Executing command : apply_diff_relay_logs --command=test--slave_user='manager' --slave_host=192.168.52.130 --slave_ip=192.168.52.130--slave_port=3306 --workdir=/var/tmp --target_version=5.6.12-log--manager_version=0.56 --relay_dir=/home/data/mysql/data--current_relay_log=mysqld-relay-bin.000011 --slave_pass=xxx
Thu Apr 9 23:54:32 2015 - [info]  Connecting to root@192.168.52.130(192.168.52.130:22)..
Can't exec "mysqlbinlog": No suchfile or directory at /usr/local/share/perl5/MHA/BinlogManager.pm line 106.
mysqlbinlog version command failed with rc1:0, please verify PATH, LD_LIBRARY_PATH, and client options
 at/usr/local/bin/apply_diff_relay_logs line 493
Thu Apr 9 23:54:32 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln205] Slaves settings check failed!
Thu Apr 9 23:54:32 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln413] Slave configuration failed.
Thu Apr 9 23:54:32 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln424] Error happened on checking configurations.  at /usr/local/bin/masterha_check_repl line 48
Thu Apr 9 23:54:32 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln523] Error happened on monitoring servers.
Thu Apr 9 23:54:32 2015 - [info] Got exit code 1 (Not master dead).
 
MySQL Replication Health is NOT OK!
[root@oraclem1 ~]#
```

解决办法：

```bash
[root@data02 ~]# type mysqlbinlog
mysqlbinlog is/usr/local/mysql/bin/mysqlbinlog
[root@data02 ~]#
[root@data02 ~]# ln -s/usr/local/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
```

####报错记录5：

```bash
Thu Apr 9 23:57:24 2015 - [info]  Connecting to root@192.168.52.130(192.168.52.130:22)..
 Checking slave recovery environment settings..
   Relay log found at /home/data/mysql/data, up to mysqld-relay-bin.000013
   Temporary relay log file is /home/data/mysql/data/mysqld-relay-bin.000013
   Testing mysql connection and privileges..sh: mysql: command not found
mysql command failed with rc 127:0!
 at/usr/local/bin/apply_diff_relay_logs line 375
         main::check()called at /usr/local/bin/apply_diff_relay_logs line 497
         eval{...} called at /usr/local/bin/apply_diff_relay_logs line 475
         main::main()called at /usr/local/bin/apply_diff_relay_logs line 120
Thu Apr 9 23:57:24 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln205] Slaves settings check failed!
Thu Apr 9 23:57:24 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln413] Slave configuration failed.
Thu Apr 9 23:57:24 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln424] Error happened on checking configurations.  at /usr/local/bin/masterha_check_repl line 48
Thu Apr 9 23:57:24 2015 - [error][/usr/local/share/perl5/MHA/MasterMonitor.pm,ln523] Error happened on monitoring servers.
Thu Apr 9 23:57:24 2015 - [info] Got exit code 1 (Not master dead).
 
MySQL Replication Health is NOT OK!
```

解决办法：

```bash
ln -s /usr/local/mysql/bin/mysql/usr/bin/mysql
```
 
####报错记录6：

```bash
Fri Apr 10 00:58:36 2015 - [info]   Executing command : apply_diff_relay_logs--command=test --slave_user='manager' --slave_host=192.168.52.130--slave_ip=192.168.52.130 --slave_port=3306 --workdir=/var/tmp--target_version=5.6.12-log --manager_version=0.56--relay_dir=/home/data/mysql/data--current_relay_log=mysqld-relay-bin.000011 --slave_pass=xxx
Fri Apr 10 00:58:36 2015 - [info]   Connecting to root@192.168.52.130(192.168.52.130:22)..
 Checking slave recovery environment settings..
   Relay log found at /home/data/mysql/data, up to mysqld-relay-bin.000013
   Temporary relay log file is/home/data/mysql/data/mysqld-relay-bin.000013
   Testing mysql connection and privileges..Warning: Using a password onthe command line interface can be insecure.
ERROR 1142 (42000) at line 1: CREATEcommand denied to user 'manager'@'192.168.52.130' for table'apply_diff_relay_logs_test'
mysql command failed with rc 1:0!
 at/usr/local/bin/apply_diff_relay_logs line 375
         main::check()called at /usr/local/bin/apply_diff_relay_logs line 497
         eval{...} called at /usr/local/bin/apply_diff_relay_logs line 475
         main::main()called at /usr/local/bin/apply_diff_relay_logs line 120
Fri Apr 10 00:58:37 2015 -[error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln205] Slaves settingscheck failed!
Fri Apr 10 00:58:37 2015 -[error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln413] Slave configurationfailed.
Fri Apr 10 00:58:37 2015 -[error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln424] Error happened onchecking configurations.  at/usr/local/bin/masterha_check_repl line 48
Fri Apr 10 00:58:37 2015 -[error][/usr/local/share/perl5/MHA/MasterMonitor.pm, ln523] Error happened onmonitoring servers.
Fri Apr 10 00:58:37 2015 - [info] Got exitcode 1 (Not master dead).
 
MySQL Replication Health is NOT OK!
```

解决办法：
执行如下授权语句sql：

```bash
GRANT CREATE,INSERT,UPDATE,DELETE,DROP ON*.* TO manager@'192.168.52.%';
```


####其他
另外，如果nohup masterha_manager执行不了，请检查master和slave是否配置正确。可以通过

```bash
change master to master_host='172.16.115.101',master_port=3306,master_user='repl',master_password='123456',master_log_file='mysql-bin.000001',MASTER_LOG_POS=0;
```

重新设置slave。