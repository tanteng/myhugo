---
title: Let’s Encrypt 开启多域名 HTTPS
author: 小谈
type: post
date: 2016-09-12T11:26:47+00:00
url: /2016/09/lets-encrypt-multi-domain-https/
categories:
  - 服务器

---
上周去腾讯参加了 IMWEB 前端大会，听了关于 HTTPS 的讲座，回来把自己的主页和博客升级了一下，开启了 HTTPS.本文主要记录一下开启 HTTPS 的过程，以及碰到的问题和解决方法，以及如何多域名开启 HTTPS，做进一步的补充。

Let&#8217;s Encrypt 这里就不详细介绍了，开启 HTTPS 的方法建议先参考这篇文章：<a href="https://phphub.org/topics/2766" target="_blank" rel="nofollow">https://phphub.org/topics/2766</a>，如果有问题，或者需要开启多个子域名 HTTPS，再参考本文。

本站有若干个子域名，其中给 www.tanteng.me 和 blog.tanteng.me 开通 https.首先给主站 tanteng.me,www.tanteng.me 开启 HTTPS.

### 1.克隆 acme-tiny 项目

<code class="lang:php decode:true ">sudo git clone https://github.com/diafygi/acme-tiny.git  
cd acme-tiny</code>

### 2.创建私钥

<code>openssl genrsa 4096 &gt; account.key</code>

这个就是账户，一个账户可以开启多个域名证书。

### 3.创建 CSR 文件

<code>openssl genrsa 4096 &gt; www.key     
openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config &lt;(cat /etc/ssl/openssl.cnf &lt;(printf "[SAN]\nsubjectAltName=DNS:yoursite.com,DNS:www.yoursite.com")) &gt; domain.csr</code>

这里就遇到了问题，提示 /etc/ssl/openssl.cnf 文件找不到。服务器上这个路径确实没有 openssl.cnf 这个文件，但是系统默认安装了 openssl，于是使用 find 命令搜索了一下，发现文件路径是 /etc/pki/tls/openssl.cnf.

于是创建命令为：

<code>openssl req -new -sha256 -key www.key -subj "/" -reqexts SAN -config &lt;(cat /etc/ssl/openssl.cnf &lt;(printf "[SAN]\nsubjectAltName=DNS:tanteng.me,DNS:www.tanteng.me")) &gt; www.csr</code>

### 4. nginx 配置文件修改

修改 nginx 配置，在 80 端口下，增加以下语句：

<code>location /.well-known/acme-challenge/ {
    alias /var/www/challenges/;
    try_files $uri =404;
}</code>

### 5. 获取签名证书

先手动创建一个目录，用来存放验证文件：

mkdir -p /var/www/challenges

再执行以下命令获取签名证书，注意 csr 文件这里使用前面步骤生成的 www.csr.

<code>sudo chmod +x acme_tiny.py  
python acme_tiny.py --account-key ./account.key --csr ./www.csr --acme-dir /var/www/challenges/ &gt; ./signed.crt</code>

如果一切顺利，应该显示如下内容：

<code class="lang:php decode:true ">Parsing account key...
Parsing CSR...
Registering account...
Registered!
Verifying www.tanteng.me...
www.tanteng.me verified!
Verifying tanteng.me...
tanteng.me verified!
Signing certificate...
Certificate signed!</code>

这就表明验证通过了。

### 6. 安装证书

<code class="lang:php decode:true ">wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem &gt; intermediate.pem  
cat signed.crt intermediate.pem &gt; chained.pem</code>

最后在 nginx 配置文件增加 443 端口配置：

<code class="lang:php decode:true ">server {
    listen 443;
    server_name *.tanteng.me;
    root   /usr/share/nginx/html/tanteng.me/public;
    index  index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files      $uri =404;
        root           /usr/share/nginx/html/tanteng.me/public;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;  
    }

    ssl on;
    ssl_certificate /home/tanteng/acme-tiny/chained.pem;
    ssl_certificate_key /home/tanteng/acme-tiny/tanteng.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA;
    ssl_session_cache shared:SSL:50m;
    ssl_prefer_server_ciphers on;
}</code>

同时，可以添加一个 80 端口添加 301 重定向的配置，将 http 请求重定向到 https：

<code class="lang:php decode:true ">server {
    listen       80;
    server_name  tanteng.me www.tanteng.me;
    return 301 https://www.tanteng.me$request_uri;
}
</code>

这样，主站就开启了 HTTPS，可以通过浏览器访问。

### 多域名开启 HTTPS

以上仅是给 www.tanteng.me 主域名开启了 HTTPS，但是访问博客 blog.tanteng.me 仍然是 HTTP 协议，给子域名开启 HTTP 只需要重复以上的步骤。

1.创建 csr 文件

<code class="lang:php decode:true ">openssl genrsa 4096 &gt; blog.key     
openssl req -new -sha256 -key blog.key -subj "/" -reqexts SAN -config &lt;(cat /etc/ssl/openssl.cnf &lt;(printf "[SAN]\nsubjectAltName=DNS:blog.tanteng.me")) &gt; blog.csr</code>

注意这里生成一个 blog.key.

2.在博客的 nginx 配置 80 端口添加：

<code class="lang:php decode:true ">location /.well-known/acme-challenge/ {
    alias /var/www/challenges/;
    try_files $uri =404;
}</code>

3.获取签名证书

<code class="lang:php decode:true ">python acme_tiny.py --account-key ./account.key --csr ./blog.csr --acme-dir /var/www/challenges/ &gt; ./blog_signed.crt</code>

注意这里使用 blog.csr，并且生成文件名改成 blog_signed.crt，也就是独立的 crt 文件。

没问题的话，执行结果如下所示：

<code class="lang:php decode:true ">[root@iZ94r80gdghZ acme-tiny]# python acme_tiny.py --account-key ./account.key --csr ./blog.csr --acme-dir /var/www/challenges/ &gt; ./blog_signed.crt
Parsing account key...
Parsing CSR...
Registering account...
Already registered!
Verifying blog.tanteng.me...
blog.tanteng.me verified!
Signing certificate...
Certificate signed!</code>

4.安装证书

<code>cat blog_signed.crt intermediate.pem &gt; blog_chained.pem</code>

这里注意修改第一个 crt 文件是 blog_singed.crt ，生成新的 pem 文件。

5.nginx ssl 配置并使用新的路径，同时 80 端口做 301 重定向。

### 开启 HTTPS 需要注意的问题

成功开启 HTTPS 后，仍需要注意一些问题，网站的所有图片，css，js，都必须替换成 https 协议，如果使用的是 CDN，如七牛CDN，需要到七牛后台开启 https 域名。

如果是调用的第三方接口，而接口没有 https 协议的，那么将无法使用。