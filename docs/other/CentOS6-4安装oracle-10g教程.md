title: CentOS6.4安装oracle 10g教程
date: 2015-11-17 13:55:47
categories: linux
tags: [oracle]
---

##一、硬件要求
1、内存 & swap
Minimum: 1 GB of RAM
Recommended: 2 GB of RAM or more
![硬件要求](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118211127_56147.png)

检查内存情况
```bash
grep MemTotal /proc/meminfo
grep SwapTotal /proc/meminfo
```

2、硬盘
由于CentOS安装后差不多有4~5G，再加上Oracle等等的安装，所以请准备至少10G的硬盘空间。
检查磁盘情况
```bash
df -h
```
![df-h](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118211509_28032.jpg)

##二、软件
系统平台：CentOS 6.3(x86_64)
CentOS-6.3-x86_64-bin-DVD1.iso
Oracle版本：Oracle 10g R2
10201_database_linux_x86_64.cpio

##三、系统安装注意
系统安装时一定要安装桌面模式，否则无法安装oracle，另外请勿开启SELinux，oracle官方不建议使用SELinux，CentOS 的防火墙也请暂时关闭，减少安装时的困扰。为防止Oracle安装过程中出现乱码，建议使用英文作为系统语言，进行Oracle的安装工作。
本文中所描述的系统命令，未经特殊标示，均为“#”代表root权限，“$”代表oracle权限。

##四、安装Oracle前的系统准备工作
首先，请先以root账号登入作一些前置设定作业。
1、关闭防火墙、禁用SELinux
setup
![setup](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118211724_11378.jpg)

vi /etc/selinux/config
修改SELINUX=disabled，然后重启。
如果不想重启系统，使用命令setenforce 0
![setenforce](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118211852_74451.jpg)

2、安装依赖包
Oracle官方文档要求的安装包：
![oracle安装依赖包](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118212103_24755.jpg)

查看Oracle相关包是否已经安装：
![oracle相关包](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118212212_30050.jpg)

用yum方式安装所需的包：
```bash
yum -y install binutils compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf elfutils-libelf-devel gcc gcc-c++ glibc glibc.i686 glibc-common glibc-devel glibc-devel.i686 glibc-headers ksh libaio libaio.i686 libaio-devel libaio-devel.i686 libgcc libgcc.i686 libstdc++ libstdc++.i686 libstdc++-devel make sysstat
```
![yum](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118212321_10098.jpg)

最后还需要安装libXp这个Library，这个一定要安装，否则安装Oracle时会出现java Exception。
```bash
yum install libXp
```
![libXp](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118212436_33952.jpg)

3、创建Oracle用户与组
在这里只讨论单主机环境，不考虑RAC环境的配置。
执行以下指令以新增oracle安装时所需要的使用者与群组。
(1) 建立群组oinstall 
```bash
groupadd oinstall
```
(2) 建立群组dba
```bash
groupadd dba
```
(3) 新增使用者oracle并将其加入oinstall和dba群组
```bash
useradd -m -g oinstall -G dba oracle
```
(4) 测试oracle账号是否建立完成
```bash
id oracle
```
(5) 建立oracle的新密码
```bash
passwd oracle
```
![passwd](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118212623_66655.jpg)

(6) 建立oracle安装目录
```bash
mkdir -p /home/app/oracle
chown -R oracle:oinstall /home/app/oracle
chmod -R 775 /home/app/oracle
```
4、将oracle使用者加入到sudo群组中
vi /etc/sudoers
找到
```bash
root        ALL=(ALL)        ALL 
```
这行，并且在底下再加入
```bash
oracle        ALL=(ALL)        ALL
```
输入wq!（由于这是一份只读文档所以需要再加上!）并且按下Enter
![sudo](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213155_30373.jpg)

5、配置系统内核参数
vi /etc/sysctl.conf
修改和添加以下内容：
```bash
kernel.shmall = 4294967296                           //表示系统一次可以使用的共享内存总量（以页为单位）。缺省值就是2097152，通常不需要修改
kernel.shmmax = 68719476736                      //定义了共享内存段的最大尺寸（以字节为单位）。缺省为32M，对于oracle来说，该缺省值太低了，通常将其设置为2G
kernel.shmmni = 4096                                    //用于设置系统范围内共享内存段的最大数量。该参数的默认值是 4096 。通常不需要更改
kernel.sem = 250 32000 100 128                    //表示设置的信号量
net.ipv4.ip_local_port_range = 1024 65000
net.core.rmem_default=4194304                     //默认的接收窗口大小
net.core.rmem_max=4194304                        //接收窗口的最大大小
net.core.wmem_default=262144                      //默认的发送窗口大小
net.core.wmem_max=262144                         //发送窗口的最大大小
```
会有一些与目前的参数重复的，就修改成文件上提供的。
![sysctl](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213249_97968.jpg)

