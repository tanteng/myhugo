---
title: window.location.reload()和window.location.href=””的区别
author: 深 呼吸
type: post
date: 2016-12-31T02:55:00+00:00
url: /2016/12/window-location-reload-and-href/
duoshuo_thread_id:
  - 6403492630994354945
categories:
  - Develop
  - 前端

---
window.location.reload() 和 window.location.href=&#8221;&#8221; 都可以达到“刷新”当前页面的效果，但是 reload() 会保留 POST 数据，而 href 相当于重新打开这个页面。

<a href="https://blog.tanteng.me/wp-content/uploads/2017/03/window-location-reload.png" target="_blank"><img class="alignnone wp-image-11348 size-full" src="https://blog.tanteng.me/wp-content/uploads/2017/03/window-location-reload.png" alt="window.location.reload()和window.location.href=''的区别" width="368" height="207" /></a>

<!--more-->

如图这个文本框和按钮，是 post 方式 ajax 请求，点击按钮状态变为不可用，同时加减分数，返回请求成功则刷新页面，如果使用 window.location.reload() 刷新页面，可以看到文本框的内容还是保留，并且按钮的状态并没有恢复，使用 window.location.href=&#8221;&#8221; 的方式就好了。