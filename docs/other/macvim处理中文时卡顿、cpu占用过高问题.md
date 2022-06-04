title: MacVim处理中文时卡顿、CPU占用过高问题
date: 2014-03-11 09:58:29
categories: linux
tags: [vim]
---

升级到vim 7.4以后情况有点好转。
---------------
---------------

我一般使用vim编程。但是输入中文和&符号时时比较痛苦。经常出现卡顿。

偶然发现，在Preferences对话框中选择Advanced选项卡，去掉Use Core Text renderer选项的勾勾，输入大段中文也不会造成卡顿，至此问题解决。

但是这样会导致屏幕上下滚动的时候有点晃眼。难道是要我抛弃vim的节奏？