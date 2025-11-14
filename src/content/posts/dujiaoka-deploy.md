---
title: 独角数卡(Dujiaoka)生产环境部署完全指南
published: 2024-01-15
description: 详细讲解独角数卡系统在Linux服务器上的完整部署流程，包括1Panel面板部署和手动部署两种方式，涵盖环境配置、系统优化、安全加固等生产环境最佳实践。
tags: [独角数卡, Laravel, 部署, Nginx, 1Panel, PHP]
category: 部署运维
draft: false
---

# 独角数卡(Dujiaoka)生产环境部署完全指南

独角数卡是一款基于Laravel + Vue开发的开源自动售货系统，支持多种支付方式，可用于销售各类虚拟商品（卡密、激活码等）。本教程将详细介绍如何在生产环境中部署该系统。

::github{repo="assimon/dujiaoka"}

## 一、系统要求

### 1.1 服务器配置

- **操作系统**: Linux (推荐CentOS 7+/Ubuntu 20.04+)
- **CPU**: 2核心以上
- **内存**: 2GB以上(推荐4GB+)
- **硬盘**: 20GB以上可用空间
- **带宽**: 3Mbps以上

### 1.2 软件环境

```bash
# 核心组件
- PHP >= 8.0 (推荐8.1+)
- MySQL >= 5.7 (推荐8.0+)
- Nginx >= 1.18
- Composer >= 2.0
- Redis >= 5.0
- Supervisor

# 必需的PHP扩展
- OpenSSL
- PDO
- Mbstring
- Tokenizer
- JSON
- BCMath
- Ctype
- Fileinfo
- XML
- GD
- Curl
- Zip
```

## 二、使用1Panel面板部署（推荐）

### 2.1 安装1Panel

1Panel是一款现代化的Linux服务器运维管理面板，相比宝塔更加轻量和安全。

```bash
# 在线安装（推荐）
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && bash quick_start.sh

# 或使用国内加速
curl -sSL https://1panel.cn/quick_start.sh -o quick_start.sh && bash quick_start.sh
```

安装完成后会显示访问地址和初始账号密码：

```
面板地址: http://your-ip:port/entrance
用户名: 1panel
密码: xxxxxxxx
```

### 2.2 准备工作

#### 2.2.1 安装必要的系统工具

在部署前，需要先安装一些1Panel应用商店没有的系统工具。

进入1Panel的**主机** → **终端**，执行以下命令：

```bash
# 根据系统类型安装必要工具

# CentOS/RHEL 系统
yum install -y epel-release
yum install -y git supervisor

# Ubuntu/Debian 系统
apt update
apt install -y git supervisor

# 启动并设置Supervisor开机自启
systemctl enable supervisor
systemctl start supervisor
systemctl status supervisor
```

:::tip
**需要安装的工具说明：**
- **git**: 用于克隆项目代码
- **supervisor**: 用于管理Laravel队列进程（持续运行）

这两个工具在1Panel应用商店中没有，必须手动安装！
:::

### 2.3 在1Panel中配置环境

#### 2.3.1 安装应用商店组件

登录1Panel面板后，进入**应用商店**安装以下组件：

1. **OpenResty** - Nginx的高性能版本
2. **MySQL** - 选择8.0版本
3. **PHP** - 选择8.1版本
4. **Redis** - 选择最新稳定版

安装步骤：
- 点击对应应用的"安装"按钮
- 设置容器名称和端口（使用默认即可）
- 设置root密码（MySQL）
- 点击确认安装

#### 2.3.2 配置PHP扩展

在1Panel中配置PHP：

1. 进入**应用商店** → **已安装**
2. 找到PHP 8.1，点击配置按钮
3. 进入**扩展管理**标签
4. 安装以下扩展：
   - opcache
   - redis
   - zip
   - gd
   - bcmath
   - fileinfo

5. 调整PHP配置（在配置文件标签）：

