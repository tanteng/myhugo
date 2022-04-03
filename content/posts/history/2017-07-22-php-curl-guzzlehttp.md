---
title: PHP发送HTTP请求的几种方式
author: 小谈
type: post
date: 2017-07-22T06:02:45+00:00
url: /2017/07/php-curl-guzzlehttp/
categories:
  - Develop
  - PHP

---
PHP 开发中我们常用 cURL 方式封装 HTTP 请求，什么是 cURL？

cURL 是一个用来传输数据的工具，支持多种协议，如在 Linux 下用 curl 命令行可以发送各种 HTTP 请求。PHP 的 cURL 是一个底层的库，它能根据不同协议跟各种服务器通讯，HTTP 协议是其中一种。

<!--more-->

现代化的 PHP 开发框架中经常会用到一个包，叫做 GuzzleHttp，它是一个 HTTP 客户端，也可以用来发送各种 HTTP 请求，那么它的实现原理是什么，与 cURL 有何不同呢？

### Does Guzzle require cURL?

No. Guzzle can use any HTTP handler to send requests. This means that Guzzle can be used with cURL, PHP&#8217;s stream wrapper, sockets, and non-blocking libraries like React. You just need to configure an HTTP handler to use a different method of sending requests.

这是 GuzzleHttp 文档 FAQ 中的一个 Question，可见 GuzzleHttp 并不依赖 cURL 库，而支持多种发送 HTTP 请求的方式。

### PHP 发送 HTTP 请求的方式

那么这里整理一下除了使用 cURL 外 PHP 发送 HTTP 请求的方式。

#### 1.cURL

略过

#### 2.stream流的方式

stream\_context\_create 作用：创建并返回一个文本数据流并应用各种选项，可用于 fopen(), file\_get\_contents() 等过程的超时设置、代理服务器、请求方式、头信息设置的特殊过程。

以一个 POST 请求为例：

<code>$data = array("uid" =&gt; $userid, "coin" =&gt; $stamps, "sign" =&gt; $sign, "type" =&gt; 'vip_act_chaihongbao');
$data = http_build_query($data);
$opts = array(
    'http' =&gt; array(
        'method'  =&gt; "POST",
        'header'  =&gt; "Content-type: application/x-www-form-urlencoded\r\nContent-length:" . strlen($data) . "\r\n",
        'content' =&gt; $data,
    )
);
$context = stream_context_create($opts);
$result = file_get_contents("http://api.example.com/user/addCoin", false, $context);
$result = json_decode($result, true);
if (isset($result['ret']) && $result['ret'] == 0) {
    return true;
}</code>

关于 PHP stream 的介绍文章：<a href="https://www.oschina.net/translate/understanding-streams-in-php" target="_blank" rel="noopener nofollow">https://www.oschina.net/translate/understanding-streams-in-php</a>

#### 3.socket方式

使用套接字建立连接，拼接 HTTP 报文发送数据进行 HTTP 请求。

一个 GET 方式的例子：

<code class="lang:php decode:true ">&lt;?php
$fp = fsockopen("www.example.com", 80, $errno, $errstr, 30);
if (!$fp) {
    echo "$errstr ($errno)&lt;br /&gt;\n";
} else {
    $out = "GET / HTTP/1.1\r\n";
    $out .= "Host: www.example.com\r\n";
    $out .= "Connection: Close\r\n\r\n";
    fwrite($fp, $out);
    while (!feof($fp)) {
        echo fgets($fp, 128);
    }
    fclose($fp);
}
?&gt;</code>

本文介绍了发送 HTTP 请求的几种不同的方式。