title: Haproxy web负载均衡
date: 2016-02-26 10:02:54
categories: linux
tags: [haproxy]
---

### 一、安装

```bash
yum install haproxy
```

修改日志
```bash
vim /etc/rsyslog.conf
```

这一行
```bash
#*.info;mail.none;authpriv.none;cron.none                /var/log/messages
```

改为
```bash
*.info;mail.none;authpriv.none;cron.none;local2.none                /var/log/messages
```

再增加一行
```bash
# haproxy
local2.*                                                /var/log/haproxy.log
```

### 二、haproxy配置详解 
HAProxy配置中分五大部分：
global：全局配置参数，进程级的，用来控制Haproxy启动前的一些进程及系统设置
defaults：配置一些默认的参数，可以被frontend，backend，listen段继承使用
frontend：用来匹配接收客户所请求的域名，uri等，并针对不同的匹配，做不同的请求处理
backend：定义后端服务器集群，以及对后端服务器的一些权重、队列、连接数等选项的设置，我将其理解为Nginx中的upstream块
listen：frontend和backend的组合体

配置案例：

```bash
global   # 全局参数的设置 
     log 127.0.0.1 local0 info 
     # log语法：log [max_level_1] 
     # 全局的日志配置，使用log关键字，指定使用127.0.0.1上的syslog服务中的local0日志设备，
     记录日志等级为info的日志 
     user haproxy 
     group haproxy 
     # 设置运行haproxy的用户和组，也可使用uid，gid关键字替代之 
     daemon 
     # 以守护进程的方式运行 
     nbproc 16
     # 设置haproxy启动时的进程数，根据官方文档的解释，我将其理解为：该值的设置应该和服务
     #器的CPU核心数一致，即常见的2颗8核心CPU的服务器，即共有16核心，则可以将其值设置为：
     #<=16 ，创建多个进程数，可以减少每个进程的任务队列，但是过多的进程数也可能会导致进程
     #的崩溃。这里我设置为16 
     maxconn 4096 
     # 定义每个haproxy进程的最大连接数 ，由于每个连接包括一个客户端和一个服务器端，所以单
     #个进程的TCP会话最大数目将是该值的两倍。 
     #ulimit -n 65536 
     # 设置最大打开的文件描述符数，在1.4的官方文档中提示，该值会自动计算，所以不建议进行
     #设置 
     pidfile /var/run/haproxy.pid 
     # 定义haproxy的pid
      
      
defaults # 默认部分的定义
     mode http
     # mode语法：mode {http|tcp|health} 。http是七层模式，tcp是四层模式，health是健康检测
     #，返回OK
     log 127.0.0.1 local2 err
     # 使用127.0.0.1上的syslog服务的local3设备记录错误信息
     retries 3
     # 定义连接后端服务器的失败重连次数，连接失败次数超过此值后将会将对应后端服务器标记为
     #不可用
		 option http-server-close
		 # 打开http协议中服务器端关闭功能，使得支持长连接，使得会话可以被重用，使得每一个日志记录都会被记录。
		 option forwardfor except 127.0.0.0/8
		 # 如果上游服务器上的应用程序想记录客户端的真实IP地址，haproxy会把客户端的IP信息发送给上游服务器，在HTTP请求中添加”X-Forwarded-For”字段,但当是haproxy自身的健康检测机制去访问上游服务器时是不应该把这样的访问日志记录到日志中的，所以用except来排除127.0.0.0，即haproxy身。

     option httplog
     # 启用日志记录HTTP请求，默认haproxy日志记录是不记录HTTP请求的，只记录“时间[Jan 5 13
     #:23:46] 日志服务器[127.0.0.1] 实例名已经pid[haproxy[25218]] 信息[Proxy http_80_in s
     #topped.]”，日志格式很简单。
     option redispatch
     # 当使用了cookie时，haproxy将会将其请求的后端服务器的serverID插入到cookie中，以保证
     #会话的SESSION持久性；而此时，如果后端的服务器宕掉了，但是客户端的cookie是不会刷新的
     #，如果设置此参数，将会将客户的请求强制定向到另外一个后端server上，以保证服务的正常
     option abortonclose
     # 当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接
     option dontlognull
     # 启用该项，日志中将不会记录空连接。所谓空连接就是在上游的负载均衡器或者监控系统为了
     #探测该服务是否存活可用时，需要定期的连接或者获取某一固定的组件或页面，或者探测扫描
     #端口是否在监听或开放等动作被称为空连接；官方文档中标注，如果该服务上游没有其他的负
     #载均衡器的话，建议不要使用该参数，因为互联网上的恶意扫描或其他动作就不会被记录下来
     option httpclose
     # 这个参数我是这样理解的：使用该参数，每处理完一个request时，haproxy都会去检查http头
     #中的Connection的值，如果该值不是close，haproxy将会将其***，如果该值为空将会添加为：
     #Connection: close。使每个客户端和服务器端在完成一次传输后都会主动关闭TCP连接。与该
     #参数类似的另外一个参数是“option forceclose”，该参数的作用是强制关闭对外的服务通道
     #，因为有的服务器端收到Connection: close时，也不会自动关闭TCP连接，如果客户端也不关
     #闭，连接就会一直处于打开，直到超时。
		timeout http-request 10s
		# 客户端发送http请求的超时时间。
		timeout queue 1m
		# 当上游服务器在高负载响应haproxy时，会把haproxy发送来的请求放进一个队列中，timeout queue定义放入这个队列的超时时间。
		timeout connect 5s
		# haproxy与后端服务器连接超时时间，如果在同一个局域网可设置较小的时间。
		timeout client 1m
		# 定义客户端与haproxy连接后，数据传输完毕，不再有数据传输，即非活动连接的超时时间。
		timeout server 1m
		# 定义haproxy与上游服务器非活动连接的超时时间。
		timeout http-keep-alive 10s
		# 设置新的http请求连接建立的最大超时时间，时间较短时可以尽快释放出资源，节约资源。
		timeout check 10s
		# 健康检测的时间的最大超时时间。
     contimeout 5000
     # 设置成功连接到一台服务器的最长等待时间，默认单位是毫秒，新版本的haproxy使用timeout
     #connect替代，该参数向后兼容
     clitimeout 3000
     # 设置连接客户端发送数据时的成功连接最长等待时间，默认单位是毫秒，新版本haproxy使用
     #timeout client替代。该参数向后兼容
     srvtimeout 3000
     # 设置服务器端回应客户度数据发送的最长等待时间，默认单位是毫秒，新版本haproxy使用
     #timeout server替代。该参数向后兼容
      
listen status 
     # 定义一个名为status的部分，可以在listen指令指定的区域中定义匹配规则和后端服务器ip，
     #相当于需要在其中配置frontend，backend的功能。一般做tcp转发比较合适，不用太多的规则
     #匹配。
     bind 0.0.0.0:1080
     # 定义监听的套接字
     mode http
     # 定义为HTTP模式
     log global
     # 继承global中log的定义
     stats refresh 30s
     # stats是haproxy的一个统计页面的套接字，该参数设置统计页面的刷新间隔为30s
     stats uri /admin?stats
     # 设置统计页面的uri为/admin?stats
     stats realm Private lands
     # 设置统计页面认证时的提示内容
     stats auth admin:password
     # 设置统计页面认证的用户和密码，如果要设置多个，另起一行写入即可
     stats hide-version
     # 隐藏统计页面上的haproxy版本信息
      
      
frontend http_80_in # 定义一个名为http_80_in的前端部分，haproxy会监听bind的端口
     bind 0.0.0.0:80
     # http_80_in定义前端部分监听的套接字
     mode http
     # 定义为HTTP模式
     log global
     # 继承global中log的定义
     option forwardfor
     # 启用X-Forwarded-For，在requests头部插入客户端IP发送给后端的server，使后端server获
     #取到客户端的真实IP
     acl static_down nbsrv(static_server) lt 1
     # 定义一个名叫static_down的acl，当backend static_sever中存活机器数小于1时会被匹配到
     acl php_web url_reg /*.php$
     #acl php_web path_end .php
     # 定义一个名叫php_web的acl，当请求的url末尾是以.php结尾的，将会被匹配到，上面两种写
     #法任选其一
     acl static_web url_reg /*.(css|jpg|png|jpeg|js|gif)$
     #acl static_web path_end .gif .png .jpg .css .js .jpeg
     # 定义一个名叫static_web的acl，当请求的url末尾是以.css、.jpg、.png、.jpeg、.js、.gif
     #结尾的，将会被匹配到，上面两种写法任选其一
     use_backend php_server if static_down
     # 如果满足策略static_down时，就将请求交予backend php_server
     use_backend php_server if php_web
     # 如果满足策略php_web时，就将请求交予backend php_server
     use_backend static_server if static_web
     # 如果满足策略static_web时，就将请求交予backend static_server
     default_backend             defaultbackserver
     #如果acl全部不满足，启用默认后端的地址池
      
      
backend php_server #定义一个名为php_server的后端部分，frontend定义的请求会到到这里处理
     mode http
     # 设置为http模式
     balance source
     # 设置haproxy的调度算法为源地址hash
     cookie SERVERID
     # 允许向cookie插入SERVERID，每台服务器的SERVERID可在下面使用cookie关键字定义
     option httpchk GET /test/index.php
     # 开启对后端服务器的健康检测，通过GET /test/index.php来判断后端服务器的健康情况
     server php_server_1 10.12.25.68:80 cookie 1 check inter 2000 rise 3 fall 3 weight 2
     server php_server_2 10.12.25.72:80 cookie 2 check inter 2000 rise 3 fall 3 weight 1
     server php_server_bak 10.12.25.79:80 cookie 3 check inter 1500 rise 3 fall 3 backup
     # server语法：server [:port] [param*]
     # 使用server关键字来设置后端服务器；为后端服务器所设置的内部名称[php_server_1]，该名
     #称将会呈现在日志或警报中、后端服务器的IP地址，支持端口映射[10.12.25.68:80]、指定该
     #服务器的SERVERID为1[cookie 1]、接受健康监测[check]、监测的间隔时长，单位毫秒[inter 
     #2000]、监测正常多少次后被认为后端服务器是可用的[rise 3]、监测失败多少次后被认为后端
     #服务器是不可用的[fall 3]、分发的权重[weight 2]、最为备份用的后端服务器，当正常的服
     #务器全部都宕机后，才会启用备份服务器[backup]
      
      
backend static_server
     mode http
     option httpchk GET /test/index.html
     server static_server_1 10.12.25.83:80 cookie 3 check inter 2000 rise 3 fall 3
      
      
 官方配置：
 http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
```

### 三、haproxy实现持久连接
1、调度算法source
haroxy 将用户IP经过hash计算后 指定到固定的真实服务器上（类似于nginx 的IP hash 指令）
配置指令        balance source
2、cookie 识别
haproxy 将WEB服务端发送给客户端的cookie中插入(或添加加前缀)haproxy定义的后端的服务器COOKIE ID。
配置指令例举  cookie  SESSION_COOKIE  insert indirect nocache
3、session 识别
haproxy 将后端服务器产生的session和后端服务器标识存在haproxy中的一张表里。客户端请求时先查询这张表。然后根据session分配后端server。
配置指令：appsession <cookie> len <length> timeout <holdtime>