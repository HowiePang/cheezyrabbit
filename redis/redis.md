# Redis


## 1、keyDB部署

<font color="red">待更新…</font>

## 2、redis扩容

假设现有一套redis集群环境（5主5从），需要向集群中新添加一台redis6（主+从），组成6节点；

查看当前集群槽位情况：

```go
//登录redis
docker exec -it kp_redis_master redis-cli -h redis1 -p 16379 -a 'KPPass821' -c
//查看集群情况
cluster nodes
```

添加新节点（主）：

```go
//新节点为ip为172.19.121.137，端口号16379，现有集群的ip和端口为172.19.120.98:16379，0bbd99xxxxxx为现有主节点的ID
docker exec -it kp_redis_master redis-cli --cluster add-node 172.19.121.137:16379  172.19.120.98:16379 --cluster-master-id 0bbd99d0e864a606db40cf6e3d684155a753bb08 -a 'KPPass821'
```

移动槽位：

```go
//redis的最大槽位为16384
//组6个节点，则每个节点的槽位为：16384/6=2731

1）从redis1移动xxx到redis6
docker exec -it kp_redis_master redis-cli \
--cluster reshard 172.19.120.98:16379 \
--cluster-from 0bbd99d0e864a606db40cf6e3d684155a753bb08   \
--cluster-to  8a9f9ec74d39e8804b2bc525195443518b2dfc26 \
--cluster-slots xxx \
-a 'KPPass821'
2）从redis2移动xxx到redis6
docker exec -it kp_redis_master redis-cli \
--cluster reshard 172.19.120.99:16379 \
--cluster-from 502ea1ac84adcb93c6ced49d44de35251b4c1ef2   \
--cluster-to  8a9f9ec74d39e8804b2bc525195443518b2dfc26 \
--cluster-slots 278 \
-a 'KPPass821'
3）以下例同...
```

检查槽位分配情况：

```go
docker exec -it kp_redis_master redis-cli --cluster check  172.19.121.137:16379 -a 'KPPass821'
```

添加从节点：

```go
//把172.19.121.137的从节点挂到redis2的主节点上
docker exec -it kp_redis_master redis-cli \
--cluster add-node 172.19.121.137:16380 172.19.121.40:16379  \
--cluster-slave --cluster-master-id 61eb0be35fc66a4d8b49ef8ca8b327d6aa49bd97 \
-a 'KPPass821'
```

如果需要更换某个从节点挂载的主机点：

```go
//进到该从节点中，执行以下命令
cluster replicate <主节点id>
```

## 3、redis编译部署

编译部署redis，os为Rocky9.7，redis版本为8.6.3

源码下载地址：https://download.redis.io/releases/

```go
cd redis-8.6.3/
yum install -y epel-release
yum groupinstall -y "Development Tools"
yum install -y jemalloc-devel openssl-devel tcl

make -j$(nproc) BUILD_TLS=yes
make install
redis-server --version
redis-cli ping

# 建目录
mkdir -p /etc/redis /data/redis /data/redis_log

# 复制默认配置
cp redis.conf /etc/redis/
# 复制默认配置
cp redis.conf /etc/redis/
// 根据实际要求修改配置文件 /etc/redis/redis.conf
// 建议配置如下
bind 0.0.0.0             # 先本地，后面需要外网再改
port 6379
daemonize yes
pidfile /run/redis_6379.pid
logfile /data/redis_log/redis.log
dir /data/redis
# 例如 8G 机器：maxmemory 6gb
maxmemory 6gb
###########################
# 4. 持久化（RDB+AOF 双保险）
###########################
# RDB 快照
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes

# AOF 日志（更安全）
appendonly yes
appendfsync everysec
no-appendfsync-on-rewrite yes
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-use-rdb-preamble yes   # 混合持久化（Redis 4.0+）
maxclients 10000

requirepass YourStrongPass123   # 生产一定要设密码
```

```go
//设置开机启动
vi /etc/systemd/system/redis.service
[Unit]
[Unit]
Description=Redis Server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
#PIDFile=/var/run/redis_6379.pid
Restart=always

[Install]
WantedBy=multi-user.target

// 添加优化参数(新版本优化要求)
echo 'vm.overcommit_memory = 1' >> /etc/sysctl.conf
sysctl -p

// 启动
systemctl daemon-reload
systemctl start redis
systemctl enable redis
systemctl status redis
```



