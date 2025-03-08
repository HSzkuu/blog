---
title: 联通创维 SK-D648-C 光猫开启 Telnet 并登录
published: 2025-03-08
description: ''
image: ''
tags: [光猫, 宽带, EPON]
category: '网络'
draft: false 
lang: ''
---
## 开启 Telnet
1. 连接到光猫，并能访问 `192.168.1.1`
2. 浏览器进入 `http://192.168.1.1/hidden_version_switch.html`
3. 点上 Telnet Enable 的勾，然后会刷新页面，勾会消失，但是不用管
![Telnet Enable](/sk-d648-c/telnet-enable.png)
这个方法的 Telnet 开启是暂时的  
## 登录 Telnet
下载 FileZilla，输入主机 `192.168.1.1` 用户名与密码都是 `useradmin`  
进入 `/var/tmp` 找到 `telnet_su_passwd` 下载下来  
用记事本或者什么打开 这串密码要用  

打开 Telnet 工具，我这里用的是 Windows 自带的  
命令提示符输入 `telnet 192.168.1.1`  
出现 `Login:` 后输入 `user`  
然后出现 `Password:` 后复制粘贴输入之前 `telnet_su_passwd` 里的密码  
现在就登入了 Telnet  
可以进一步提权：输入 `su`  
也会出现 `Password:` 还是之前的密码复制粘贴进去就可以了

## 查看信息

`sismac show`  
剩下的不会了