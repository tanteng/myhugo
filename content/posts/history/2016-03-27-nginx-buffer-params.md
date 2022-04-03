---
title: Nginx性能调优之buffer参数设置
author: 小谈
type: post
date: 2016-03-27T10:07:51+00:00
url: /2016/03/nginx-buffer-params/
duoshuo_thread_id:
  - 6266671996512240385
categories:
  - 服务器

---
打开Nginx的error.log日志文件，发现很多warn的警告错误，提示：

①2016/03/25 13:18:35 [warn] 1171#0: *10875 an upstream response is buffered to a temporary file /var/cache/nginx/fastcgi_temp/0/08/0000000080 while reading upstream, client: 106.38.241.105, server: blog.tanteng.me, request: “GET /page/38/ HTTP/1.1”, upstream: “fastcgi://127.0.0.1:9000”, host: “blog.tanteng.me”.

<!--more-->

如图所示：

<a href="http://static.blog.tanteng.me/wp-content/uploads/2016/03/nginx_error_2.png" target="_blank"><img class="alignnone wp-image-9747 size-full" title="Nginx错误日志截图" src="http://static.blog.tanteng.me/wp-content/uploads/2016/03/nginx_error_2.png" alt="nginx_error_2" width="1118" height="468" /></a>

以及这样的警告：

②2016/03/25 15:16:07 [warn] 6172#0: *1243 a client request body is buffered to a temporary file /var/cache/nginx/client_temp/0000000001, client: 193.201.227.83, server: blog.tanteng.me, request: “POST /xmlrpc.php HTTP/1.1”, host: “blog.tanteng.me”, referrer: “http://tantengvip.com/xmlrpc.php”

这个需要设置增加client\_body\_buffer_size的大小。缓冲区设置小了，Nginx会把内容写到硬盘，这样会影响性能。于是在nginx.conf中增加如下fastcgi buffers参数设置：

<code class="lang:vim decode:true ">location ~ \.php$ {
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    fastcgi_connect_timeout 60;
    fastcgi_send_timeout 180;
    fastcgi_read_timeout 180;
    fastcgi_buffer_size 128k;
    fastcgi_buffers 256 16k;
    client_body_buffer_size 1024k;
    include        fastcgi_params;
}</code>

经过设置后，一段时间内都没有发现这种错误日志。

### **Nginx 的 buffer 机制**

对于来自 FastCGI Server 的 Response，Nginx 将其缓冲到内存中，然后依次发送到客户端浏览器。缓冲区的大小由 fastcgi\_buffers 和 fastcgi\_buffer_size 两个值控制。

比如如下配置：

<code class="lang:vim decode:true">fastcgi_buffers      8 4K;
fastcgi_buffer_size  4K;</code>

fastcgi\_buffers 控制 nginx 最多创建 8 个大小为 4K 的缓冲区，而 fastcgi\_buffer_size 则是处理 Response 时第一个缓冲区的大小，不包含在前者中。所以总计能创建的最大内存缓冲区大小是 8\*4K+4K = 36k。而这些缓冲区是根据实际的 Response 大小动态生成的，并不是一次性创建的。比如一个 8K 的页面，Nginx 会创建 2\*4K 共 2 个 buffers。

当 Response 小于等于 36k 时，所有数据当然全部在内存中处理。如果 Response 大于 36k 呢？fastcgi_temp 的作用就在于此。多出来的数据会被临时写入到文件中，放在这个目录下面。

### Buffer Size 优化

buffer的大小是你需要调优最重要参数。如果buffer size太小就会到导致nginx使用临时文件存储response，这会引起磁盘读写IO，流量越大问题越明显。

client\_body\_buffer\_size 处理客户端请求体buffer大小。用来处理POST提交数据，上传文件等。client\_body\_buffer\_size 需要足够大以容纳如果需要上传POST数据。

fastcgi\_buffers，proxy\_buffers 处理后端响应。如果这个buffer不够大，同样会引起磁盘都系IO。需要注意的是它们有一个上限值，这个上限值受 fastcgi\_max\_temp\_file\_size 、 proxy\_max\_temp\_file\_size控制。

### FastCGI缓冲设置主要参数

#### fastcgi_buffers 4 64k

这个参数指定了从FastCGI进程到来的应答，本地将用多少和多大的缓冲区读取，假设一个PHP或JAVA脚本所产生页面大小为256kb，那么会为其分配4个64kb的缓冲来缓存；若页面大于256kb，那么大于256kb的部分会缓存到fastcgi_temp指定路径中，这并非是个好办法，内存数据处理快于硬盘，一般该值应该为站点中PHP或JAVA脚本所产生页面大小中间值，如果站点大部分脚本所产生的页面大小为256kb，那么可把值设置为16 16k,4 64k等。

#### fastcgi\_buffer\_size=64k

读取fastcgi应答第一部分需要多大缓冲区，该值表示使用1个64kb的缓冲区读取应答第一部分(应答头),可以设置为fastcgi_buffers选项缓冲区大小。

#### fastcgi\_connect\_timeout=300

连接到后端fastcgi超时时间，单位秒，下同。

#### fastcgi\_send\_timeout=300

向fastcgi请求超时时间(这个指定值已经完成两次握手后向fastcgi传送请求的超时时间)

#### fastcgi\_reAd\_timeout=300

接收fastcgi应答超时时间，同理也是2次握手后。

##### 有用的链接

  * <a href="http://nginx.org/en/docs/http/ngx_http_core_module.html#client_body_buffer_size" target="_blank" rel="nofollow">Module ngx_http_core_module</a>
  * <a href="http://my.oschina.net/linland/blog/373315?p=1" target="_blank" rel="nofollow">Nginx缓冲区常见问题</a>