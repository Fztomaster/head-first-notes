## Nginx

### 概述

​	nginx是轻量级的web服务器/反向代理服务器，俄罗斯-伊戈尔·赛索耶夫 开发，2004年10月4号
作用：
	Web服务器部署静态web应用
	负载均衡服务器实现反向代理
	邮件服务器实现收发邮件

### Linux安装nginx

​	安装gcc命令：yum -y install gcc-c++
	安装nginx依赖：
		yum install pcre pcredevel
		yum install zlib zlibdevel
		yum install openssl openssldevel
	在nginx解压目录配置nginx编译安装环境：
		./configure
		make
		make install
	开发80端口：
		/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT

### 负载均衡
 	高并发请求时，使用服务器集群，对外访问路径只能有一个，把用户请求分配到压力较轻的服务器，实现各服务器的压力是均衡的

### 正向代理和反向代理

​	正向代理：客户端代理，我们访问谷歌，访问不了因为有防火墙，可以通过香港服务器(代理服务器)帮我们把请求转达给谷歌，然后谷歌响应给香港服务器，再响应给我们。
	反向代理：服务器端代理，10台服务器集群，共同提供一个	web应用服务，为了实现负载均衡或者安全，为这10台服务器提供一个代理服务器，客户端访问web应用时实际上访问的是代理服务器，由代理服务器把请求转发给某一个服务器处理，然后再由代理服务器把响应结果响应给客户端。
### ngnix的目录结构

​	conf：存放配置文件，主要是nginx.conf
	nginx.exe：nginx启动程序

### ngnix基本操作

​	启动：start nginx.exe
	访问：http://localhost
	重新加载：nginx -s reload
	关闭：nginx -s stop