```ini
memory_limit = 256M
post_max_size = 50M
upload_max_filesize = 50M
max_execution_time = 300
disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru
```

### 2.4 创建网站

#### 2.4.1 添加站点

1. 进入**网站** → **网站管理**
2. 点击**创建网站**
3. 填写信息：
   - 域名: `your-domain.com`
   - 别名: `www.your-domain.com`
   - PHP版本: `PHP-8.1`
   - 网站目录: `/opt/1panel/apps/openresty/openresty/www/sites/dujiaoka`

4. 点击确认创建

#### 2.4.2 创建数据库

1. 进入**数据库** → **MySQL**
2. 点击**创建数据库**
3. 填写信息：
   - 数据库名: `dujiaoka`
   - 用户名: `dujiaoka`
   - 密码: 设置一个强密码
   - 权限: 本地服务器

### 2.5 部署项目

#### 2.5.1 进入终端

在1Panel面板中，进入**主机** → **终端**，或使用SSH连接服务器。

#### 2.5.2 安装Composer

由于1Panel应用商店没有Composer，需要手动安装：

```bash
# 下载Composer安装程序
curl -sS https://getcomposer.org/installer | php

# 全局安装
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

# 验证安装
composer --version

# 配置国内镜像（可选，加速下载）
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

#### 2.5.3 下载项目

```bash
# 进入网站目录
cd /opt/1panel/apps/openresty/openresty/www/sites/dujiaoka

# 克隆项目
git clone https://github.com/assimon/dujiaoka.git .

# 如果没有git，先安装
# CentOS/RHEL
yum install -y git
# Ubuntu/Debian
apt install -y git
```

#### 2.5.4 安装依赖

```bash
# 安装Composer依赖（生产环境）
composer install --optimize-autoloader --no-dev
```

:::tip
如果在前面已配置阿里云镜像，这里下载速度会很快
:::

#### 2.5.5 配置环境变量

```bash
# 复制环境配置文件
cp .env.example .env

# 生成应用密钥
php artisan key:generate

# 编辑配置文件
vim .env
```

修改`.env`配置：

```ini
APP_NAME=独角数卡
APP_ENV=production
APP_DEBUG=false
APP_URL=https://your-domain.com

# 数据库配置
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=dujiaoka
DB_USERNAME=dujiaoka
DB_PASSWORD=your_database_password

# Redis配置
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

# 缓存驱动
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis

# 邮件配置（用于通知）
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-email-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=your-email@gmail.com
MAIL_FROM_NAME="${APP_NAME}"
```

#### 2.5.6 初始化数据库

```bash
# 执行数据库迁移
php artisan migrate --force

# 填充初始数据
php artisan db:seed --force
```

#### 2.4.6 设置权限

```bash
# 设置目录权限
chmod -R 775 storage
chmod -R 775 bootstrap/cache

# 设置所有者（1Panel默认使用www用户）
chown -R www:www /opt/1panel/apps/openresty/openresty/www/sites/dujiaoka
```

### 2.6 配置Nginx

在1Panel中配置Nginx伪静态：

1. 进入**网站** → **网站管理**
2. 找到刚创建的网站，点击**配置**
3. 选择**伪静态**标签
4. 填入以下配置：

```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}

location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

location ~ /\.(?!well-known).* {
    deny all;
}
```

### 2.6 配置队列和定时任务

#### 2.6.1 创建Supervisor配置

在1Panel终端中：

```bash
# 创建Supervisor配置目录
mkdir -p /etc/supervisor/conf.d

