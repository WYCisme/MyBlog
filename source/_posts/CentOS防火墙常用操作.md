---
title: CentOS防火墙常用操作
reward: false
copyright: false
tags:
  - 防火墙
  - CentOS
categories:
  - 学习
date: 2020-09-02 09:34:41
---





## CentOS防火墙常用操作

> 转自：https://www.cnblogs.com/haw2106/p/10730916.html

------

注：CentOS7之前用来管理防火墙的工具是iptable，7之后使用的是Firewall

样例：在CentOS7上安装tomcat后，在linux本机上可以访问tomcat主页，[http://ip:8080](https://link.jianshu.com/?t=http://ip:8080), 但是在其他同网段的机器上却不能访问该地址，原因是因为linux在安装之后默认只开放个别端口供外机访问，这个时候我们只需要将8080端口设置为向外机开放即可。

首先尝试iptables，iptables无效后可尝试防火墙firewalld。

 

方法一、在外部访问CentOS中部署应用时，需要关闭防火墙。

关闭防火墙命令：**systemctl stop firewalld.service**

开启防火墙：**systemctl start firewalld.service**

关闭开机自启动：**systemctl disable firewalld.service**

开启开机启动：**systemctl enable firewalld.service**

 

方法二、CentOS7使用firewall工具管理防火墙，代替了原来的iptables

操作步骤如下：

```shell
1 #查看防火墙状态=》使用root的身份=》结果为running
2 firewall-cmd --state
3 #永久性的开放8080端口
4 firewall-cmd --add-port=8080/tcp permanent
5 #重载生效刚才的端口设置
6 firewall-cmd --reload
```

**常用的firewall命令**常用命令介绍

```shell
firewall-cmd --state                           ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
firewall-cmd --get-zones                       ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                 ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent     ##永久添加80端口
firewall-cmd --remove-port=80/tcp --permanent  ##永久移除80端口
firewall-cmd --list-ports                      ##查看已经开放的端口
iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
man firewall-cmd                               ##查看帮助
```

