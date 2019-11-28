高可用高并发架构
=========
## 说明
     高可用高并发架构学习，笔记和源码持续更新
## 目录
* [高可用高并发架构](#高可用高并发架构)
	* [一、高可用](#一高可用)
		* [1、负载均衡与反向代理](#1负载均衡与反向代理)
		* [2、隔离术](#2隔离术)
		* [3、限流](#3限流)
		* [4、降级](#4降级)
		* [5、超时与重试](#5超时与重试)
		* [6、回滚](#6回滚)
		* [7、压测预案](#7压测预案)
	* [二、高并发](#二高并发)
		* [1、缓存](#1缓存)	
		* [2、连接池线程池](#2连接池线程池)
		* [3、异步并发](#3异步并发)
		* [4、扩容](#4扩容)
		* [5、队列](#5队列)
	
### 一、高可用
### 1、负载均衡与反向代理
- 负载均衡的作用<br>
1）、转发功能：按照一定的算法【权重、轮询】，将客户端请求转发到不同应用服务器上，减轻单个服务器压力，提高系统并发量。<br>
2）、故障移除：通过心跳检测的方式，判断应用服务器当前是否可以正常工作，如果服务器期宕掉，自动将请求发送到其他应用服务器。<br>
3）、恢复添加：如检测到发生故障的应用服务器恢复工作，自动将其添加到处理用户请求队伍中。<br>
- 外网DNS应该用来实现用GSLB（全局负载均衡）进行流量调度，如将用户分配到离他最近的服务器上以提升体验
- 对于内网DNS，可以实现简单的轮询负载均衡，但会有一定的缓存时间并且没有失败重试机制，我们可以考虑选择如HaProxy和Nginx
- Nginx提供了HTTP（ngx_http_upstreamm_module）七层负载均衡，TCP（ngx_stream_upstream_module）四层负载均衡<br>
A.upstream配置：<br>
1.在http指令下配置upstream即可<br>
2.主要配置：<br>
IP地址和端口<br>
权重：默认是1，越高分配给这台服务器的请求就越多<br>
B.负载均衡算法<br>
1.用来解决用户请求到来时如何选择upstream server进行处理，默认采用的是round-robin(轮询)，同时支持ip_hash<br>
2.hash key [consistent]，对某个key进行哈希或者使用一致性哈希算法进行负载均衡，建议考虑使用一致性哈希算法<br>
3.least_conn，将请求均衡到最少活跃连接的上游服务器<br>
C.失败重试<br>
1.主要两部分配置：<br>
upstream server：server xxxx:80 max_fails=2  fail_timeout=10s weight=1;...
proxy_pass：配置proxy_next_upstream相关配置，proxy_next_upstream、proxy_next_upstream_timeout、proxy_next_upstream_tries
D.健康检查<br>
1.nginx对上游服务器的健康检查默认采用的是惰性策略<br>
2.TCP心跳检查：check interval=3000 rise=1 fall=3 timeout=2000 type=tcp;<br>
3.HTTP心跳检查：<br>
check_http_send "HEAD /status HTTP/1.0\r\n\r\n";
check_http_expect_alive http_2xx http_3xx;
3.使用的是opensresty模块，安装nginx之前需要先打nginx_upstream_check_module补丁<br>
E.其他配置<br>
1.备份上游服务器，backup<br>
2.不可用上游服务器，down<br>
F.长连接<br>
1.可以通过keepalive指令配置长连接数量<br>
G.HTTP反向代理<br>
1.反向代理除了实现负载均衡之外，还提供如缓存来减少上游服务器的压力<br>
2.还可以开启gzip压缩，减少网络传输的数据包大小<br>
H.HTTP动态负载均衡<br>
1.Consul是一款开源的分布式服务注册与发瑞系统，通过HTTP API可以使得服务注册、发现实现起来非常简单<br>
- Nginx静态负载均衡<br>
启用ngx_stream_core_module，安装Nginx时，添加--with-stream<br>
配置在stream指令下<br>
可配置数据库连接<br>
- Nginx动态负载均衡<br>
nginx-upsync-module，提升了HTTP七层动态负载均衡，动态更新上游服务器不需要reload nginx
### 2、隔离术
- 
### 3、限流
- 
### 4、降级
- 
### 5、超时与重试
- 
### 6、回滚
- 
### 7、压测预案
- 
### 二、高并发
### 1、缓存