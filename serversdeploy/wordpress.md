# Wordpress

[TOC]

记录 wordpress 独立站基础底座安装于部署。

假设我们已经在vps上面安装好了 nginx、mysql等服务，接下来需要安装 php 和 wordpress。笔者环境为 Rocky 9.4。php版本为8.3，wordpress为v6.8.1。mysql为v8.34，nginx为1.27.2，vps规格为2c4G。

## 1、PHP环境安装

我们使用 **Remi 仓库** 获取 PHP 8.3

```go
sudo dnf install -y epel-release
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

启用 PHP 8.3 模块并安装

```go
// 重置默认 PHP 模块
sudo dnf module reset php -y
// 启用 PHP 8.3
sudo dnf module enable php:remi-8.3 -y
// 安装 PHP + 所有必需扩展
sudo dnf install -y \
    php \
    php-fpm \
    php-mysqlnd \
    php-gd \
    php-xml \
    php-mbstring \
    php-json \
    php-curl \
    php-zip \
    php-opcache \
    php-cli \
    php-intl \
    php-bcmath \
    php-soap \
    php-xmlrpc
```

编辑主池配置文件:

```go
sudo vim /etc/php-fpm.d/www.conf
```

**修改以下内容：**

```go
; 用户和组（必须与 Nginx 一致）
user = nginx
group = nginx

; 使用 Unix Socket（高性能、安全）
listen = /run/php-fpm/www.sock

; 设置 socket 权限（让 Nginx 能访问）
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

; 进程管理（动态模式，适合 4G 内存，dynamic）,小内存推荐用ondemand
pm = ondemand
pm.max_children = 20
pm.start_servers = 4
pm.min_spare_servers = 4
pm.max_spare_servers = 8

; 内存限制（防止单个请求吃光内存）
php_admin_value[memory_limit] = 256M
php_admin_value[max_execution_time] = 120
php_admin_value[upload_max_filesize] = 16M
php_admin_value[post_max_size] = 16M
```

```go
sudo vim /etc/php.ini

expose_php = Off

; 开启 OPcache（大幅提升性能）
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=180
opcache.fast_shutdown=1
```

启动并启用 PHP-FPM:

```go
sudo systemctl enable --now php-fpm
sudo systemctl status php-fpm
```

✅ 应显示 `active (running)`
验证 PHP 是否正常工作

```go
sudo mkdir -p /var/www/html
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
sudo vim /etc/nginx/conf.d/test.conf
---
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index info.php;

    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/www.sock;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
---
sudo nginx -t && nginx -s reload
```

浏览器访问：`http://你的服务器IP/info.php`
→ 应看到 **PHP 信息页**，确认：

- PHP Version 8.3.x
- OPcache enabled
- Loaded extensions 包含 `mysqli`, `gd`, `mbstring`, `curl`, `zip`, `bcmath` 等

删除测试文件（安全）:

```go
sudo rm /var/www/html/info.php
sudo rm /etc/nginx/conf.d/test.conf
sudo nginx -s reload
```

## 2、Wordpress整合

下载最新版 WordPress

```go
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz

sudo mkdir -p /var/www/wordpress
sudo cp -r wordpress/* /var/www/wordpress/
sudo chown -R nginx:nginx /var/www/wordpress
sudo chmod -R 755 /var/www/wordpress
```

创建 WordPress 数据库

```go
sudo mysql -u root -p
---
CREATE DATABASE wordpress_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'127.0.0.1' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wpuser'@'127.0.0.1';
FLUSH PRIVILEGES;
EXIT;
```

配置 WordPress

```go
cd /var/www/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo vim wp-config.php
---
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'your_strong_password');
define('DB_HOST', '127.0.0.1:3306');
---
```

添加安全密钥（自动获取），复制输出内容，替换 `AUTH_KEY`, `SECURE_AUTH_KEY` 等字段

```go
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

完成 Web 安装：

1. 访问你的域名或 IP：`http://你的服务器IP`
2. 按照向导填写：
   - 站点标题：baidu Handmade
   - 用户名：不要用 `admin`（建议 `owner`）
   - 密码：强密码
   - 邮箱：你的邮箱
3. 安装完成后，进入 `/wp-admin`

## 3、Nginx配置

```go
cat /etc/nginx/conf.d/wordpress-ip.conf
---
#server {
#    listen 80;
#    server_name baidu.com www.baidu.com;
#    return 301 https://$host$request_uri;
#}


server {
    listen 8443 ssl;
    server_name baidu.com www.baidu.com;
    http2 on;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff always;
    ssl_certificate /etc/nginx/certs/www.baidu.com.pem;
    ssl_certificate_key /etc/nginx/certs/www.baidu.com.key;

    root /var/www/wordpress;
    index index.php index.html index.htm;

    access_log /data/mysql_logs/baidu.log main;
    error_log /data/mysql_logs/baidu_error.log;

    # 防 XSS
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval';" always;
    client_max_body_size 50M;
    # 禁止访问管理台
    #location /wp-admin {
    #    allow 120.230.51.0/24;
    #    deny all;
    #}

    # 禁止访问敏感文件
    location ~* /(\.ht|\.git|\.env|readme\.html|license\.txt|xmlrpc\.php) {
        deny all;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2?)$ {
        expires 1d;
        add_header Cache-Control "public, must-revalidate";
    }

    # PHP 处理（关键！）
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # WordPress 伪静态（必须！否则文章页 404）
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    # 安全：禁止执行 uploads 中的 PHP
    location ~* /wp-content/uploads/.*\.php$ {
        deny all;
    }

    # 隐藏 wp-config.php
    location = /wp-config.php {
        deny all;
    }

    if ($request_method !~ ^(GET|HEAD|POST|OPTIONS|PUT|DELETE)$ ) {
    return 405;
    }

}
---
nginx -t && nginx -s reload
```