# 创建队列配置文件
cat > /etc/supervisor/conf.d/dujiaoka-worker.conf <<'EOF'
[program:dujiaoka-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /opt/1panel/apps/openresty/openresty/www/sites/dujiaoka/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www
numprocs=2
redirect_stderr=true
stdout_logfile=/opt/1panel/apps/openresty/openresty/www/sites/dujiaoka/storage/logs/worker.log
stopwaitsecs=3600
EOF

# 重新加载Supervisor配置
supervisorctl reread
supervisorctl update
supervisorctl start dujiaoka-worker:*
```

#### 2.7.2 配置定时任务

```bash
# 编辑crontab
crontab -e

# 添加以下内容
* * * * * php /opt/1panel/apps/openresty/openresty/www/sites/dujiaoka/artisan schedule:run >> /dev/null 2>&1
```

### 2.7 配置SSL证书

在1Panel中配置HTTPS：

1. 进入**网站** → **网站管理**
2. 找到网站，点击**配置** → **SSL**
3. 选择证书申请方式：
   - **Let's Encrypt**: 自动申请免费证书
   - **上传证书**: 使用已有证书

4. 如果使用Let's Encrypt：
   - 选择DNS验证或HTTP验证
   - 填写邮箱
   - 点击申请

5. 申请成功后，开启**强制HTTPS**

## 三、手动部署（进阶）

### 3.1 系统初始化

```bash
# 更新系统
# CentOS/RHEL
yum update -y

# Ubuntu/Debian
apt update && apt upgrade -y

# 安装必要工具
yum install -y wget curl vim git unzip
```

### 3.2 安装PHP 8.1

#### CentOS/RHEL

```bash
# 添加EPEL和Remi仓库
yum install -y epel-release
yum install -y https://rpms.remirepo.net/enterprise/remi-release-7.rpm

# 启用PHP 8.1
yum install -y yum-utils
yum-config-manager --enable remi-php81

# 安装PHP及扩展
yum install -y php php-cli php-fpm php-mysql php-redis php-gd \
    php-mbstring php-xml php-bcmath php-json php-zip php-opcache \
    php-fileinfo php-tokenizer php-pdo php-curl

# 启动PHP-FPM
systemctl enable php-fpm
systemctl start php-fpm
```

#### Ubuntu/Debian

```bash
# 添加PPA
apt install -y software-properties-common
add-apt-repository ppa:ondrej/php
apt update

# 安装PHP
apt install -y php8.1 php8.1-fpm php8.1-mysql php8.1-redis \
    php8.1-gd php8.1-mbstring php8.1-xml php8.1-bcmath \
    php8.1-zip php8.1-opcache php8.1-curl

# 启动PHP-FPM
systemctl enable php8.1-fpm
systemctl start php8.1-fpm
```

### 3.3 安装MySQL 8.0

```bash
# CentOS/RHEL
yum install -y mysql-server
systemctl enable mysqld
systemctl start mysqld

# 获取临时密码
grep 'temporary password' /var/log/mysqld.log

# 安全配置
mysql_secure_installation

# Ubuntu/Debian
apt install -y mysql-server
systemctl enable mysql
systemctl start mysql
mysql_secure_installation
```

创建数据库：

```bash
mysql -u root -p

CREATE DATABASE dujiaoka CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'dujiaoka'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON dujiaoka.* TO 'dujiaoka'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3.4 安装Redis

```bash
# CentOS/RHEL
yum install -y redis
systemctl enable redis
systemctl start redis

# Ubuntu/Debian
apt install -y redis-server
systemctl enable redis-server
systemctl start redis-server
```

### 3.5 安装Nginx

```bash
# CentOS/RHEL
yum install -y nginx
systemctl enable nginx
systemctl start nginx

# Ubuntu/Debian
apt install -y nginx
systemctl enable nginx
systemctl start nginx
```

配置Nginx站点：

```bash
# 创建配置文件
vim /etc/nginx/conf.d/dujiaoka.conf
```

添加以下内容：

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    root /www/wwwroot/dujiaoka/public;
    
    index index.php index.html;
    
    # 日志配置
    access_log /var/log/nginx/dujiaoka-access.log;
    error_log /var/log/nginx/dujiaoka-error.log;
    
    # 安全头
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";
    
    # Laravel路由
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    # PHP处理
    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        
        # 超时设置
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
    }
    
    # 禁止访问隐藏文件
    location ~ /\.(?!well-known).* {
        deny all;
    }
    
    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

