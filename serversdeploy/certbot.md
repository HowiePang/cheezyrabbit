# certbot

[TOC]

开源自动化签发域名证书工具

官网1: https://letsencrypt.org/
官网2: https://certbot.eff.org/
社区地址：https://github.com/certbot/certbot

## 1、编译安装

快速安装可以选择在线源的方式，如yum、apt等，我们此处用源码编译的方式安装，可以更灵活的选择版本和其他等。

此处我们以v5.6版本为例：https://github.com/certbot/certbot/archive/refs/tags/v5.6.0.tar.gz

笔者环境为容器环境，Rocky9.7，默认的python版本为3.9，此版本需要python3.10以上，所以先要进行py版本升级。

```go
// 将默认的py3.9升级到3.10及以上。
// Rocky9 系统自带 Python3.9 是系统底层依赖（dnf/yum 依赖它），不能删除 / 替换默认 python3，采用并行安装 Python3.10，单独给 Certbot 虚拟环境使用
yum groupinstall "Development Tools"
yum install wget tar gcc gcc-c++ openssl-devel libffi-devel
yum install libffi-devel openssl-devel bzip2-devel
yum install ncurses-devel sqlite-devel readline-devel tk-devel xz-devel libuuid-devel
yum install gdbm-devel

cd /usr/src
sudo wget https://www.python.org/ftp/python/3.10.15/Python-3.10.15.tgz
sudo tar -zxvf Python-3.10.15.tgz
cd Python-3.10.15

// 配置编译（安装到独立目录，不和系统Python冲突）
./configure --prefix=/usr/local/python3.10 --enable-optimizations
// 编译安装（-j 后面填CPU核心数，比如4核写-j4，单核直接去掉-j）
// altinstall 关键：不会覆盖系统 python3 软链接，避免系统命令崩溃
make -j$(nproc)
sudo make altinstall

// 验证 Python3.10 是否安装成功
// 正常输出版本号即安装完成
/usr/local/python3.10/bin/python3.10 --version
/usr/local/python3.10/bin/pip3.10 --version
```

```go
// Certbot 源码目录，重建适配 Python3.10 的虚拟环境
mkdir -p /opt/certbot
tar -zxvf certbot-5.6.0.tar.gz
cd certbot-5.6.0

// 创建名为 venv 的独立虚拟环境
/usr/local/python3.10/bin/python3.10 -m venv venv
// 激活进入虚拟环境（终端前缀会出现 (venv) 标识）
source venv/bin/activate
cd certbot
pip install ./
// 验证成功会出现版本号码
certbot --version
```

```go
// 全局软链接（任意位置直接调用 certbot）
deactivate
sudo ln -s /opt/certbot/certbot-5.6.0/venv/bin/certbot /usr/local/bin/certbot
certbot --version
```

## 2、基本用法

1、仅申请证书，standalone 模式（本机 80 端口空闲时用）

```go
certbot certonly --standalone \
-d demo.demo.com \
--email admin@demo.com \
--agree-tos \
--non-interactive
```

申请成功后，证书路径如下：

```go
/etc/letsencrypt/live/demo.demo.com/
├── privkey.pem   # 私钥（保密）
├── fullchain.pem # 完整证书链（绝大多数程序首选）
├── cert.pem      # 站点证书
└── chain.pem     # 中间证书
```

2、测试续期是否正常（不会实际更新证书）

```go
certbot renew --dry-run
```

3、检验证书有效期

```go
// 查看本机所有证书详情+到期时间
certbot certificates
```

