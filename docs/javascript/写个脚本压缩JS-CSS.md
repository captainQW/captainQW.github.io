title: 写个脚本压缩JS,CSS
date: 2015-05-07 15:00:13
categories: shell
tags: yuicompressor
---

1、下载最新版的yuicompressor.jar
```php
https://github.com/yui/yuicompressor/releases
```

2、遍历目录，压缩JS、CSS
```php
#!/bin/bash
for i in `find path -name "*.css"`;
do
    echo "compress $i"
    java -jar yuicompressor-2.4.8.jar --charset=utf8 -o $i $i --nomunge
done
```