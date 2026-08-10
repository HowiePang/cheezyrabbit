# Nacos

配置中心+注册中心 nacos部署

Nacos部署（配置中心+注册中心）

笔者环境为 Rocky9.8，部署的 Nacos 版本为 v3.2.3，==jdk 环境为 v17==。

下载地址：https://nacos.io/download/nacos-server/

二进制下载链接：https://download.nacos.io/nacos-server/nacos-server-3.2.3.zip

官方文档：https://nacos.io/docs/latest/overview

```go
yum install java-17 java-17-openjdk-devel

// 数据库创建
CREATE DATABASE `nacos_config` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'nacos'@'%' IDENTIFIED BY 'Nacosxxxxx';
GRANT ALL PRIVILEGES ON nacos_config.* TO 'nacos'@'%';
FLUSH PRIVILEGES;
SHOW GRANTS FOR 'nacos'@'%';
use nacos_config
source /data/nacos/conf/mysql-schema.sql
```

```go
cd /data ; unzip ./nacos-server-3.2.3.zip ; cd nacos
vim conf/application.properties
------------
# 配置文件，选用外部mysql储存
server.port=8848
nacos.inetutils.ip-address=xxx.xxx.xxx.xxx
spring.sql.init.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?
# 注意 allowPublicKeyRetrieval=true 要开启，不然启动会报错
characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true&useSSL=false
db.user.0=nacos
db.password.0=xxxxxxxx
# 鉴权认证
nacos.core.auth.enabled=true
nacos.core.auth.server.identity.key=nacos_wd_test
nacos.core.auth.server.identity.value=nacos_wd_test
nacos.core.auth.plugin.nacos.token.secret.key=dWVia2pxd2JlZmtqaHdldmZoamt3ZWJmaGp2cXdqa2h2ZGhoeXZraGp2YmpraGJ1YmhiSEpWSkhW
#开口面板
nacos.console.ui.enabled=true
------------
单机模式启动：sh startup.sh -m standalone
单机模式关闭：sh shutdown.sh
默认控制台端口为 8080
```