测试并重载Nginx：

```bash
nginx -t
systemctl reload nginx
```

### 3.6 安装Composer

```bash
# 下载Composer
curl -sS https://getcomposer.org/installer | php

# 全局安装
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

# 配置国内镜像（可选）
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

### 3.7 部署项目

```bash
# 创建项目目录
mkdir -p /www/wwwroot
cd /www/wwwroot

# 克隆项目
git clone https://github.com/assimon/dujiaoka.git
cd dujiaoka

# 安装依赖
composer install --optimize-autoloader --no-dev

# 配置环境
cp .env.example .env
php artisan key:generate

# 编辑配置
vim .env
# 按照前面的配置说明填写

# 初始化数据库
php artisan migrate --force
php artisan db:seed --force

# 设置权限
chmod -R 775 storage bootstrap/cache
chown -R nginx:nginx 

/www/wwwroot/dujiaoka

# 如果使用www用户
chown -R www:www /www/wwwroot/dujiaoka
```

### 3.8 配置队列和定时任务

#### 安装Supervisor

```bash
# CentOS/RHEL
yum install -y supervisor
systemctl enable supervisord
systemctl start supervisord

# Ubuntu/Debian
apt install -y supervisor
systemctl enable supervisor
systemctl start supervisor
```

#### 配置队列Worker

```bash
cat > /etc/supervisor/conf.d/dujiaoka-worker.conf <<'EOF'
[program:dujiaoka-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /www/wwwroot/dujiaoka/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=nginx
numprocs=2
redirect_stderr=true
stdout_logfile=/www/wwwroot/dujiaoka/storage/logs/worker.log
stopwaitsecs=3600
EOF

# 重新加载配置
supervisorctl reread
supervisorctl update
supervisorctl start dujiaoka-worker:*
```

#### 配置定时任务

```bash
crontab -e

# 添加Laravel调度器
* * * * * cd /www/wwwroot/dujiaoka && php artisan schedule:run >> /dev/null 2>&1
```

## 四、系统初始化配置

### 4.1 访问后台

访问 `https://your-domain.com/admin` 使用默认账号登录：

```
账号: admin
密码: admin
```

:::warning
首次登录后请立即修改默认密码！
:::

### 4.2 基础设置

1. **网站信息**
   - 网站名称
   - 网站公告
   - 联系方式

2. **支付配置**
   - 配置支付宝
   - 配置微信支付
   - 配置其他支付方式

3. **邮件配置**
   - SMTP服务器设置
   - 测试邮件发送

4. **商品管理**
   - 创建商品分类
   - 添加商品
   - 设置价格

## 五、性能优化

### 5.1 启用OPcache

编辑 `/etc/php.ini` 或 `/etc/php/8.1/fpm/php.ini`：

```ini
[opcache]
opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
```

重启PHP-FPM：

```bash
systemctl restart php-fpm
# 或
systemctl restart php8.1-fpm
```

### 5.2 配置Redis缓存

确保`.env`中已配置Redis：

```ini
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
```

清空并重建缓存：

```bash
php artisan cache:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### 5.3 数据库优化

```sql
-- 连接MySQL
mysql -u root -p

USE dujiaoka;

-- 添加索引（根据实际查询优化）
ALTER TABLE orders ADD INDEX idx_status (status);
ALTER TABLE orders ADD INDEX idx_created_at (created_at);
ALTER TABLE goods ADD INDEX idx_category (category_id);

-- 优化表
OPTIMIZE TABLE orders;
OPTIMIZE TABLE goods;
```

### 5.4 Nginx性能优化

编辑Nginx配置：

```nginx
# 启用gzip压缩
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_types text/plain text/css text/xml text/javascript 
           application/json application/javascript application/xml+rss 
           application/rss+xml application/atom+xml image/svg+xml;

# 连接优化
keepalive_timeout 65;
keepalive_requests 100;

# 缓冲区优化
client_body_buffer_size 128k;
client_max_body_size 50m;
client_header_buffer_size 2k;
large_client_header_buffers 4 8k;

# FastCGI优化
fastcgi_buffer_size 64k;
fastcgi_buffers 4 64k;
fastcgi_busy_buffers_size 128k;
```

## 六、安全加固

### 6.1 配置防火墙

```bash
# 使用firewalld (CentOS/RHEL)
systemctl start firewalld
systemctl enable firewalld

# 开放必要端口
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload

# 或使用ufw (Ubuntu/Debian)
ufw allow 22/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw enable
```

### 6.2 禁用危险PHP函数

编辑 `php.ini`：

```ini
disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru
```

### 6.3 设置文件权限

```bash
# 设置正确的所有者
chown -R nginx:nginx /www/wwwroot/dujiaoka

# 设置目录权限
find /www/wwwroot/dujiaoka -type d -exec chmod 755 {} \;

# 设置文件权限
find /www/wwwroot/dujiaoka -type f -exec chmod 644 {} \;

# 特殊目录需要写权限
chmod -R 775 /www/wwwroot/dujiaoka/storage
chmod -R 775 /www/wwwroot/dujiaoka/bootstrap/cache
```

### 6.4 隐藏服务器信息

#### 隐藏Nginx版本

编辑 `/etc/nginx/nginx.conf`：

```nginx
http {
    server_tokens off;
    ...
}
```

#### 隐藏PHP版本

编辑 `php.ini`：

```ini
expose_php = Off
```

### 6.5 配置HTTPS

使用Let's Encrypt免费证书：

```bash
# 安装Certbot
# CentOS/RHEL
yum install -y certbot python3-certbot-nginx

# Ubuntu/Debian
apt install -y certbot python3-certbot-nginx

# 申请证书
certbot --nginx -d your-domain.com -d www.your-domain.com

# 自动续期测试
certbot renew --dry-run
```

Nginx HTTPS配置会自动生成，或手动添加：

```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;
    
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # SSL优化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # 其他配置...
}

