title: phantomjs中文问题
date: 2014-01-11 11:15:08
categories: javascript
tags: [phantomjs]
---

用phantomjs去截取中文页面的网站可能会出现乱码的情况，也就是截图中中文的位置全是方框。
解决办法就是安装字体。
在centos中执行：yum install bitmap-fonts bitmap-fonts-cjk
在ubuntu/debian中执行：sudo apt-get install xfonts-wqy
这样再去截图中文的页面就不会出现一堆的方框了。

如果需要使用微软雅黑字体，可以这样操作：

```bash
	mkdir /usr/share/fonts/win/ 
  下载微软雅黑字体: 
  wget https://nipao.googlecode.com/files/msyh.ttf -O /usr/share/fonts/win/msyh.ttf 
  建立字体索引，更新字体缓存: 
  cd /usr/share/fonts/win/ 
  mkfontscale 
  mkfontdir 
  fc-cache
```