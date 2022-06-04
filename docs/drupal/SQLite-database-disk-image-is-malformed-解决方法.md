title: Drupal SQLite: database disk image is malformed 解决方法
date: 2015-07-31 13:22:07
categories: linux
tags: [sqlite]
---

我碰到的这种情况，并不是网上说的磁盘坏了。而是由于频繁读写，并且有BLOB大数据频繁读写。

解决办法就是通过导出导入方式对损坏的库文件作恢复。

首先导出数据

```bash
sqlite3 my.sqlite3
sqlite>.output tmp.sql
sqlite>.dump
sqlite>.quit
```

再导入到一个新库中

```bash
sqlite3 new.sqlite3
sqlite>.read tmp.sql
sqlite>.quit
```

覆盖new.sqlite文件到sites/default/files/.ht.sqlite即可。

另外，如果只是临时数据，可以放到/dev/shm，就是内存数据库啦。。。