# HTTP跳转HTTPS
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}
```

## 七、常见问题解决

### 7.1 500错误

**问题**: 访问网站出现500错误

**解决方案**:

```bash
# 1. 检查storage权限
chmod -R 775 storage bootstrap/cache
chown -R nginx:nginx storage bootstrap/cache

# 2. 清除缓存
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# 3. 查看错误日志
tail -f storage/logs/laravel.log
tail -f /var/log/nginx/error.log
```

### 7.2 队列不工作

**问题**: 订单创建后不自动处理

**解决方案**:

```bash
# 检查队列进程
supervisorctl status

# 重启队列
supervisorctl restart dujiaoka-worker:*

# 手动执行队列测试
php artisan queue:work

# 查看失败任务
php artisan queue:failed
php artisan queue:retry all
```

### 7.3 数据库连接失败

**问题**: SQLSTATE[HY000] [2002] Connection refused

**解决方案**:

```bash
# 检查MySQL服务
systemctl status mysqld

# 检查.env配置
DB_HOST=127.0.0.1  # 不要用localhost
DB_PORT=3306

# 测试连接
mysql -h 127.0.0.1 -u dujiaoka -p

# 检查MySQL绑定地址
vim /etc/my.cnf
# 确保有: bind-address = 127.0.0.1
```

### 7.4 文件上传失败

**问题**: 上传图片或文件失败

**解决方案**:

```bash
# 检查storage权限
chmod -R 775 storage/app/public

# 创建软链接
php artisan storage:link

# 增加PHP上传限制
vim /etc/php.ini
# 修改：
upload_max_filesize = 50M
post_max_size = 50M

# 重启PHP-FPM
systemctl restart php-fpm
```

### 7.5 支付回调失败

**问题**: 支付成功但订单状态未更新

**解决方案**:

```bash
# 1. 确保队列正常运行
supervisorctl status dujiaoka-worker