编辑完之后，储存，然后执行：
```bash
sysctl –p
```
启用刚刚所做的变更。
![sysctl-p](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213427_99326.jpg)

6、编辑/etc/security/limits.conf
vi /etc/security/limits.conf
添加以下四行
```bash
oracle  soft        nproc   2047
oracle  hard        nproc   16384
oracle  soft        nofile  1024
oracle  hard        nofile  65536
```
![limits](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213539_84050.jpg)

7、编辑/etc/pam.d/login
vi /etc/pam.d/login
添加以下两行
```bash
session required /lib64/security/pam_limits.so
session required pam_limits.so
```
![login](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213641_35440.jpg)

8、修改/etc/profile
vi /etc/profile
将以下代码新增到profile档案中。
```bash
if [ $USER = "oracle" ]; then
    if [ $SHELL = "/bin/ksh" ]; then
        ulimit -p 16384
        ulimit -n 65536
    else
        ulimit -u 16384 -n 65536
    fi
fi 
```
![profile](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213744_91921.jpg)

9、修改Linux发行版本信息
由于Oracle 10g发行的时候，CentOS 6没有发行，所以Oracle 10g并没有对CentOS 6确认支持，需要修改文件让Oracle 10g支持CentOS 6。
我们需要手工修改Linux的发行注记，让Oracle 10g支持CentOS 6。
编辑/etc/redhat-release文件
vi /etc/redhat-release
将其中的内容CentOS release 6.3 (Final)修改为redhat 4
![redhat-release](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118213909_63963.jpg)

10、创建Oracle安装文件夹以及数据存放文件夹
```bash
mkdir /opt/oracle
mkdir /opt/oracle/102
chown -R oracle:dba /opt/oracle
```
![mkdir](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214010_35072.jpg)

11、配置Linux主机
检查/etc/hosts文件中是否有localhost的记录（指向127.0.0.1即可），若没有的话，在后面配置Oracle监听的时候会出现一些问题，导致无法启动监听，在此手工添加此记录即可。
![配置linux主机](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214213_76672.jpg)
第一阶段到此完毕，接下来，完成这些设定之后，请先注销root账号，并且以oracle账号再次登入系统。

12、配置oracle用户环境变量
```bash
cd /home/oracle
vi .bash_profile
```
修改并加入以下內容
```bash
export ORACLE_BASE=/home/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/10.2.0/db_1
export ORACLE_SID=pltest02
export PATH=.:${PATH}:$HOME/bin:$ORACLE_HOME/bin
export PATH=${PATH}:/usr/bin:/bin:/usr/bin/X11:/usr/local/bin
export PATH=${PATH}:$ORACLE_BASE/common/oracle/bin
export ORACLE_TERM=xterm
export TNS_ADMIN=$ORACLE_HOME/network/admin
export ORA_NLS10=$ORACLE_HOME/nls/data
export LD_LIBRARY_PATH32=$ORACLE_HOME/lib32
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$ORACLE_HOME/oracm/lib
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/lib:/usr/lib:/usr/local/lib
export CLASSPATH=$ORACLE_HOME/jre
export CLASSPATH=${CLASSPATH}:$ORACLE_HOME/jlib
export CLASSPATH=${CLASSPATH}:$ORACLE_HOME/rdbms/jlib
export CLASSPATH=${CLASSPATH}:$ORACLE_HOME/network/jlib
export THREADS_FLAG=native
export TEMP=/tmp
export TMPDIR=/tmp
export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK
```
![export](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214338_80244.jpg)

保存后使用如下命令，使设置生效：
```bash
source /home/oracle/.bash_profile
```

##五、安装Oracle，并进行相关设置
1、解压缩安装文件
将下载的10201_database_linux_x86_64.cpio放至即将安装oracle的文件夹/opt/oracle
回到终端模式并且进入到oracle文件夹：
```bash
cd /opt/oracle
```
解压缩10201_database_linux_x86_64.cpio
```bash
cpio -idmv < 10201_database_linux_x86_64.cpio
```
接着会看到一连串的解压缩动作。
![解压缩](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214635_35567.jpg)

