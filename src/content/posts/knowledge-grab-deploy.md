---
title: Knowledge-Grab 知识抓取工具部署指南
published: 2024-01-16
description: 详细介绍如何部署 Knowledge-Grab 开源知识管理和网页抓取工具，支持Docker和手动两种部署方式，包含完整的配置说明和使用教程。
tags: [Knowledge-Grab, Go, 部署, Docker, 知识管理]
category: 部署运维
draft: false
---

# Knowledge-Grab 知识抓取工具部署指南

Knowledge-Grab 是一个基于 Go 语言开发的开源知识管理和网页抓取工具，可以帮助你轻松抓取、保存和管理网页内容，支持多种格式输出。

::github{repo="alterem/knowledge-grab"}

## 一、项目简介

### 1.1 主要特性

- 🚀 **高性能抓取** - 基于 Go 并发特性，快速抓取网页内容
- 📝 **多格式支持** - 支持 Markdown、HTML、PDF 等多种格式
- 🔍 **智能解析** - 自动提取正文内容，过滤广告和无关信息
- 💾 **本地存储** - 支持本地文件系统存储
- 🎨 **简洁界面** - 提供 Web 管理界面
- 🔧 **灵活配置** - 支持自定义抓取规则和输出格式

### 1.2 技术栈

- **后端**: Go 1.21+
- **前端**: HTML/CSS/JavaScript
- **数据库**: SQLite (可选 MySQL/PostgreSQL)
- **容器**: Docker & Docker Compose

## 二、系统要求

### 2.1 硬件要求

- **CPU**: 1核心以上
- **内存**: 512MB以上（推荐1GB+）
- **硬盘**: 5GB以上可用空间
- **网络**: 稳定的互联网连接

### 2.2 软件要求

#### 使用 Docker 部署

```bash
- Docker >= 20.10
- Docker Compose >= 2.0
```

#### 手动部署

```bash
- Go >= 1.21
- Git
- SQLite3 (或 MySQL/PostgreSQL)
```

## 三、Docker 部署（推荐）

### 3.1 安装 Docker

#### CentOS/RHEL

```bash
# 卸载旧版本
yum remove docker docker-client docker-client-latest docker-common \
    docker-latest docker-latest-logrotate docker-logrotate docker-engine

# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加Docker仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装Docker
yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动Docker
systemctl start docker
systemctl enable docker

# 验证安装
docker --version
docker compose version
```

#### Ubuntu/Debian

```bash
# 更新包索引
apt update

# 安装依赖
apt install -y ca-certificates curl gnupg lsb-release

# 添加Docker GPG密钥
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 添加Docker仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动Docker
systemctl start docker
systemctl enable docker

# 验证安装
docker --version
docker compose version
```

### 3.2 克隆项目

```bash
# 创建项目目录
mkdir -p /opt/apps
cd /opt/apps

# 克隆项目
git clone https://github.com/alterem/knowledge-grab.git
cd knowledge-grab
```

### 3.3 配置环境变量

```bash
# 创建环境配置文件
cp .env.example .env

# 编辑配置
vim .env
```

配置示例：

```ini
# 应用配置
APP_NAME=Knowledge-Grab
APP_ENV=production
APP_PORT=8080
APP_DEBUG=false

# 数据库配置（SQLite）
DB_TYPE=sqlite
DB_PATH=/data/knowledge.db

# 或使用MySQL
# DB_TYPE=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_NAME=knowledge_grab
# DB_USER=root
# DB_PASSWORD=your_password

# 存储配置
STORAGE_PATH=/data/contents
MAX_FILE_SIZE=50M

# 抓取配置
FETCH_TIMEOUT=30s
MAX_CONCURRENT=10
USER_AGENT=Mozilla/5.0 (compatible; Knowledge-Grab/1.0)

# 日志配置
LOG_LEVEL=info
LOG_PATH=/data/logs
```

### 3.4 使用 Docker Compose 部署

创建 `docker-compose.yml`（如果项目没有提供）：

