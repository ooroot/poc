---
title: 'OpenWrt安装Nginx并绑定域名访问NAS'
description: 'OpenWrt上面安装Nginx并通过ddns-go把变化ipv6和Cloudflare上的域名绑定，从而公网访问NAS，路由器等内网穿透服务。'
pubDate: 2025-08-22
author: Wu
cover: assets/cover.png
recommend: true
tags: ['OpenWrt', 'NAS', 'Nginx', 'DDNS-go', 'SSL']
postType: coverSplit
---

本文只是作为个人记录，不太记录常规细节


## DDNS-go 部分

我是的是 immortalwrt 系统>软件包>更新列表后搜索ddns-go后安装即可

我的域名在Cloudflare上，后台选择Cloudflare并填写Token（Cloudflare可以申请，顺便把ssl证书也一起申请了）,IPv6勾选启用，通过网卡获取ipv6或者其他方式，然后填写入要绑定的域名（例如：nas.xxxx.com），输入登录ddns-go后台的账户密码保存即可。

## Nginx 部分

### 安装

 1. immortalwrt软件包搜索luci-nginx并安装，会自动安装依赖包并启动nginx和禁用uhttpd（原openwrt精简版的web服务）
 
 2. 或者安装ssh用命令安装
 
    2.1. 安装 openssh-sftp-server

    ```shell
    opkg update && openssh-sftp-server # 安装
    echo "PermitRootLogin yes" >> "/etc/ssh/sshd_config" # 允许通过root登录
    ```
    2.2. 安装 nginx

    ```shell
    opkg update && opkg install luci-nginx && opkg install luci-ssl-nginx # 安装 nginx
    ```
### 配置nginx

ssh登录路由器

输入 uci set nginx.global.uci_enable=false 回车

输入 uci commit 回车提交修改 脱离 OpenWrt UCI，方便施展拳脚

输入vi /etc/nginx/nginx.conf

粘贴下面的配置 Esc :wq 保存

```shell

# This file is re-created when Nginx starts.
# Consider using UCI or creating files in /etc/nginx/conf.d/ for configuration.
# Parsing UCI configuration is skipped if uci set nginx.global.uci_enable=false
# For details see: https://openwrt.org/docs/guide-user/services/webserver/nginx
# UCI_CONF_VERSION=1.2

worker_processes auto;

user root;

include module.d/*.module;

events {}

http {
	access_log off;
	log_format openwrt
		'$request_method $scheme://$host$request_uri => $status'
		' (${body_bytes_sent}B in ${request_time}s) <- $http_referer';

	include mime.types;
	default_type application/octet-stream;
	sendfile on;

	client_max_body_size 128M;
    client_header_buffer_size 32k; # 加大缓存
    large_client_header_buffers 4 32k; #加大缓存

	gzip on;
	gzip_vary on;
	gzip_proxied any;

	root /www;

	server { #see uci show 'nginx._lan'
		listen 443 ssl default_server;
		listen [::]:443 ssl default_server;
		server_name _lan; # 改为_lan
		include restrict_locally;
		include conf.d/*.locations;
		ssl_certificate /etc/nginx/conf.d/_lan.crt;
		ssl_certificate_key /etc/nginx/conf.d/_lan.key;
		ssl_session_cache shared:SSL:32k;
		ssl_session_timeout 64m;
		access_log off; # logd openwrt;
	}

	server { #see uci show 'nginx._redirect2ssl'
		listen 80;
		listen [::]:80;
        server_name _lan;
		#server_name _redirect2ssl;# 注释http跳转https
		#return 302 https://$host$request_uri; # 注释http跳转https
        include restrict_locally; # 添加引用外网访问限制
		include conf.d/*.locations; # 添加引入本地资源
        access_log off; # logd openwrt;
	}

	include conf.d/*.conf;
}

```
输入 nginx -t 测试一下配置文件有没有问题（nginx -T详细）

输入 /etc/init.d/nginx reload 重新载入配置

### Cloudflare

Cloudflare 购买域名

域名解析

申请ssl证书

申请域名Token

等....此处省略1万个步骤

### NAS域名反代

把ssl证书和key上传 /etc/nginx/conf.d 其他路径也可以能访问即可

新建配置 vi /etc/nginx/conf.d/nas.conf 粘贴以下配置并保存

```shell

server {
   listen 80;  # 监听的外部端口，与 OpenWrt 端口转发设置一致
   listen [::]:80; # 监听ipv6 80端口
   server_name nas.xxx.com; # 你的域名或公网 IP
   return 302 https://$host$request_uri; # http 302跳转 https 安全呀
}
server {
   listen 443 ssl;
   listen [::]:443 ssl;
   server_name nas.xxx.com; # 你的域名或公网 IP

   ssl_certificate /etc/nginx/conf.d/xxx.pem; # 域名ssl证书，注意路径 相同目录即可
   ssl_certificate_key /etc/nginx/conf.d/xxx.key; # 域名ssl证书key

   location / {
       proxy_pass https://192.168.1.5; # NAS访问ip
       # 以下是增加访问时header的信息
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
   }
}

```

测试 nginx -t

重载 /etc/init.d/nginx reload

### 域名hosts劫持

在内网通过域名访问NAS时劫持域名为内网 

vi /etc/hosts

添加 192.168.1.1 nas.xxx.com 用于将域名解析到IP地址

保存 esc :wq 


## 注意事项

1. 浏览器缓存
2. restrict_locally 注释 #deny all;