解压缩完成会在同一个文件夹中看到database的文件夹，请进入到database文件夹中：
```bash
cd database
```
准备执行数据库安装，如果你的centos是中文环境，安装时会出现中文乱码，请下以下指令
```bash
export LANG=en_US
```
接着执行
```bash
./runInstaller
```
如果无法看到安装界面，请使用root帐户执行如下命令后再运行安装程序：
```bash
export DISPLAY=:0.0 
xhost + 
./runInstaller
```
![runInstatller](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214735_68730.jpg)

遇到错误：Exception in ...... /lib/i386/libawt.so: libXp.so.6: cannot open shared object file: No such file or directory
![exception-in](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214829_59489.jpg)

解决：
yum -y install libXp.i686
分析：看报错信息"/lib/i386/libawt.so: libXp.so.6: cannot open shared object file: No such file or directory"，libXp需要安装i386的包，而不能安装X64的包。上面认为64位的linux需要安装64位的libXp包，所以导致这个问题。
再次执行 
```bash
$ ./runInstaller
```
遇到错误：Exception in ...... /lib/i386/libawt.so: libXt.so.6: cannot open shared object file: No such file or directory
![exception-in](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118214942_33265.jpg)

解决：
yum -y install libXt.i686
再次执行 
```bash
$ ./runInstaller
```
遇到错误：Exception in ...... /lib/i386/libawt.so: libXtst.so.6: cannot open shared object file: No such file or directory
![exception-in](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215043_90710.jpg)

解决：
yum -y install libXtst.i686
再次执行 
```bash
$ ./runInstaller
```
开始执行安装程序。
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215145_27087.jpg)

由于相关的前置作业已经在之前做好了，在这个步骤只需要将UNIX DBA Group选择为dba以及输入SYS, SYSTEM等账号共享的database Password即可。然后选择Next即可。
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215315_24196.jpg)

同样的，将群组选择为dba群组，按Next
在这个步骤中，请点选Checking Network Configuration requirements为User Verified，接着按下Next
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215418_91532.png)

最后出现Install Summary画面，此时只要按下Install按钮，系统即开始安装。
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215529_80624.jpg)

安装过程...
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215624_41287.png)

安装进度大约到65%时会有错误提示：
Error in invoking target 'collector' of makefile '/opt/oracle/102/sysman/lib/ins_emdb.mk'.
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215719_28070.jpg)

同时oraInventory/logs/目录下的安装日志文件里面会有如下类似错误提示：
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215811_56664.jpg)

这是oracle安装程序的一个bug，可以忽略此错误继续安装，对系统没什么影响。
在Configuration Assistants 时会出现错误提示：
OUI-25031:Some of the configuration assistants failed.
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215907_97381.jpg)

分析：主机名映射错误
解决：修改/etc/hosts文件，增加IP地址与主机名的映射如下：
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118215952_57630.jpg)

接着会遇到错误提示：
ORA-27125:unable to create shared memory segment
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220100_66235.jpg)

解决：
1. 确定安装oracle所使用的用户组
```bash
id oracle
```
可以看到oracle组dba id 为501。
2. 修改内核参数
```bash
echo "501" >/proc/sys/vm/hugetlb_shm_group
```
就可以了。
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220150_52635.jpg)

安装到数据库设置助理，可以在这边选取password management作密码的修改，如不需要修改，只需要按下ok按钮即可。
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220251_89589.jpg)

安装完成前，出现以下的设置脚本：
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220351_27433.jpg)

开启一个新的终端，su到root。
将要求执行的两段script依序执行。
/opt/oracle/oraInventory/orainstRoot.sh
/opt/oracle/102/root.sh
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220426_11212.jpg)

执行画面如上图。
执行完后，回到安装窗口按下OK完成所有的oracle安装。安装完成会出现以下画面。安装
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220520_19858.jpg)

此时，您可以以上述网址，作为测试，登入账号可以为sys或system
http://CentOS-Oracle:5560/isqlplus
http://CentOS-Oracle:5560/isqlplus/dba
http://CentOS-Oracle:1158/em
![安装](https://static.verycloud.cn/sites/default/files/pic/image/20151118/20151118220623_26138.jpg)

以上画面都成功代表oracle已经正常安装了。

