title: infiniDB安装配置
date: 2015-06-24 13:48:03
categories: mysql
tags: [infiniDB]
---

infiniDB是为大数据而生的列式数据库，适合做统计分析，如SUM,AVG,MAX,MIN等。笔者没有研究过infoBright，貌似需要收费，并且社区版本不支持DML。

总结了一些infiniDB的特点:
1、所有字段的默认值都为NULL，不需要设置其他值。
2、不需要创建索引，不需要优化表
3、多线程设计，查询时完美利用多CPU
4、高并发:理论上无并发的限制,只受制于服务器的容量
5、DML(可以视为是SQL的子集)支持 : 语句insert, update, delete
6、数据存储方面主要是按列拆，按行(范围)拆，核心算法是hash join，跟oracle很类似。
	具体有以下几方面：
	1）、Block : 8k的数据块,有Logical Block ID,大小不能定制,但预读的数目可以定制
	2）、Extent : 一个逻辑空间尺寸,存在于一个或多个的称为segment文件的物理文件中. extent大小受1)默认的行数2)一个列的数据类型, 如默认行数是8M,对一字节数据类型来说,Extent大小就是8M; 对于8字节数据类型来说,就是64M;对于可变长数据类型来说也是64M.当一个Extent满了,一个新的Extent就会被创建出来.
	3）、Segment File : 当一个Segment文件达到它的Extent包含的最大数目,一个新的Segment文件就会被创建出来
	4）、Partition : 与row-based DB不同之处在于它是一个逻辑上的对象.由一个或多个Segment文件组成. 一个列的partition数目是不限的.

![数据存储示例图](https://static.verycloud.cn/sites/default/files/pic/image/20150624/20150624140759_49711.png)

> 安装过程：

```bash
1、访问https://github.com/infinidb/infinidb/blob/4.6.2-1/INSTALL，按照帮助文档进行安装，笔者安装的版本是4.6.2

或者：

cd /root
git clone http://github.com/infinidb/mysql
git clone http://github.com/infinidb/infinidb
-- or --
tar -zxf <srcfile>
cd mysql
./configure --prefix=/data/mysql/
make
make install
cd ../infinidb
./configure --prefix=/data/
make
make install

注意：root下的文件夹名称必须为infinidb和mysql，否则编译的时候会报错。

2、安装完以后执行/usr/local/Calpont/bin/postConfigure进行配置，按默认就行
3、设置一些别名， . /usr/local/Calpont/bin/calpontAlias
4、直接访问idbmysql进入管理。
```
![idbmysql](https://static.verycloud.cn/sites/default/files/pic/image/20150624/20150624141731_29245.jpg)


> 测试

10亿条数据count:
![10亿条数据count速度](https://static.verycloud.cn/sites/default/files/pic/image/20150624/20150624141741_84796.jpg)

性能杠杠滴！