```yaml
version: '3.8'

services:
  knowledge-grab:
    build: .
    container_name: knowledge-grab
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./data:/data
      - ./config:/config
    environment:
      - APP_ENV=production
      - APP_PORT=8080
    env_file:
      - .env
    networks:
      - knowledge-net

networks:
  knowledge-net:
    driver: bridge
```

### 3.5 启动服务

```bash
# 构建并启动容器
docker compose up -d

# 查看日志
docker compose logs -f

# 查看容器状态
docker compose ps

# 停止服务
docker compose stop

# 重启服务
docker compose restart
```

### 3.6 访问应用

打开浏览器访问：`http://your-server-ip:8080`

## 四、使用预编译二进制文件部署（推荐新手）

如果不想安装 Go 环境或 Docker，可以直接使用项目提供的预编译二进制文件，这是最简单快速的部署方式。

### 4.1 下载预编译包

访问项目的 [Releases 页面](https://github.com/alterem/knowledge-grab/releases)，下载适合你系统的版本：

```bash
# 创建应用目录
mkdir -p /opt/apps/knowledge-grab
cd /opt/apps/knowledge-grab

# 下载最新版本（以 Linux amd64 为例）
# 请访问 Releases 页面查看最新版本号
VERSION="v1.0.0"  # 替换为最新版本号

# Linux amd64
wget https://github.com/alterem/knowledge-grab/releases/download/${VERSION}/knowledge-grab-linux-amd64.tar.gz

# 或者 Linux arm64
# wget https://github.com/alterem/knowledge-grab/releases/download/${VERSION}/knowledge-grab-linux-arm64.tar.gz

# 解压
tar -xzf knowledge-grab-linux-amd64.tar.gz

# 赋予执行权限
chmod +x knowledge-grab

# 验证安装
./knowledge-grab --version
```

:::tip
**可用的预编译版本：**
- Linux amd64 (x86_64)
- Linux arm64 (aarch64)
- Windows amd64
- macOS amd64
- macOS arm64 (M1/M2)
:::

### 4.2 快速配置

```bash
# 创建必要的目录
mkdir -p config data logs

# 创建配置文件
cat > config/config.yaml <<'EOF'
app:
  name: Knowledge-Grab
  env: production
  port: 8080
  debug: false

database:
  type: sqlite
  path: ./data/knowledge.db

storage:
  path: ./data/contents
  max_file_size: 52428800  # 50MB

fetcher:
  timeout: 30s
  max_concurrent: 10
  user_agent: "Mozilla/5.0 (compatible; Knowledge-Grab/1.0)"

log:
  level: info
  path: ./logs
EOF
```

### 4.3 运行应用

#### 方式1：直接运行（测试）

```bash
# 前台运行
./knowledge-grab

# 指定配置文件
./knowledge-grab --config config/config.yaml

# 后台运行
nohup ./knowledge-grab > logs/app.log 2>&1 &

# 查看进程
ps aux | grep knowledge-grab

# 停止进程
pkill knowledge-grab
```

#### 方式2：配置为系统服务（推荐）

```bash
# 创建 systemd 服务文件
cat > /etc/systemd/system/knowledge-grab.service <<'EOF'
[Unit]
Description=Knowledge-Grab Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/apps/knowledge-grab
ExecStart=/opt/apps/knowledge-grab/knowledge-grab --config config/config.yaml
Restart=on-failure
RestartSec=5s

StandardOutput=journal
StandardError=journal
SyslogIdentifier=knowledge-grab

[Install]
WantedBy=multi-user.target
EOF

# 重新加载 systemd
systemctl daemon-reload

# 启动服务
systemctl start knowledge-grab

# 设置开机自启
systemctl enable knowledge-grab

# 查看状态
systemctl status knowledge-grab

# 查看日志
journalctl -u knowledge-grab -f
```

### 4.4 配置 Nginx 反向代理

```bash
# 安装 Nginx（如果还没安装）
# CentOS/RHEL
yum install -y nginx

# Ubuntu/Debian
apt install -y nginx

# 创建站点配置
cat > /etc/nginx/conf.d/knowledge-grab.conf <<'EOF'
server {
    listen 80;
    server_name your-domain.com;

    access_log /var/log/nginx/knowledge-grab-access.log;
    error_log /var/log/nginx/knowledge-grab-error.log;

    client_max_body_size 50M;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
EOF

# 测试配置
nginx -t

# 重载 Nginx
systemctl reload nginx

# 设置开机自启
systemctl enable nginx
```

### 4.5 访问应用

打开浏览器访问：
- HTTP: `http://your-server-ip:8080` （直接访问）
- HTTP: `http://your-domain.com` （通过Nginx）

:::tip
**使用预编译包的优势：**
- ✅ 无需安装 Go 环境
- ✅ 无需编译，下载即用
- ✅ 部署速度快
- ✅ 文件体积小
- ✅ 适合快速测试和生产部署
:::

## 五、从源码编译部署（高级）

如果需要自定义修改源码，或者 Releases 中没有适合你系统的版本，可以选择从源码编译。

### 5.1 安装 Go 环境

#### 下载并安装 Go

```bash
# 下载Go 1.21（根据最新版本调整）
cd /tmp
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz

# 解压到/usr/local
rm -rf /usr/local/go
tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz

# 配置环境变量
echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile
echo 'export GOPATH=$HOME/go' >> /etc/profile
echo 'export PATH=$PATH:$GOPATH/bin' >> /etc/profile
source /etc/profile

# 验证安装
go version
```

#### 配置 Go 代理（国内用户）

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

### 4.2 克隆并编译项目

```bash
# 克隆项目
cd /opt/apps
git clone https://github.com/alterem/knowledge-grab.git
cd knowledge-grab

# 下载依赖
go mod download

# 编译项目
go build -o knowledge-grab main.go

# 或使用 Makefile（如果提供）
make build
```

### 4.3 配置应用

```bash
# 创建配置文件
mkdir -p config data logs

# 创建配置文件
cat > config/config.yaml <<EOF
app:
  name: Knowledge-Grab
  env: production
  port: 8080
  debug: false

database:
  type: sqlite
  path: ./data/knowledge.db

storage:
  path: ./data/contents
  max_file_size: 52428800  # 50MB

fetcher:
  timeout: 30s
  max_concurrent: 10
  user_agent: "Mozilla/5.0 (compatible; Knowledge-Grab/1.0)"

log:
  level: info
  path: ./logs
EOF
```

### 4.4 创建系统服务

使用 systemd 管理服务：

```bash
cat > /etc/systemd/system/knowledge-grab.service <<EOF
[Unit]
Description=Knowledge-Grab Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/apps/knowledge-grab
ExecStart=/opt/apps/knowledge-grab/knowledge-grab
Restart=on-failure
RestartSec=5s

# 环境变量
Environment="APP_ENV=production"
Environment="APP_PORT=8080"

# 日志
StandardOutput=journal
StandardError=journal
SyslogIdentifier=knowledge-grab

[Install]
WantedBy=multi-user.target
EOF

# 重新加载systemd
systemctl daemon-reload

# 启动服务
systemctl start knowledge-grab

# 设置开机自启
systemctl enable knowledge-grab

# 查看服务状态
systemctl status knowledge-grab

# 查看日志
journalctl -u knowledge-grab -f
```

### 4.5 配置 Nginx 反向代理

```bash
# 创建Nginx配置
cat > /etc/nginx/conf.d/knowledge-grab.conf <<EOF
server {
    listen 80;
    server_name your-domain.com;

    # 日志配置
    access_log /var/log/nginx/knowledge-grab-access.log;
    error_log /var/log/nginx/knowledge-grab-error.log;

    # 客户端上传大小限制
    client_max_body_size 50M;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 超时设置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://127.0.0.1:8080;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
EOF

# 测试Nginx配置
nginx -t

# 重载Nginx
systemctl reload nginx
```

## 五、配置 HTTPS

### 5.1 使用 Let's Encrypt

```bash
# 安装Certbot
# CentOS/RHEL
yum install -y certbot python3-certbot-nginx

# Ubuntu/Debian
apt install -y certbot python3-certbot-nginx

# 申请证书
certbot --nginx -d your-domain.com

# 测试自动续期
certbot renew --dry-run
```

### 5.2 手动配置 SSL

更新 Nginx 配置：

```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    # SSL优化
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;

    # 其他配置...
}

# HTTP跳转HTTPS
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}
```

## 六、使用指南

### 6.1 基本功能

#### 抓取网页

```bash
# 使用命令行
./knowledge-grab fetch https://example.com

# 指定输出格式
./knowledge-grab fetch https://example.com --format markdown

# 批量抓取
./knowledge-grab fetch -f urls.txt
```

#### Web 界面操作

1. 访问 `https://your-domain.com`
2. 输入要抓取的URL
3. 选择输出格式（Markdown/HTML/PDF）
4. 点击"抓取"按钮
5. 查看和管理抓取的内容

### 6.2 API 使用

#### 抓取API

```bash
# POST /api/fetch
curl -X POST https://your-domain.com/api/fetch \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "markdown"
  }'
```

#### 获取内容列表

```bash
# GET /api/contents
curl https://your-domain.com/api/contents
```

#### 获取单个内容

```bash
# GET /api/contents/:id
curl https://your-domain.com/api/contents/123
```

## 七、高级配置

### 7.1 自定义抓取规则

创建 `rules/custom.yaml`：

```yaml
rules:
  - name: "技术博客"
    pattern: "https://blog\\.example\\.com/.*"
    selectors:
      title: "h1.post-title"
      content: "article.post-content"
      author: ".author-name"
      date: ".post-date"
    excludes:
      - ".advertisement"
      - ".sidebar"

  - name: "新闻网站"
    pattern: "https://news\\.example\\.com/.*"
    selectors:
      title: "h1"
      content: ".article-body"
    clean:
      - "script"
      - "style"
      - "iframe"
```

### 7.2 配置定时任务

#### 使用 cron 定时抓取

```bash
# 编辑crontab
crontab -e

# 每天凌晨2点抓取指定URL列表
0 2 * * * cd /opt/apps/knowledge-grab && ./knowledge-grab fetch -f /path/to/urls.txt >> /var/log/knowledge-grab-cron.log 2>&1
```

### 7.3 数据库迁移到 MySQL

```bash
# 1. 创建MySQL数据库
mysql -u root -p <<EOF
CREATE DATABASE knowledge_grab CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'knowledge'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON knowledge_grab.* TO 'knowledge'@'localhost';
FLUSH PRIVILEGES;
EOF

# 2. 更新配置文件
vim config/config.yaml
```

修改数据库配置：

```yaml
database:
  type: mysql
  host: 127.0.0.1
  port: 3306
  name: knowledge_grab
  user: knowledge
  password: your_password
  charset: utf8mb4
  max_idle_conns: 10
  max_open_conns: 100
```

```bash
# 3. 重启服务
systemctl restart knowledge-grab
```

## 八、性能优化

### 8.1 并发抓取优化

编辑配置文件：

```yaml
fetcher:
  max_concurrent: 20        # 增加并发数
  timeout: 60s              # 增加超时时间
  retry_times: 3            # 重试次数
  retry_delay: 5s           # 重试延迟
```

### 8.2 存储优化

```yaml
storage:
  type: filesystem          # 
or s3
  path: ./data/contents
  compress: true              # 启用压缩
  max_file_size: 104857600    # 100MB
  cleanup_days: 90            # 90天后清理旧文件
```

### 8.3 缓存配置

```yaml
cache:
  enabled: true
  type: redis                 # 或 memory
  redis:
    host: 127.0.0.1
    port: 6379
    password: ""
    db: 0
  ttl: 3600                   # 缓存时间（秒）
```

### 8.4 日志优化

```yaml
log:
  level: warn                 # 生产环境使用warn或error
  format: json                # JSON格式便于分析
  max_size: 100               # 单文件最大100MB
  max_backups: 5              # 保留5个备份
  max_age: 30                 # 保留30天
  compress: true              # 压缩旧日志
```

## 九、监控与维护

### 9.1 查看运行状态

```bash
# Docker方式
docker compose ps
docker compose logs -f --tail=100

# 系统服务方式
systemctl status knowledge-grab
journalctl -u knowledge-grab -f

# 查看资源占用
docker stats knowledge-grab
# 或
top -p $(pgrep knowledge-grab)
```

### 9.2 数据备份

创建备份脚本：

```bash
#!/bin/bash
# /root/backup-knowledge-grab.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backup/knowledge-grab"
APP_DIR="/opt/apps/knowledge-grab"

# 创建备份目录
mkdir -p $BACKUP_DIR/{database,data,config}

# 备份数据库（SQLite）
if [ -f "$APP_DIR/data/knowledge.db" ]; then
    cp $APP_DIR/data/knowledge.db $BACKUP_DIR/database/knowledge_$DATE.db
fi

# 备份内容文件
tar -czf $BACKUP_DIR/data/contents_$DATE.tar.gz $APP_DIR/data/contents

# 备份配置文件
tar -czf $BACKUP_DIR/config/config_$DATE.tar.gz $APP_DIR/config

# 删除30天前的备份
find $BACKUP_DIR -type f -mtime +30 -delete

echo "Backup completed: $DATE"
```

设置定时备份：

```bash
chmod +x /root/backup-knowledge-grab.sh

crontab -e
# 每天凌晨3点备份
0 3 * * * /root/backup-knowledge-grab.sh >> /var/log/knowledge-grab-backup.log 2>&1
```

### 9.3 日志管理

配置 logrotate：

```bash
cat > /etc/logrotate.d/knowledge-grab <<EOF
/opt/apps/knowledge-grab/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    create 0644 root root
    sharedscripts
    postrotate
        systemctl reload knowledge-grab > /dev/null 2>&1 || true
    endscript
}
EOF
```

### 9.4 更新应用

#### Docker 方式

```bash
cd /opt/apps/knowledge-grab

# 备份当前版本
docker compose stop
cp -r data data.backup

# 拉取最新代码
git pull origin main

# 重新构建
docker compose build --no-cache

# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f
```

#### 手动部署方式

```bash
cd /opt/apps/knowledge-grab

# 停止服务
systemctl stop knowledge-grab

# 备份当前版本
cp knowledge-grab knowledge-grab.backup
cp -r data data.backup

# 拉取最新代码
git pull origin main

# 重新编译
go build -o knowledge-grab main.go

# 启动服务
systemctl start knowledge-grab

# 查看状态
systemctl status knowledge-grab
```

## 十、常见问题

### 10.1 抓取失败

**问题**: 抓取某些网站时失败

**解决方案**:

```bash
# 1. 检查网络连接
curl -I https://target-website.com

# 2. 增加超时时间
# 编辑配置文件
vim config/config.yaml
# 修改：
fetcher:
  timeout: 60s

# 3. 修改 User-Agent
fetcher:
  user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# 4. 查看详细日志
tail -f logs/app.log | grep ERROR
```

### 10.2 内存占用过高

**问题**: 应用占用内存过大

**解决方案**:

```yaml
# 限制并发数
fetcher:
  max_concurrent: 5

# 限制文件大小
storage:
  max_file_size: 10485760  # 10MB

# 启用内容清理
storage:
  cleanup_days: 30
```

### 10.3 端口被占用

**问题**: 8080端口已被占用

**解决方案**:

```bash
# 查找占用端口的进程
lsof -i :8080
netstat -tulnp | grep 8080

# 修改应用端口
vim config/config.yaml
# 改为其他端口，如 8081

# 重启服务
systemctl restart knowledge-grab
```

### 10.4 数据库锁定

**问题**: SQLite 数据库锁定

**解决方案**:

```bash
# 1. 检查是否有多个实例运行
ps aux | grep knowledge-grab

# 2. 停止所有实例
killall knowledge-grab

# 3. 检查数据库文件
sqlite3 data/knowledge.db "PRAGMA integrity_check;"

# 4. 考虑迁移到 MySQL
# 参考第7.3节
```

### 10.5 证书过期

**问题**: HTTPS 证书过期

**解决方案**:

```bash
# 手动续期
certbot renew

# 检查自动续期
systemctl status certbot.timer

# 强制续期
certbot renew --force-renewal

# 重启Nginx
systemctl reload nginx
```

## 十一、安全加固

### 11.1 配置防火墙

```bash
# 使用 firewalld (CentOS/RHEL)
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# 使用 ufw (Ubuntu/Debian)
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 8080/tcp
ufw enable
```

### 11.2 限制访问

在 Nginx 中添加访问控制：

```nginx
# IP白名单
location /admin {
    allow 192.168.1.0/24;
    deny all;
    proxy_pass http://127.0.0.1:8080;
}

# 基本认证
location / {
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://127.0.0.1:8080;
}
```

创建密码文件：

```bash
# 安装工具
yum install -y httpd-tools  # CentOS
apt install -y apache2-utils  # Ubuntu

# 创建用户
htpasswd -c /etc/nginx/.htpasswd admin

# 重载Nginx
nginx -s reload
```

### 11.3 配置请求限制

```nginx
# 在 Nginx 中限制请求频率
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    
    server {
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://127.0.0.1:8080;
        }
    }
}
```

## 十二、Docker Compose 完整配置示例

```yaml
version: '3.8'

services:
  knowledge-grab:
    build: .
    container_name: knowledge-grab
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./data:/app/data
      - ./config:/app/config
      - ./logs:/app/logs
    environment:
      - APP_ENV=production
      - APP_PORT=8080
      - TZ=Asia/Shanghai
    env_file:
      - .env
    networks:
      - knowledge-net
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # 可选：添加 MySQL
  mysql:
    image: mysql:8.0
    container_name: knowledge-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: knowledge_grab
      MYSQL_USER: knowledge
      MYSQL_PASSWORD: password
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - knowledge-net
    ports:
      - "3306:3306"

  # 可选：添加 Redis
  redis:
    image: redis:7-alpine
    container_name: knowledge-redis
    restart: unless-stopped
    volumes:
      - redis-data:/data
    networks:
      - knowledge-net
    ports:
      - "6379:6379"

networks:
  knowledge-net:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
```

## 十三、总结

本教程详细介绍了 Knowledge-Grab 的完整部署流程：

### ✅ 部署方式

1. **Docker 部署** - 推荐方式，简单快捷，易于管理
2. **手动部署** - 更灵活，适合定制化需求

### ✅ 核心功能

- 🚀 网页内容抓取和保存
- 📝 多格式输出（Markdown/HTML/PDF）
- 💾 本地存储管理
- 🎨 Web 管理界面
- 🔧 灵活的配置选项

### ✅ 运维要点

- **监控** - 定期检查服务状态和资源使用
- **备份** - 自动备份数据库和内容文件
- **日志** - 配置日志轮转，定期清理
- **更新** - 及时更新到最新版本
- **安全** - 配置防火墙、HTTPS、访问控制

### 📋 部署检查清单

- [ ] 安装 Docker 或 Go 环境
- [ ] 克隆项目代码
- [ ] 配置环境变量
- [ ] 启动服务并验证
- [ ] 配置 Nginx 反向代理
- [ ] 申请并配置 SSL 证书
- [ ] 设置自动备份脚本
- [ ] 配置日志轮转
- [ ] 配置防火墙规则
- [ ] 测试核心功能
- [ ] 配置监控告警

### 🔗 相关资源

- [项目 GitHub](https://github.com/alterem/knowledge-grab)
- [Go 官方文档](https://go.dev/doc/)
- [Docker 官方文档](https://docs.docker.com/)
- [Nginx 配置指南](https://nginx.org/en/docs/)

### 💡 使用建议

1. **优先使用 Docker** - 部署简单，环境一致
2. **定期备份数据** - 防止数据丢失
3. **监控服务状态** - 及时发现和解决问题
4. **合理配置并发** - 根据服务器性能调整
5. **使用 HTTPS** - 保证数据传输安全
6. **定期更新版本** - 获取新特性和修复

祝你部署顺利，高效管理你的知识库！🎉

---

**最后更新**: 2024-01-16  
**适用版本**: Knowledge-Grab latest