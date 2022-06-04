title: Drupal7 WEB性能优化
date: 2015-08-17 11:11:30
categories: Drupal
tags: [WEB性能优化]
---

* 一、DNS prefetch

DNS 作为互联网的基础协议，其解析的速度似乎容易被网站优化人员忽视。现在浏览器厂商已经有在针对 DNS 进行优化，典型的一次 DNS 解析耗费 20-120 毫秒，减少 DNS 解析数是个优化的方式，而能够缩减 DNS 解析的时间也是有经济效益的事情。这就是浏览器厂商重视 DNS Prefetching 的主要原因。DNS Prefetching 对于性能的收益可以简单的用”DNS 同步请求到异步”来解释，也就是具有此属性的域名不需要用户点击链接就在后台解析，而域名解析和内容载入是串行的网络操作，所以这个方式能减少用户的等待时间，提升用户体验。

Google Chrome 内置就有 DNS Prefetching 技术 ，而 Firefox 3.5 也引入了这一 新特性。至于 IE，貌似还未支持。

Drupal如何起dns prefetch呢? 先上一段代码.

```php
/**
 * hook_html_head_alter()
 */
function themename_html_head_alter(&$header) {
  // Enable prefetching.
  $header['dns_prefetch_control'] = array(
    '#type' => 'html_tag',
    '#tag' => 'meta',
    '#attributes' => array('http-equiv' => 'x-dns-prefetch-control', 'content' => 'on'),
    '#weight' => -10001,
  );  
  
  $dns_prefetch_domains = array(
    'www.google.cn',
    'www.baidu.com',
  );
  foreach ($dns_prefetch_domains as $key => $value) {
    $header['dns_prefetch_control_' . $key] = array(
      '#type' => 'html_tag',
      '#tag' => 'link',
      '#attributes' => array('rel' => 'dns-prefetch', 'href' => '//' . drupal_strip_dangerous_protocols(trim($value))),
    );     
  }
}
```

打开浏览器，就可以看到如下的输出：
```bash
<meta content="on" http-equiv="x-dns-prefetch-control">
<link href="//www.google.cn" rel="dns-prefetch">
<link href="//www.baidu.com" rel="dns-prefetch">
```

* 二、启用压缩
压缩的目的是让传输的数据变得更小。我们的线上代码（JS、CSS 和 HTML）都会做压缩，图片也会做压缩（PNGOUT、Pngcrush、JpegOptim、Gifsicle 等）。对于文本文件，在服务端发送响应之前进行 GZip 压缩也很重要，通常压缩后的文本大小会减小到原来的 1/4 - 1/3。对代码进行内容压缩已经有成熟的工具和标准流程了，而服务端的 GZip 更是标配，所以「压缩」是一项收益投入比很高的优化手段。


* 三、使用 HTTP 缓存
任何一个 WEB 项目，要提高性能，各个环节的缓存必不可少。利用好 HTTP 协议的缓存机制，可以大幅减少传输数据，减少请求，这又是一项收益投入比超高的优化手段。这里把之前我写的 HTTP/1.1 缓存机制介绍翻出来：
首先，服务端可以通过响应头里的 Last-Modified（最后修改时间） 或者 ETag（内容特征） 标记实体。浏览器会存下这些标记，并在下次请求时带上 If-Modified-Since: 上次 Last-Modified 的内容 或 If-None-Match: 上次 ETag 的内容，询问服务端资源是否过期。如果服务端发现并没有过期，直接返回一个状态码为 304、正文为空的响应，告知浏览器使用本地缓存；如果资源有更新，服务端返回状态码 200、新的 Last-Modified、Etag 和正文。这个过程被称之为 HTTP 的协商缓存，通常也叫做弱缓存。
可以看到协商缓存并不会节省连接数，但是在缓存生效时，会大幅减小传输内容（304 响应没有正文，一般只有几百字节）。另外为什么有两个响应头都可以用来实现协商缓存呢？这是因为一开始用的 Last-Modified 有两个问题：1）只能精确到秒，1 秒内的多次变化反映不出来；2）时间采用绝对值，如果服务端 / 客户端时间不对都可能导致缓存失效。HTTP/1.1 并没有规定 ETag 的生成规则，而一般实现者都是对资源内容做摘要，能解决前面两个问题。
另外一种缓存机制是服务端通过响应头告诉浏览器，在什么时间之前（Expires）或在多长时间之内（Cache-Control: Max-age=xxx），不要再请求服务器了。这个机制我们通常称之为 HTTP 的强缓存。
一旦资源命中强缓存规则后，再次访问完全没有 HTTP 请求（Chrome 开发者工具的 Network 面板依然会显示请求，但是会注明 from cache；Firefox 的 firebug 也类似，会注明 BFCache），这会大幅提升性能。所以我们一般会对 CSS、JS、图片等资源使用强缓存，而入口文件（HTML）一般使用协商缓存或不缓存，这样可以通过修改入口文件中对强缓存资源的引入 URL 来达到即时更新的目的。
这里也解释下为什么有了 Expire，还要有 Cache-Control。也有两个原因：1）Cache-Control 功能更强大，对缓存的控制能力更强；2）Cache-Control 采用的 max-age 是相对时间，不受服务端 / 客户端时间不对的影响。
另外关于浏览器的刷新（F5 / cmd + r）和强刷（Ctrl + F5 / shift + cmd +r）：普通刷新会使用协商缓存，忽略强缓存；强刷会忽略浏览器所有缓存（并且请求头会携带 Cache-Control:no-cache 和 Pragma:no-cache，用来通知所有中间节点忽略缓存）。只有从地址栏或收藏夹输入网址、点击链接等情况下，浏览器才会使用强缓存。

* 四、使用HTTP2
  1、HTTP/2 的 Server Push 机制，可以让重要的 JS、CSS 等资源尽快加载，从而不再需要 HTTP/1 中「将重要资源内联在页面头部」的优化方案了。
  
  2、HTTP/2 支持了多路复用，HTTP 连接变得十分廉价，之前为了节省连接数所采用的类似于「资源合并、资源内联」等优化手段不再需要了。多路复用可以在一个 TCP 连接上建立大量 HTTP 连接，也就不存在 HTTP 连接数限制了，HTTP/1 中常见的「静态域名」优化策略不但用不上了，还会带来负面影响，需要去掉。另外，HTTP/2 的头部压缩功能也能大幅减少 HTTP 协议头部带来的开销。

* 五、使用CDN
	国内比较好的CDN服务商如VeryCloud等。