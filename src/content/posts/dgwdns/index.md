---
title: '双网关双dns'
description: '双网关双dns，进行内外网各走一套'
pubDate: 2024-10-11
cover: "network-services.jpeg"
tags:
  - 网络
  - 路由器
  - DNS
  - Gateway
  - Network-Services
---

本文只是作为个人记录

up: 2025-03-05 去掉adg，改为主路由+[ppdns](https://github.com/kkkgo/PaoPaoDNS)+[ppgw](https://github.com/kkkgo/PaoPaoGateWay)   

ppdns改为国内国外分流，国外域名跳过代理使用force_dnscrypt_list.txt控制

paopao-dns compose.yml
```yml

services:
  paopaodns:
    image: sliamb/paopaodns:latest
    #image: public.ecr.aws/sliamb/paopaodns:latest
    container_name: PaoPaoDNS
    #network_mode: "host"
    restart: always
    volumes:
      - ./data:/data
    environment:
      - TZ=Asia/Shanghai
      - UPDATE=weekly
      - DNS_SERVERNAME=PaoPaoDNS,blog.03k.org
      - DNSPORT=53
      - CNAUTO=yes
      - CNFALL=yes
      - CN_TRACKER=yes
      - USE_HOSTS=no
      - IPV6=yes
      - SOCKS5=192.168.123.3:1080
      - SERVER_IP=192.168.123.2
      - CUSTOM_FORWARD=192.168.123.3:53
      - AUTO_FORWARD=yes
      - AUTO_FORWARD_CHECK=yes
      - USE_MARK_DATA=yes
      - HTTP_FILE=yes
    ports:
      - "53:53/udp"
      - "53:53/tcp"
      - "5304:5304/udp"
      - "5304:5304/tcp"
      - "7889:7889/tcp"
```
paopao-gateway ppgw.ini
```ini
#paopao-gateway

# mode=socks5|ovpn|yaml|suburl|free
# default: free
mode=suburl

# Set fakeip's CIDR here
# default: fake_cidr=7.0.0.0/8
fake_cidr=7.0.0.0/8

# Set your trusted DNS here
# default: dns_ip=1.0.0.1
dns_ip=192.168.123.2
# default: dns_port=53
# If used with PaoPaoDNS, you can set the 5304 port
dns_port=5304

# Clash's web dashboard
clash_web_port="80"
clash_web_password="123456"

# default penport=no
# socks+http mixed 1080
openport=yes

# default: udp_enable=no
udp_enable=no

# sniff
sniff=yes

# default:30
sleeptime=30

# socks5 mode settting
# default: socks5_ip=gatewayIP
socks5_ip="192.168.123.3"
# default: socks5_port="7890"
socks5_port="1080"

# ovpn mode settting
# The ovpn file in the same directory as the ppgw.ini.
# default: ovpnfile=custom.ovpn
ovpnfile="custom.ovpn"
ovpn_username=""
ovpn_password=""

# yaml mode settting
# The yaml file in the same directory as the ppgw.ini.
# default: yamlfile=custom.yaml
yamlfile="custom.yaml"

# suburl mode settting
suburl="https://subscribe?token=69"
subtime=1d

# fast_node=check/yes/no
fast_node=yes
test_node_url="https://www.youtube.com/generate_204"
ext_node="Traffic|Expire| GB|Days|Date"
cpudelay="3000"
# dns burn setting
# depend on fast_node=yes & mode=suburl/yaml
dns_burn=no
# If used with PaoPaoDNS, you can set the PaoPaoDNS:53
ex_dns="192.168.123.2:53"

# Network traffic records
net_rec=yes
max_rec=5000
```

主路由添加：路由 7.0.0.0/8 下一跳192.168.123.3（ppgw）

LAN-DHCP服务器-高级设置-DHCP选项
添加：

tag:q,6,192.168.123.2,192.168.123.1

注：为包含q标签客户端指定dns1和dns2，

123.2（ppdns），123.1为本地dns，即宽带dns也防止客户端自动添加dns2


采用opwrt主路由+paopaogateway网关+paopaodns外网dns+adgh内网dns等进行默认设备常规上网，指定mac设备自由上网

## 架构

### opwrt主路由:

- IP:192.168.1.1
- 网关:192.168.1.1
- DNS:192.168.1.1

### paopaogateway网关 简称PPGW

- IP:192.168.1.2
- 网关:192.168.1.1
- DNS:192.168.1.1

### paopaodns DNS 简称PPDNS

- IP:192.168.1.3
- 网关:192.168.1.1
- DNS:192.168.1.1

### adgh DNS

- IP:192.168.1.1:3000

通过劫持opwrt主路由dns:53端口作为国内dns

## opwrt部分：

### 静态路由设置

网络>路由>静态 IPv4 路由

添加>静态 IPv4 路由：

目标：7.0.0.0/8（ppgw.ini配置文件fake_cidr=7.0.0.0/8）

网关：192.168.1.2（PPGW IP）

添加静态 IPv6 路由：

由ipv4 192.168.1.2 转换 ::ffff:c0a8:0102

### 通过MAC下发网关和固定IP

网络>接口>DHCP/DNS>静态地址分配

- 添加
    - 选择MAC
    - IPv4地址
    - 标签（输入fq）


网络>接口>lan

DHCP服务器>高级设置>DHCP高级选项

- 添加
    - 6,192.168.1.1 #默认网关
    - tag:fq,6,192.168.1.3 #PPDNS IP


### adgh劫持53端口接管常规上网设备的DNS

网络>接口>DHCP/DNS>设置及端口

DNS 服务器端口 输入 0

## PPDNS 部分

官方推荐docker-compose构建

docker-compose环境

ssh连接输入

```shell
mkdir ppdns
cd ppdns
vim docker-compose.yml
docker-compose up -d
```

docker-compose.yml 文件内容：

```yml
services:
  paopaodns:
    image: sliamb/paopaodns:latest
    container_name: PaoPaoDNS
    restart: always
    volumes:
      - /home/paopaodns:/data
    environment:
      - TZ=Asia/Shanghai
      - UPDATE=weekly
      - DNS_SERVERNAME=PaoPaoDNS,blog.03k.org
      - DNSPORT=53
      - CNAUTO=yes
      - CNFALL=yes
      - CN_TRACKER=yes
      - USE_HOSTS=no
      - IPV6=no
      - SOCKS5=192.168.1.2:1080
      - SERVER_IP=192.168.1.3
      - CUSTOM_FORWARD=192.168.1.2:53
      - AUTO_FORWARD=yes
      - AUTO_FORWARD_CHECK=yes
      - USE_MARK_DATA=yes
      - HTTP_FILE=yes
    ports:
      - "53:53/udp"
      - "53:53/tcp"
      - "5304:5304/udp"
      - "5304:5304/tcp"
      - "7889:7889/tcp"
```
目录 /home/paopaodns/ppgw.ini 配置文件：

```yml
#paopao-gateway

# mode=socks5|ovpn|yaml|suburl|free
# default: free
mode=yaml

# Set fakeip's CIDR here
# default: fake_cidr=7.0.0.0/8
fake_cidr=7.0.0.0/8

# Set your trusted DNS here
# default: dns_ip=1.0.0.1
dns_ip=192.168.1.3
# default: dns_port=53
# If used with PaoPaoDNS, you can set the 5304 port
dns_port=5304

# Clash's web dashboard
clash_web_port="80"
clash_web_password="password"

# default：openport=no
# socks+http mixed 1080
openport=no

# default: udp_enable=no
udp_enable=no

# default:30
sleeptime=30

# socks5 mode settting
# default: socks5_ip=gatewayIP
socks5_ip="192.168.1.2"
# default: socks5_port="7890"
socks5_port="7890"

# ovpn mode settting
# The ovpn file in the same directory as the ppgw.ini.
# default: ovpnfile=custom.ovpn
ovpnfile="custom.ovpn"
ovpn_username=""
ovpn_password=""

# yaml mode settting
# The yaml file in the same directory as the ppgw.ini.
# default: yamlfile=custom.yaml
yamlfile="custom.yaml"

# suburl mode settting
suburl=""
subtime=1d

# fast_node=check/yes/no
fast_node=no
test_node_url="https://www.youtube.com/generate_204"
ext_node="Traffic|Expire| GB|Days|Date"
cpudelay="3000"
fall_direct="no"
# dns burn setting
# depend on fast_node=yes & mode=suburl/yaml
dns_burn=no
# If used with PaoPaoDNS, you can set the PaoPaoDNS:53
ex_dns="192.168.1.3:53"

# Network traffic records
net_rec=yes
max_rec=5000
```

 目录/home/paopaodns/custom.yaml 配置文件：

```yml
proxy-providers:
    provider1:
        type: http
        url: "机场url"
        interval: 172800
        path: ./provider1.yaml
        health-check:
            enable: true
            url: http://www.gstatic.com/generate_204
            interval: 300

proxy-groups:
  - name: PROXY
    type: select
    proxies:
      - Manual-Select
      - Auto-Select
      - DIRECT
  - name: Manual-Select
    type: select
    use:
      - provider1
  - name: Auto-Select
    type: url-test
    tolerance: 10
    use:
      - provider1

rule-providers:
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400

  icloud:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/icloud.txt"
    path: ./ruleset/icloud.yaml
    interval: 86400

  apple:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/apple.txt"
    path: ./ruleset/apple.yaml
    interval: 86400

  google:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/google.txt"
    path: ./ruleset/google.yaml
    interval: 86400

  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    path: ./ruleset/proxy.yaml
    interval: 86400

  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    path: ./ruleset/direct.yaml
    interval: 86400

  private:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
    path: ./ruleset/private.yaml
    interval: 86400

  gfw:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/gfw.txt"
    path: ./ruleset/gfw.yaml
    interval: 86400

  tld-not-cn:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/tld-not-cn.txt"
    path: ./ruleset/tld-not-cn.yaml
    interval: 86400

  telegramcidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/telegramcidr.txt"
    path: ./ruleset/telegramcidr.yaml
    interval: 86400

  cncidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/cncidr.txt"
    path: ./ruleset/cncidr.yaml
    interval: 86400

  lancidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/lancidr.txt"
    path: ./ruleset/lancidr.yaml
    interval: 86400

  applications:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/applications.txt"
    path: ./ruleset/applications.yaml
    interval: 86400
    
rules:
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,tld-not-cn,PROXY
  - RULE-SET,gfw,PROXY
  - RULE-SET,telegramcidr,PROXY
  - MATCH,DIRECT
```

## PPGW 部分

在PPDNS主机上继续生成PPGW ISO镜像
  
```shell
mkdir ppgwios
cd ppgwios
docker pull sliamb/ppgwiso
```
下载mihomo替换默认clash核心
https://github.com/MetaCubeX/mihomo/releases

最后打包iso

```shell
docker run --rm -v .:/data sliamb/ppgwiso
```
虚拟机加载iso启动完成

我自己用PVE虚拟机 上传iso，光驱指定iso文件启动，启动自动（30秒间隔）搜索并加载PPDNS目录下的ppgw.ini配置文件，mihomo则加载custom.yaml文件。
