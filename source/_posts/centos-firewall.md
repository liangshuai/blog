title: "CentOS 7开放端口和关闭防火墙"
date: 2015-12-29 15:09:32
tags:
- CentOS
- Vagrant
- Firewall
---

想用CentOS + Vagrant搭建一个开发环境，搭建之后发现一直无法访问虚拟机中的服务，起初以为是Vagrant配置的问题，尝试修改配置，可是发现都不行，甚至连端口转发的方式都无法访问，于是猜测是因为CentOS7防火墙的关系，试了试打开端口和关闭防火墙，发现都可以访问到了。



<!-- more -->

```sh
#vagrant 配置
config.vm.network "forwarded_port", guest: 3000, host: 3000
config.vm.network "private_network", ip: "192.168.33.10"
# config.vm.network "public_network"
```

### 开放端口

永久的开放需要的端口

```sh
sudo firewall-cmd --zone=public --add-port=3000/tcp --permanent
sudo firewall-cmd --reload
```
之后检查新的Rule

```sh
firewall-cmd --list-all
```

### 关闭防火墙

由于只是用于开发环境，所以打算把防火墙关闭掉
```sh
//临时关闭防火墙,重启后会重新自动打开
systemctl restart firewalld
//检查防火墙状态
firewall-cmd --state
firewall-cmd --list-all

//Disable firewall

systemctl disable firewalld
systemctl stop firewalld
systemctl status firewalld

//Enable firewall

systemctl enable firewalld
systemctl start firewalld
systemctl status firewalld
```