# RabbitMQ

## 1、基本部署

二进制部署源码下载地址：https://github.com/rabbitmq/rabbitmq-server/releases
<br/>Otp下载地址：https://www.erlang.org/downloads
<br/>erlang版本对应要求：https://www.rabbitmq.com/docs/which-erlang

此次的安装介质如下：
otp_src_27.2.4.tar.gz
rabbitmq-server-generic-unix-4.3.1.tar.xz

```go
yum install gcc gcc-c++ make openssl-devel ncurses-devel tar xz

// 安装erlang环境
tar -zxvf otp_src_27.2.4.tar.gz
cd otp_src_27.2.4
// 指定安装目录
./configure --prefix=/usr/local/erlang27 --without-wx --without-odbc --enable-static
// 多核编译加速
make -j$(nproc)
make install

echo 'export PATH=$PATH:/usr/local/erlang27/bin' >> /etc/profile
source /etc/profile
// 校验版本
erl -version
/usr/local/erlang27/bin/erl -eval 'io:format("OTP Version: ~p~n", [erlang:system_info(otp_release)]).' -s init stop
// 输出必须显示 OTP 27.2.4make
```

```go
// 安装rabbitmq
useradd rabbitmq -s /sbin/nologin -M
mkdir -p /home/rabbitmq && chown rabbitmq:rabbitmq /home/rabbitmq && chmod 700 /home/rabbitmq
xz -d rabbitmq-server-generic-unix-4.3.1.tar.xz
tar -xvf rabbitmq-server-generic-unix-4.3.1.tar -C /usr/local/
mv /usr/local/rabbitmq_server-4.3.1 /usr/local/rabbitmq
chown -R rabbitmq:rabbitmq /usr/local/rabbitmq

echo 'export PATH=$PATH:/usr/local/rabbitmq/sbin' >> /etc/profile
source /etc/profile

mkdir -p /data/rabbitmq/{data,logs}
chown -R rabbitmq:rabbitmq /data/rabbitmq
mkdir -p /usr/local/rabbitmq/etc/rabbitmq
chown rabbitmq:rabbitmq /usr/local/rabbitmq/etc/rabbitmq
---
vi /usr/local/rabbitmq/etc/rabbitmq/rabbitmq-env.conf
# 数据持久目录
RABBITMQ_MNESIA_BASE=/data/rabbitmq/data
# 日志根目录
RABBITMQ_LOG_BASE=/data/rabbitmq/logs
# Erlang虚拟机最大堆内存，根据服务器调整，16G机器设8192m
ERL_ARGS="+M4096m +K true"
# 单机节点名称，主机名替换成你机器名字
RABBITMQ_NODENAME=rabbit@zdxj01-test
# AMQP端口
RABBITMQ_NODE_PORT=35672
# Web管理端口
RABBITMQ_MANAGEMENT_PORT=15672
---
vi /usr/local/rabbitmq/etc/rabbitmq/rabbitmq.conf
# 内存高水位：整机40%内存上限
vm_memory_high_watermark.relative = 0.4
# 内存回落比例
vm_memory_high_watermark_paging_ratio = 0.75
# 磁盘剩余低于5GB阻塞生产者
disk_free_limit.absolute = 40GB
# 全局最大客户端连接
max_connections = 10000
# 消息默认持久化
#default_durable = true
# Web管理开启
management.listener.port = 15672
management.listener.ssl = false
# TCP心跳60秒
heartbeat = 60

```

```go
配置自启动
/etc/systemd/system/rabbitmq.service
---
[Unit]
Description=RabbitMQ 4.3.1 Message Queue
After=network.target

[Service]
User=rabbitmq
Group=rabbitmq
# 补全系统基础PATH，包含/bin /usr/bin（dirname就在这里）+ Erlang + RabbitMQ
Environment=PATH=/usr/local/erlang27/bin:/usr/local/rabbitmq/sbin:/bin:/usr/bin:/usr/sbin
Environment=RABBITMQ_MNESIA_BASE=/data/rabbitmq/data
Environment=RABBITMQ_LOG_BASE=/data/rabbitmq/logs
ExecStart=/usr/local/rabbitmq/sbin/rabbitmq-server
ExecStop=/usr/local/rabbitmq/sbin/rabbitmqctl stop
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
LimitNPROC=65536

[Install]
WantedBy=multi-user.target
```

```go
systemctl daemon-reload
systemctl start rabbitmq
systemctl enable rabbitmq
systemctl status rabbitmq
ss -ltnp | grep beam.smp
```

```go
// 启用web插件
rabbitmq-plugins enable rabbitmq_management
// 创建远程管理账户

rabbitmqctl list_users
// 如有报错需要执行下面的命令：
// 把root的cookie写入rabbitmq家目录
chmod 700 /root/.erlang.cookie
cp /root/.erlang.cookie /home/rabbitmq/.erlang.cookie
// 修改文件归属为rabbitmq运行用户
chown rabbitmq:rabbitmq /home/rabbitmq/.erlang.cookie
// Erlang强制权限只能700
chmod 700 /home/rabbitmq/.erlang.cookie
// 重启 RabbitMQ 加载新 cookie
ls -la /home/rabbitmq/
ls -la /root

systemctl restart rabbitmq
```

```go
// 安全操作：删除默认guest账号
rabbitmqctl delete_user guest
// 账号admin 密码自定义
rabbitmqctl add_user admin Admin@123456
// 赋予管理员角色
rabbitmqctl set_user_tags admin administrator
// 赋予全部vhost权限
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
// 校验运行状态
rabbitmqctl status
// 查看加载的配置
rabbitmqctl environment | grep -E "mnesia|log|vm_memory"
```

