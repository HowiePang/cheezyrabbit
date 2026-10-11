# xxl-job

```go
// 执行内置的数据库脚本
CREATE DATABASE `nacos_config` CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'xxljob'@'%' IDENTIFIED BY 'OWAW31SvNxJTMilxZL81fQ7Z';
GRANT ALL PRIVILEGES ON xxl_job.* TO 'xxljob'@'%';
FLUSH PRIVILEGES;
```

```go
// 启动命令
docker run -d \
-p 18080:8080 \
-v /data/xxljob:/data/applogs \
-e PARAMS="\
--spring.datasource.url=jdbc:mysql://xxx.xxx.xxx.xxx:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai \
--spring.datasource.username=xxljob \
--spring.datasource.password=xxxxxxxxxxxxxxxxx \
--spring.datasource.driver-class-name=com.mysql.jdbc.Driver \
" \
--name xxl-job-admin \
--restart=always \
xuxueli/xxl-job-admin:3.4.0
```