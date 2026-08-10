# 简单ss代理

[TOC]

介绍代理网络环境搭建（原生ip+双isp），以美区为例。

大概有两种方案：

方案1、HK服务器 ------》 美区socks5服务，直接从HK出口直连到最终的美区socks5，简单，费用上面稍便宜，但是安全性不足，tk直播会有点卡；

方案2、HK服务器 ------》 美区LA服务器中转 -------》美区socks5服务，中间加一层LA中转，这种方案最终出口LA地区，安全性上面稍佳，较稳定，但是费用上面需要额外支付服务器，配置上面也稍微麻烦点。

我们此处采用方案2进行部署。

\-----------------------------------------------------

我们需要提前购买好：HK、LA和美区socks5端口。

HK和LA的机器上都部署好ss服务，此处不做多余介绍可看相关文章，部署完之后，进行一些优化配置如下：

```go
// cat  /etc/sysctl.conf
fs.file-max = 51200
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1

//执行命令
sysctl -p


// tail  /etc/security/limits.conf
# End of file
* soft nofile 51200
* hard nofile 51200

# for server running in root:
root soft nofile 51200
root hard nofile 51200

//执行命令
ulimit -n 51200
```

然后需要在HK机器上部署socat服务，进行端口映射：

```go
yum install socat

// 启动socat端口为48990，远程LA的ss的端口为48989
cat /etc/systemd/system/socat-forward.service
[Unit]
Description=Socat Port Forward to US Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/socat TCP-LISTEN:48990,fork,reuseaddr TCP:xxx.xxx.xxx.xxx:48989
Restart=always
RestartSec=5
User=root
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl enable --now socat-forward
```

最好规划好LA的防火墙，只允许HK的ip连入48989端口，提高安全性，防火墙配置语法规则如下：

```go
//安装 iptables
dnf install -y iptables-services

//清空规则
iptables -F
iptables -X

//先设置默认策略为 ACCEPT（临时保命）
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

//添加规则
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 48989 -s HK的ip地址 -j ACCEPT
iptables -A INPUT -s HK的ip地址 -p icmp -j ACCEPT

//设置默认策略
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

//保存
iptables-save > /etc/sysconfig/iptables
systemctl enable --now iptables

//附加删除防火墙规则语法
# sudo iptables -L INPUT -n --line-numbers
# sudo iptables -D INPUT 7
```

上面作为之后，接下来就用小火箭客户端工具连接，配置好socks5的配置，再配置好HK的连接配置（48990端口），把socks5的配置进行 “proxy pass” 到 hk 的配置上，开启socks5连接就能访问到美区 ，且出口显示为家庭带宽网络。

## 附加

在非ios或者mac系统的客户端上，因为没有小火箭，所有socks5的代理转发非常不便，这时需要在服务器上面安装一个gost作为链式转发，以便win或者安卓上面也能达到小火箭的功能。

以上面方案1为例，需要在HK上面部署 gost，客户端通过 **ss协议** 连接到HK的特定端口，HK再内部转发到 socks5 服务，其配置文件如下：

<img src="./assets/image-20260810163458008.png" alt="image-20260810163458008" style="zoom:50%;" />

```go
wget https://github.com/ginuerzh/gost/releases/download/v2.11.2/gost-linux-amd64-2.11.2.gz
gunzip gost-linux-amd64-2.11.2.gz
sudo mv gost-linux-amd64-2.11.2 /usr/local/bin/gost
sudo chmod +x /usr/local/bin/gost

systemctl cat gost
// cat /etc/systemd/system/gost.service
[Unit]
Description=gost proxy chain
After=network.target

[Service]
ExecStart=/usr/local/bin/gost -C /etc/gost/config.json
Restart=always
User=root

[Install]
WantedBy=multi-user.target

cat /etc/gost/config.json
{
  "ServeNodes": [
    "ss://aes-256-gcm:MySecurefgPass782025@0.0.0.0:48991"
  ],
  "ChainNodes": [
    "socks5://mfQwa9NEkWJL4DS:CNXcgbiW1QPd7Yc@xxx.xxx.xxx.xxx:48652"
  ],
  "Routes": [
    {
      "ChainNodes": [
        "socks5://mfQwa9NEkWJL4DS:CNXcgbiW1QPd7Yc@xxx.xxx.xxx.xxx:48652"
      ]
    }
  ]
}

systemctl daemon-reload
systemctl enable --now gost
netstat -nap | grep 48991
```

然后就用支持ss协议的客户端连接到HK的对应端口上，就行连接到美区的socks5服务器了，这就是解决非ios或者mac系统客户端连接不便的方案之一，也是最简单的方案。