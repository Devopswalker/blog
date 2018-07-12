---
title: 基于 VPS 搭建 Shadowsocks 服务端及客户端配置 
date: 2018-07-13 00:51:12
updated: 2018-07-13 00:55:47
categories: 奇淫技巧
tags: [VPN,VPS,SS,技巧]
photos: 
- http://pbcq37bxu.bkt.clouddn.com/shadowsocks-logo.png
---

### 云服务器选择

> 自用：https://www.vultr.com/ （自选 / 费用适中 / 适合建站 / 日本节点部分IP容易被墙）

> 备胎：https://cn.aliyun.com/  （自选 / 优惠活动 / 容易受国家政策干预）

> 对比：https://www.10besty.com/best-vps-hosting-services/


### 搭建步骤

#### 安装 Python 环境

> 本教程基于 Linux CentOS 7

```
# install in linux centos 7
$ yum install m2crypto python-setuptools

# pip
$ easy_install pip

# install shadowsocks
$ pip install shadowsocks
```
#### 添加 SS 配置
```
# add config file
$ vi /etc/shadowsocks.json
```
> 内容如下：
```
{
    "server":"0.0.0.0",
    "server_port":8888,   // 自定义端口号
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"yourspwd",// 自定义密码
    "timeout":300,
    "method":"aes-256-cfb",// 选择加密方式
    "fast_open": false
}

```
#### 开启防火墙
```
# install
$ yum install firewalld

# start
$ systemctl start firewalld
```
#### 防火墙监听端口
```
# server_port
$ firewall-cmd --permanent --zone=public --add-port=server_port/tcp
$ firewall-cmd --reload
```
#### 启动服务
```
# start server
$ ssserver -c /etc/shadowsocks.json

# set nohup
$ nohup ssserver -c /etc/shadowsocks.json
```

#### 开机启动
```
# add command to /etc/rc.local
$ vi /etc/rc.local
```
> 内容如下：
```
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local
/usr/bin/ssserver -c /etc/shadowsocks.json -d start 
```

### 客户端使用
> MacOS 软件下载地址：[ShadowsocksX-NG](https://pan.baidu.com/s/17B8DmYY5SvUWnlMU8ZOLlQ) 密码：gunx

> 手机 App Store 搜索 Shadowing 有惊喜

> [SS客户端各个版本下载地址](https://help.fyvps.com/index.php/archives/51/)

#### 安装和配置
- [x] 安装：过程略过，绿色软件解压缩打开即用。
- [x] 配置：电脑设置如下图所示

![image](http://pbcq37bxu.bkt.clouddn.com/ss-server-config.jpg)
- [x] 使用：轻触打开 Shadowsocks 开始墙外看世界吧！

### 注意事项

> 1. 遵纪守法，非礼勿视；
> 2. 严禁商用，后果自负；