# 2. 检查域名可被外网访问
curl https://your-domain.com

# 3. 查看支付日志
tail -f storage/logs/laravel.log | grep payment

# 4. 检查防火墙
firewall-cmd --list-all

# 5. 测试回调URL
# 确保支付平台能访问你的回调地址
```

### 7.6 Composer安装慢

**问题**: composer install 很慢或失败

**解决方案**:

```bash
# 使用国内镜像
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 增加内存限制
php -d memory_limit=-1 /usr/local/bin/composer install

# 使用swap（内存不足时）
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

## 八、系统维护

### 8.1 数据备份

创建自动备份脚本：

```bash
#!/bin/bash
# /root/backup-dujiaoka.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/dujiaoka"
PROJECT_DIR="/www/wwwroot/dujiaoka"
DB_NAME="dujiaoka"
DB_USER="dujiaoka"
DB_PASS="your_password"

# 创建备份目录
mkdir -p $BACKUP_DIR/{database,files}

# 备份数据库
mysqldump -u$DB_USER -p$DB_PASS $DB_NAME | gzip > $BACKUP_DIR/database/db_$DATE.sql.gz

# 备份文件
tar -czf $BACKUP_DIR/files/storage_$DATE.tar.gz $PROJECT_DIR/storage/app
tar -czf $BACKUP_DIR/files/config_$DATE.tar.gz $PROJECT_DIR/.env

# 删除30天前的备份
find $BACKUP_DIR -type f -mtime +30 -delete

echo "Backup completed: $DATE"
```

设置定时备份：

```bash
chmod +x /root/backup-dujiaoka.sh

# 添加到crontab
crontab -e

# 每天凌晨3点备份
0 3 * * * /root/backup-dujiaoka.sh >> /var/log/backup.log 2>&1
```

### 8.2 系统更新

```bash
# 1. 备份当前版本
cp -r /www/wwwroot/dujiaoka /www/wwwroot/dujiaoka_backup_$(date +%Y%m%d)

# 2. 进入项目目录
cd /www/wwwroot/dujiaoka

# 3. 拉取最新代码
git pull origin master

# 4. 更新依赖
composer install --optimize-autoloader --no-dev

# 5. 执行迁移
php artisan migrate --force

# 6. 清除并重建缓存
php artisan cache:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 7. 重启队列
supervisorctl restart dujiaoka-worker:*
```

### 8.3 日志管理

配置日志轮转：

```bash
# 创建logrotate配置
vim /etc/logrotate.d/dujiaoka
```

添加内容：

```
/www/wwwroot/dujiaoka/storage/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    create 0644 nginx nginx
    sharedscripts
    postrotate
        systemctl reload php-fpm
    endscript
}
```

查看日志：

```bash
# 实时查看Laravel日志
tail -f /www/wwwroot/dujiaoka/storage/logs/laravel.log

# 查看Nginx访问日志
tail -f /var/log/nginx/dujiaoka-access.log

# 查看Nginx错误日志
tail -f /var/log/nginx/dujiaoka-error.log

# 查看队列日志
tail -f /www/wwwroot/dujiaoka/storage/logs/worker.log
```

### 8.4 性能监控

安装监控工具（开发环境）：

```bash
cd /www/wwwroot/dujiaoka

# 安装Telescope
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate

# 发布资源
php artisan telescope:publish
```

访问 `https://your-domain.com/telescope` 查看应用性能。

:::warning
生产环境建议只在需要时临时启用Telescope
:::

## 九、进阶配置

### 9.1 配置CDN

使用阿里云OSS或七牛云：

```bash
# 安装OSS扩展
composer require jacobcyl/ali-oss-storage
```

修改 `config/filesystems.php`：

