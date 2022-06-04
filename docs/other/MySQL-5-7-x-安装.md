title: mysql-5.7.x安装
date: 2016-04-25 13:23:29
categories: mysql
tags: [mysql_install]
---

###1、添加yum源

#### wget
```bash
--------------- On RHEL/CentOS 7 ---------------
# wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```

```bash
--------------- On RHEL/CentOS 6 ---------------
# wget http://dev.mysql.com/get/mysql57-community-release-el6-7.noarch.rpm
```

```bash
--------------- On RHEL/CentOS 5 ---------------
# wget http://dev.mysql.com/get/mysql57-community-release-el5-7.noarch.rpm
```

```bash
--------------- On Fedora 23 ---------------
# wget http://dev.mysql.com/get/mysql57-community-release-fc23-7.noarch.rpm
```

```bash
--------------- On Fedora 22 ---------------
# wget http://dev.mysql.com/get/mysql57-community-release-fc22-7.noarch.rpm
```

```bash
--------------- On Fedora 21 ---------------
# wget http://dev.mysql.com/get/mysql57-community-release-fc21-7.noarch.rpm
```

####安装

```bash
--------------- On RHEL/CentOS 7 ---------------
# yum localinstall mysql57-community-release-el7-7.noarch.rpm
```

```bash
--------------- On RHEL/CentOS 6 ---------------
# yum localinstall mysql57-community-release-el6-7.noarch.rpm
```

```bash
--------------- On RHEL/CentOS 5 ---------------
# yum localinstall mysql57-community-release-el5-7.noarch.rpm
```

```bash
--------------- On Fedora 23 ---------------
# dnf localinstall mysql57-community-release-fc23-7.noarch.rpm
```

```bash
--------------- On Fedora 22 ---------------
# dnf localinstall mysql57-community-release-fc22-7.noarch.rpm
```

```bash
--------------- On Fedora 21 ---------------
# yum localinstall mysql57-community-release-fc21-7.noarch.rpm
```

####校验源
```bash
# yum repolist enabled | grep "mysql.*-community.*"
# dnf repolist enabled | grep "mysql.*-community.*"      [On Fedora 22+ versions]
```

![Verify-MySQL-Yum-Repository](https://static.verycloud.cn/sites/default/files/images/Verify-MySQL-Yum-Repository.png)

###2、安装mysql

```bash
# yum install mysql-community-server mysql mysql-libs mysql-devel
# dnf install mysql-community-server mysql mysql-libs mysql-devel     [On Fedora 22+ versions]
```

###3、开启mysql

```bash
service mysqld start
```

查看mysql版本
```bash
# mysql --version
mysql  Ver 14.14 Distrib 5.7.12, for Linux (x86_64) using  EditLine wrapper
```

###4、配置mysql

#### 查看默认密码
```bash
# grep 'temporary password' /var/log/mysqld.log
```

执行
```bash
# mysql_secure_installation
```

配置示例：

```bash
Securing the MySQL server deployment.

Enter password for user root: Enter New Root Password

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 2
Using existing password for root.

Estimated strength of the password: 50 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password: Set New MySQL Password

Re-enter new password: Re-enter New MySQL Password

Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```

至此，安装完成！