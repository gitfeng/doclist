---
title:  Nginx 配置解释
categories: 技术
date: 2016-02-21 16:20:01
tags: [http]
description: 说明 nginx 的配置坑

---

## 配置结构

概览：
```
|--main			    nginx设置
   |--http		    http服务配置
      |--server	    虚拟主机配置
         |--location   特定url的配置
```

main域配置：
```
worker_processes  4;   #配置工作进程数量
worker_cpu_affinity 1000 0100 0010 0001; #并绑定至指定CPU

#配置日志位置和pid文件位置
error_log "/home/iknow/log/webserver/error_log" warn;
pid       "/home/iknow/log/nginx.pid";

#配置I/O方式和指定工作进程可使用连接数
events {
    use epoll;
    worker_connections  204800;
}
```
http 配置举例：
```
http{
	include			mime.types;
	default_type		application/octet-stream;
	sendfile			on;
	keepalive_timeout	65;

	server {
		listen 		80;
		server_name 	*.baidu.com;
		error_page 400 404 500 http://zhidao.baidu.com/errot/html
		location / {
			root 			/home/iknow/webroot;
			index 		index.php
			fastcgi_pass    unix:/var/php-cgi.sock;
			include 		vhost/dispatch.conf;
		}
	}
}

```

## 配置注意事项

1. 与lighttpd转发规则不同，rewrite只匹配path
rewrite '\^/submit(/[^\?]*)?$' /submit/index.php$1 break;

2. 如果需要匹配query_string，使用if加变量，注意这里的request_uri和arg_**
```
if ( $request_uri ~ "^/q\?pt=([^=]*)$" ) {
    rewrite "^/+q" /home/index.php?pt=$arg_pt break;
}
```

3. 注意rewrite规则中，break和last的区别，除非必要，请使用break。
   使用last的一个good case：
http://stackoverflow.com/questions/20470191/external-links-url-encoding-leads-to-3f-and-3d-on-nginx-server
```
last与break的区别在于，last并不会停止对下面location的匹配。
```

4. 302重定向
rewrite "\^/submit/(wap)(.*)?" http://zhidao.baidu.com/error.html? redirect;

5. 301重定向
if ( $request_uri ~ "\^/+q\?(.*)?(ct=17&(.*)?tn=ikaslist)(&.*)?$" ) {
    rewrite "^.*$" http://zhidao.baidu.com/search permanent;
}

## location 的匹配顺序

1. 误区： location 的匹配顺序是“先匹配正则，再匹配普通”。
矫正： location 的匹配顺序其实是“先匹配普通，再匹配正则”。我这么说，大家一定会反驳我，因为按“先匹配普通，再匹配正则”解释不了大家平时习惯的按“先匹配正则，再匹配普通”的实践经验。这里我只能暂时解释下，造成这种误解的原因是：正则匹配会覆盖普通匹配（实际的规则，比这复杂，后面会详细解释）。

2. 误区：location 的执行逻辑跟 location 的编辑顺序无关。
矫正：这句话不全对，“普通 location ”的匹配规则是“最大前缀”，因此“普通 location ”的确与 location 编辑顺序无关；但是“正则 location ”的匹配规则是“顺序匹配，且只要匹配到第一个就停止后面的匹配”；“普通location ”与“正则 location ”之间的匹配顺序是？先匹配普通 location ，再“考虑”匹配正则 location 。注意这里的“考虑”是“可能”的意思，也就是说匹配完“普通 location ”后，有的时候需要继续匹配“正则 location ”，有的时候则不需要继续匹配“正则 location ”。两种情况下，不需要继续匹配正则 location ：（ 1 ）当普通 location 前面指定了“ ^~ ”，特别告诉 Nginx 本条普通 location 一旦匹配上，则不需要继续正则匹配；（ 2 ）当普通location 恰好严格匹配上，不是最大前缀匹配，则不再继续匹配正则。

3. 总结一句话：  “正则 location 匹配让步普通 location 的严格精确匹配结果；但覆盖普通 location 的最大前缀匹配结果”

------- 搜索顺序以及生效优先级 -------
因为可以定义多个 Location 块，每个 Location 块可以有各自的 pattern 。因此就需要明白（不管是 Nginx 还是你），当 Nginx 收到一个请求时，它是如何去匹配 URI 并找到合适的 Location 的。
要注意的是，写在配置文件中每个 Server 块中的 Location 块的次序是不重要的，Nginx 会按 location modifier 的优先级来依次用 URI 去匹配 pattern ，顺序如下：
    1. =
    2. (None)    如果 pattern 完全匹配 URI（不是只匹配 URI 的头部）
    3. ^~
    4. ~ 或 ~*
    5. (None)    pattern 匹配 URI 的头部
