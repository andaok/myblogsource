---
title: 封装tengine为docker镜像
date: 2017-12-20 11:18:54
updated: 2017-12-20 11:18:54
tags: [docker,nginx]
categories: docker
top: 9
---

> #### xtengine

封装tengine为docker镜像, 基于cloudposse/tengine,增加了状态监控和动态调整upstream pool,后期会加入监控和管理UI。

> ##### install & config

docker run -d -p 80:80 -p 443:443 -p 8081:8081  itwye/xtengine

在你的工作主机增加dns解析, 如在"/etc/hosts"增加如下, $docker_server请替换为你的docker宿主机IP.

```
$docker_server prod.xtengine.com
$docker_server test.xtengine.com
$docker_server manage.xtengine.com
```

> ##### usage

upstreams detail 
```
curl  http://manage.xtengine.com:8081/detail
```
upstreams list
```
curl  http://manage.xtengine.com:8081/list
```
单个upstream detail
```
curl  http://manage.xtengine.com:8081/upstream/prod.xtengine.com
```

upstream其它操作请见:  [upstream other operate](http://tengine.taobao.org/document/http_dyups.html)

xtengine requset statistics
```
curl  http://manage.xtengine.com:8081/reqstat
```
xtengine vts module
```
http://manage.xtengine.com:8081/ng_status
```
访问测试用upstream
```
curl http://test.xtengine.com  # 8089
curl http://prod.xtengine.com  # 8088
```




