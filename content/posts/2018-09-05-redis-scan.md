---
title: Redis生产环境查看数据库键
author: 深 呼吸
type: post
date: 2018-09-05T14:17:03+00:00
url: /2018/09/redis-scan/
categories:
  - Develop
format: aside

---
如果要在redis生产环境服务器查看有哪些数据库键，当数据量特别大的时候，千万不要用 keys * ，这样会卡死，可以使用 scan 命令迭代。