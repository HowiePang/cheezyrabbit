# 包源

记录一下用yum或者apt安装服务的一些技巧

[TOC]

**yumdownloader命令**
该命令用于仅下载软件包而不进行安装，可以适用离线rpm安装的场景：即先在能通公网的环境中下载拿到包，最后放到离线环境中rpm安装；

```go
sudo yum install yum-utils
sudo yumdownloader package-name                       //默认把RPM包下载到当前目录下
sudo yumdownloader --destdir=/path/to/directory package-name      //指定下载目录
sudo rpm -ivh package-name           //放到离线环境下安装
```

## 1、在线 apt 源

```go
ubuntu22.04
----------------
deb http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ jammy-backports main restricted universe multiverse

ubuntu20.04
-----------------
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
##測試版源
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# 源碼
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
##測試版源
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
```

## 2、在线 yum 源

```go
//阿里云在线yum源
cat > /etc/yum.repos.d/CentOS-Aliyun.repo <<EOF
# CentOS Base OS
[alilinux]
name=Alilinux Base
baseurl=http://mirrors.aliyun.com/centos/7/os/x86_64/
gpgcheck=0
enabled=1
metadata_expire=1800

# CentOS Extras
[alilinux-extras]
name=Alilinux Extras
baseurl=http://mirrors.aliyun.com/centos/7/extras/x86_64/
gpgcheck=0
enabled=1
metadata_expire=1800

# CentOS Updates
[alilinux-updates]
name=Alilinux Updates
baseurl=http://mirrors.aliyun.com/centos/7/updates/x86_64/
gpgcheck=0
enabled=1
metadata_expire=1800

//添加完之后更新repolist
yum clean all
yum makecache
yum install epel-release
```

