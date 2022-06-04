title: windows下安装配置drupalconsole
date: 2016-10-24 17:38:17
tags:
  - drupalconsole
  - windows
categories:
  - drupal
---


### drupalconsole是啥

> The new CLI for Drupal. A tool to generate boilerplate code, interact with and debug Drupal.

以上是官网的简介，总而言之是为drupal8而生的一个命令行工具

[这里是官网](https://drupalconsole.com/)

### drupalconsole在windows下的安装

本篇仅介绍在windows下的安装配置，linux请参见官网

1、下载`drupalconsole`

[点击这里](https://drupalconsole.com/installer)下载`drupalconsole`，更名`drupal`

2、找个目录给他安顿

例如：`D:\system\drupalconsole`，如下图所示：

![drupalconsole家目录](https://static.verycloud.cn/sites/default/files/images/drupalconsole_home.png)

3、配置环境变量

把`php.exe`所在的目录加入环境变量`PATH`中

再把上一步`drupalconsole`的安顿目录加入`windows`的环境变量`PATH`中，用户或者系统的都可以

如下图：

![drupalconsole家目录](https://static.verycloud.cn/sites/default/files/images/system_path.png)

4、打开`git`的命令行工具测试一下

![drupalconsole测试](https://static.verycloud.cn/sites/default/files/images/drupalconsole.png)

> 再插一句，貌似windows不认识非exe的可执行程序，需要安装个git的命令行工具