```php
'disks' => [
    'oss' => [
        'driver' => 'oss',
        'access_id' => env('OSS_ACCESS_KEY_ID'),
        'access_key' => env('OSS_ACCESS_KEY_SECRET'),
        'bucket' => env('OSS_BUCKET'),
        'endpoint' => env('OSS_ENDPOINT'),
        'cdn_domain' => env('OSS_CDN_DOMAIN'),
    ],
],
```

### 9.2 负载均衡部署

多服务器部署架构：

```
                   +-------------------+
                   |   Load Balancer   |
                   |    (Nginx/LVS)    |
                   +---------+---------+
                             |
           +-----------------+-----------------+
           |                 |                 |
    +------v------+   +------v------+   +------v------+
    |   Web-1     |   |   Web-2     |   |   Web-3     |
    | (Nginx+PHP) |   | (Nginx+PHP) |   | (Nginx+PHP) |
    +------+------+   +------+------+   +------+------+
           |                 |                 
|
           +-----------------+-----------------+
                             |
                    +--------v--------+
                    | MySQL Master    |
                    |   + Redis       |
                    +-----------------+
```

**配置要点**：

1. **共享存储**: 使用NFS共享storage目录
2. **会话共享**: 使用Redis存储session
3. **数据库**: 使用独立MySQL服务器
4. **队列**: 在一台服务器上统一运行队列

### 9.3 启用全站缓存

```bash
# 安装predis
composer require predis/predis

# 配置.env
CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

# 缓存配置
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## 十、总结

本教程详细介绍了独角数卡系统的完整部署流程，包括：

### ✅ 部署方式

1. **1Panel面板部署** - 适合新手，界面友好，操作简单
2. **手动部署** - 适合进阶用户，更灵活可控

### ✅ 核心配置

- **环境要求**: PHP 8.1+, MySQL 8.0+, Redis, Nginx
- **权限设置**: storage和cache目录必须可写
- **队列管理**: 使用Supervisor管理队列进程
- **定时任务**: 配置crontab执行定期任务

### ✅ 优化要点

- **性能优化**: OPcache、Redis缓存、数据库索引
- **安全加固**: 防火墙、HTTPS、权限控制
- **系统维护**: 自动备份、日志管理、监控告警

### 📋 部署检查清单

部署完成后，请确认以下项目：

- [ ] 网站可正常访问
- [ ] 后台可正常登录，已修改默认密码
- [ ] 队列进程正常运行 (`supervisorctl status`)
- [ ] 定时任务已配置 (`crontab -l`)
- [ ] SSL证书已配置且自动续期
- [ ] 数据库已优化，添加必要索引
- [ ] 日志轮转已配置
- [ ] 自动备份脚本已设置
- [ ] 防火墙规则已配置
- [ ] 文件上传功能正常
- [ ] 支付回调测试通过
- [ ] 邮件发送功能正常

### 🔗 相关资源

- [独角数卡官方GitHub](https://github.com/assimon/dujiaoka)
- [Laravel官方文档](https://laravel.com/docs)
- [1Panel官方文档](https://1panel.cn/docs)
- [Nginx配置指南](https://nginx.org/en/docs/)
- [Let's Encrypt文档](https://letsencrypt.org/docs/)

### 💡 最佳实践建议

1. **使用1Panel面板** - 降低运维难度，提高效率
2. **定期备份数据** - 每天自动备份数据库和重要文件
3. **监控系统状态** - 使用监控工具及时发现问题
4. **保持系统更新** - 定期更新系统和应用版本
5. **安全第一** - 使用HTTPS，定期更新密码
6. **优化性能** - 启用缓存，优化数据库查询
7. **日志分析** - 定期查看日志，及时发现异常

### 🆘 获取帮助

如果在部署过程中遇到问题：

1. 查看项目Issues: https://github.com/assimon/dujiaoka/issues
2. 查看系统日志定位问题
3. 在社区寻求帮助
4. 参考本教程的常见问题章节

祝你部署顺利！🎉

---

**最后更新**: 2024-01-15  
**适用版本**: 独角数卡 v2.x