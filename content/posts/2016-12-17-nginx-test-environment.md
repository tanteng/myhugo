---
title: 使用 Nginx 反向代理和负载均衡搭建多人测试环境
author: 小谈
type: post
date: 2016-12-17T06:19:44+00:00
url: /2016/12/nginx-test-environment/
duoshuo_thread_id:
  - 6364950772873954050
categories:
  - Develop
  - 服务器

---
<img class="alignnone size-full wp-image-11120" src="https://blog.tanteng.me/wp-content/uploads/2016/12/nginx-test.jpeg" alt="" width="1328" height="874" />

假如我们使用 git 进行版本控制，在一个大型网站中，开发人员在不同的分支上开发不同的需求，当一个需求开发完成需要测试，我们会把这个分支告诉测试人员，然后测试人员在测试机的网站目录拉取这个分支进行测试。

<!--more-->

设想这样一个场景：当只有一个分支提测的时候，在测试机上可以直接拉取这个分支的代码进行测试，但是如果同时有多个分支都要同时进行测试，那么就没办法在一台测试机上同时进行了。

因为这台测试机网站目录只有一个，我们无法同时拉取不同分支的代码，那么如何在一台测试机上同时支持多人测试不同的分支呢？

### 实现原理

当我们访问一个网站，请求头中会有 User-Agent 的头部，如 Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:50.0) Gecko/20100101 Firefox/50.0，这个 UA 是可以自定义的，很多浏览器插件也支持新增自定义 UA，如火狐的 User Agent Swicher 插件。

<img class="alignnone size-full wp-image-11126" src="https://blog.tanteng.me/wp-content/uploads/2016/12/ua-tanteng.jpeg" alt="" width="970" height="542" />

如图，这里新增了一个自定义 UA，内容改成了自己的名字，通过这个 UA 请求网站，我们可以在请求头中看到 UA 变成自己的了，这就是一个标识。

根据这个 UA ，通过 nginx 进行判断，不同的 UA 转发到不同的端口，每个端口下对应一个网站目录。

### Nginx 判断 UA

通过 Nginx 可以获取每次请求的 UA，即 $http\_user\_agent 变量。

如在 location 域中可以这样判断 UA 并设置一个标识：

<code class="lang:php decode:true ">if ( $http_user_agent ~ "dashen" ) {
    set $flag "01";
}</code>

比如判断 UA 内容为 dashen，设置 $flag 为 01，可以设置很多个这样的标识。

### Nginx 反向代理和负载均衡

通过判断不同的 UA，我们可以通过反向代理转发到不同的机器和端口，这里同一台测试机可以转发到本机的不同的端口，监听不同的端口设置不同的网站目录。

具体如下：

<code class="lang:php decode:true ">server
{
    listen 192.168.1.251:80;
    server_name *.example.com;
    index index.html index.htm index.php;
    charset utf-8;
	location / {
		set $flag "00";
		if ( $http_user_agent ~ "dashen" ) {
            set $flag "01";
        }
		if ( $http_user_agent ~ "mianwo" ) {
            set $flag "02";
        }
		if ( $http_user_agent ~ "bingkuai" ) {
            set $flag "03";
        }
		if ( $http_user_agent ~ "hadoop" ) {
            set $flag "04";
        }
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        if ( $flag = "00" ){
            add_header Z-Server mobile;
            proxy_pass http://webserver_mobile;
        }
        if ( $flag = "01" ){
            add_header Z-Server dashen;
            proxy_pass http://webserver_dashen;
        }
        if ( $flag = "02" ){
            add_header Z-Server mianwo;
            proxy_pass http://webserver_mianwo;
        }
        if ( $flag = "03" ){
            add_header Z-Server bingkuai;
            proxy_pass http://webserver_bingkuai;
        }
        if ( $flag = "04" ){
            add_header Z-Server hadoop;
            proxy_pass http://webserver_hadoop;
        }
	}
}</code>

这段配置将不同的 UA 请求反向代理到不同的负载均衡服务器，下面看具体的负载均衡配置。

### Nginx 负载均衡配置

这里定义了几个负载均衡配置，每个负载均衡配置实际上只配了一台机器，即本机的不同端口。

<code class="lang:php decode:true ">upstream webserver_mobile{
    server 127.0.0.1:8900 max_fails=2 weight=2 fail_timeout=10s;
}
upstream webserver_dashen{
    server 127.0.0.1:8901 max_fails=2 weight=2 fail_timeout=10s;
}
upstream webserver_mianwo{
    server 127.0.0.1:8902 max_fails=2 weight=2 fail_timeout=10s;
}
upstream webserver_bingkuai{
    server 127.0.0.1:8903 max_fails=2 weight=2 fail_timeout=10s;
}
upstream webserver_hadoop{
    server 127.0.0.1:8904 max_fails=2 weight=2 fail_timeout=10s;
}</code>

那么，还没结束，接下来需要做的是，像一个正常的网站那样去配置多个 Nginx 虚拟主机，不同的是这里需要监听几个不同的端口，就需要几个虚拟主机配置，只是网站的目录不同，如 UA 是 dashen ，对应网站目录是 /vhosts/example.com/dashen，如 UA 是 mianwo，对应的网站目录是 /vhosts/example.com/mianwo.

这样一来，不同的测试人员，在对应自己 UA 的网站目录下拉取分支，通过浏览器插件配置自己的 UA，就实现了多人同时在一台机器上拉取不同的分支，而且访问同样的域名，根据 UA 不同实现网站根目录分开，而且互不影响。