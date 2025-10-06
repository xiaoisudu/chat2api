# Chat2API 部署与使用文档

> 版本：1.8.8-beta2  
> 更新日期：2025-10-06

## 📋 目录

- [项目简介](#项目简介)
- [系统要求](#系统要求)
- [部署方式](#部署方式)
  - [方式一：直接部署](#方式一直接部署)
  - [方式二：Docker 部署](#方式二docker-部署)
  - [方式三：Docker Compose 部署（推荐）](#方式三docker-compose-部署推荐)
  - [方式四：Zeabur 部署](#方式四zeabur-部署)
- [环境变量配置](#环境变量配置)
- [使用指南](#使用指南)
  - [1. Token 管理](#1-token-管理)
  - [2. 逆向 API 使用](#2-逆向-api-使用)
  - [3. 官网镜像使用](#3-官网镜像使用)
- [进阶配置](#进阶配置)
- [常见问题](#常见问题)
- [安全建议](#安全建议)

---

## 项目简介

Chat2API 是一个简单的 ChatGPT TO API 代理服务，具有以下特性：

- 🌟 无需账号即可使用免费、无限的 GPT-3.5
- 💥 支持 AccessToken 使用账号，支持 O3-mini/high、O1/mini/Pro、GPT-4/4o/mini、GPTs
- 🔍 回复格式与真实 API 完全一致，适配几乎所有客户端
- 👮 配套用户管理端 Chat-Share 使用

---

## 系统要求

### 直接部署
- **操作系统**：Linux / macOS / Windows
- **Python 版本**：3.11 或更高
- **内存**：至少 512MB RAM
- **磁盘空间**：至少 1GB 可用空间

### Docker 部署
- **Docker**：20.10 或更高版本
- **Docker Compose**：1.29 或更高版本（如使用 Docker Compose）
- **内存**：至少 1GB RAM
- **磁盘空间**：至少 2GB 可用空间

---

## 部署方式

### 方式一：直接部署

#### 步骤 1：克隆项目

```bash
git clone https://github.com/LanQian528/chat2api
cd chat2api
```

#### 步骤 2：安装依赖

```bash
pip install -r requirements.txt
```

或使用虚拟环境（推荐）：

```bash
python -m venv venv
source venv/bin/activate  # Linux/macOS
# 或
venv\Scripts\activate  # Windows

pip install -r requirements.txt
```

#### 步骤 3：配置环境变量（可选）

创建 `.env` 文件或直接设置环境变量：

```bash
export API_PREFIX=your_prefix
export AUTHORIZATION=sk-your-authorization-key
# 更多环境变量请参考下方"环境变量配置"章节
```

#### 步骤 4：启动服务

```bash
python app.py
```

服务将在 `http://0.0.0.0:5005` 启动。

#### 步骤 5：验证部署

```bash
curl http://localhost:5005/
```

---

### 方式二：Docker 部署

#### 快速启动

```bash
docker run -d \
  --name chat2api \
  -p 5005:5005 \
  lanqian528/chat2api:latest
```

#### 带环境变量启动

```bash
docker run -d \
  --name chat2api \
  -p 5005:5005 \
  -e API_PREFIX=your_prefix \
  -e AUTHORIZATION=sk-your-authorization-key \
  -e ENABLE_GATEWAY=true \
  -v $(pwd)/data:/app/data \
  lanqian528/chat2api:latest
```

#### 查看日志

```bash
docker logs -f chat2api
```

#### 停止和重启

```bash
# 停止容器
docker stop chat2api

# 重启容器
docker restart chat2api

# 删除容器
docker rm -f chat2api
```

---

### 方式三：Docker Compose 部署（推荐）

#### 基础部署

##### 步骤 1：创建项目目录

```bash
mkdir chat2api
cd chat2api
```

##### 步骤 2：下载配置文件

```bash
wget https://raw.githubusercontent.com/LanQian528/chat2api/main/docker-compose.yml
```

##### 步骤 3：修改配置

编辑 `docker-compose.yml`，修改环境变量：

```yaml
version: '3'

services:
  chat2api:
    image: lanqian528/chat2api:latest
    container_name: chat2api
    restart: unless-stopped
    ports:
      - '5005:5005'
    volumes:
      - ./data:/app/data
    environment:
      - TZ=Asia/Shanghai
      - AUTHORIZATION=sk-your-authorization-key  # 修改此处
      - API_PREFIX=your_prefix  # 可选，添加 API 前缀
      - ENABLE_GATEWAY=true  # 可选，启用网关模式

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --cleanup --interval 300 chat2api
```

##### 步骤 4：启动服务

```bash
docker-compose up -d
```

##### 步骤 5：查看日志

```bash
docker-compose logs -f chat2api
```

---

#### 高级部署（带 WARP 代理，推荐用于 PLUS 账号）

适用于需要使用代理或遇到 IP 限制的场景。

##### 步骤 1：下载 WARP 配置文件

```bash
wget https://raw.githubusercontent.com/LanQian528/chat2api/main/docker-compose-warp.yml -O docker-compose.yml
```

##### 步骤 2：修改配置

编辑 `docker-compose.yml`：

```yaml
services:
  warp:
    image: caomingjun/warp
    container_name: warp
    restart: always
    # ... 其他配置保持不变

  chat2api:
    image: lanqian528/chat2api:latest
    container_name: chat2api
    restart: unless-stopped
    ports:
      - '5005:5005'
    environment:
      - TZ=Asia/Shanghai
      - AUTHORIZATION=sk-your-authorization-key  # 修改此处
      - PROXY_URL=socks5://warp:1080  # 使用 WARP 代理
      - ENABLE_GATEWAY=true  # 可选
    # ... 其他配置
```

##### 步骤 3：启动服务

```bash
docker-compose up -d
```

##### 步骤 4：验证 WARP 连接

```bash
# 查看 WARP 健康状态
docker-compose ps

# 查看 WARP 日志
docker-compose logs -f warp
```

---

### 方式四：Zeabur 部署

点击下方按钮一键部署到 Zeabur：

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://zeabur.com/templates/6HEGIZ?referralCode=LanQian528)

部署后在 Zeabur 控制台配置环境变量即可。

---

## 环境变量配置

### 完整环境变量列表

| 分类   | 变量名               | 示例值                                                         | 默认值                   | 描述                                                           |
|------|-------------------|-------------------------------------------------------------|-----------------------|--------------------------------------------------------------|
| 安全相关 | API_PREFIX        | `your_prefix`                                               | `None`                | API 前缀密码，设置后需请求 `/your_prefix/v1/chat/completions`           |
|      | AUTHORIZATION     | `sk-xxx,sk-yyy`                                             | `[]`                  | 授权码，用于多账号轮询，英文逗号分隔                                           |
|      | AUTH_KEY          | `your_auth_key`                                             | `None`                | 私人网关需要加 `auth_key` 请求头才设置该项                                  |
| 请求相关 | CHATGPT_BASE_URL  | `https://chatgpt.com`                                       | `https://chatgpt.com` | ChatGPT 网关地址，多个网关用逗号分隔                                       |
|      | PROXY_URL         | `http://ip:port`,<br/>`socks5://ip:port`                    | `[]`                  | 全局代理 URL，出 403 时启用，多个代理用逗号分隔                                |
|      | EXPORT_PROXY_URL  | `http://ip:port`                                            | `None`                | 出口代理 URL，防止请求图片和文件时泄漏源站 IP                                  |
| 功能相关 | HISTORY_DISABLED  | `true`                                                      | `true`                | 是否不保存聊天记录并返回 conversation_id                                 |
|      | POW_DIFFICULTY    | `00003a`                                                    | `00003a`              | 工作量证明难度，不建议修改                                                |
|      | RETRY_TIMES       | `3`                                                         | `3`                   | 出错重试次数                                                       |
|      | CONVERSATION_ONLY | `false`                                                     | `false`               | 是否直接使用对话接口                                                   |
|      | ENABLE_LIMIT      | `true`                                                      | `true`                | 开启后不尝试突破官方次数限制                                               |
|      | UPLOAD_BY_URL     | `false`                                                     | `false`               | 开启后按照 `URL+空格+正文` 进行对话                                       |
|      | SCHEDULED_REFRESH | `false`                                                     | `false`               | 是否定时刷新 AccessToken                                           |
|      | RANDOM_TOKEN      | `true`                                                      | `true`                | 是否随机选取后台 Token                                               |
| 网关功能 | ENABLE_GATEWAY    | `false`                                                     | `false`               | 是否启用网关模式（镜像站）                                                |
|      | AUTO_SEED         | `true`                                                      | `true`                | 是否启用随机账号模式                                                   |

### 环境变量配置示例

#### Docker 环境变量

```bash
docker run -d \
  --name chat2api \
  -p 5005:5005 \
  -e API_PREFIX=myapi \
  -e AUTHORIZATION=sk-test123,sk-test456 \
  -e ENABLE_GATEWAY=true \
  -e PROXY_URL=http://proxy.example.com:8080 \
  -v $(pwd)/data:/app/data \
  lanqian528/chat2api:latest
```

#### .env 文件格式（直接部署）

```env
# 安全配置
API_PREFIX=myapi
AUTHORIZATION=sk-test123,sk-test456
AUTH_KEY=my-secret-key

# 请求配置
CHATGPT_BASE_URL=https://chatgpt.com
PROXY_URL=http://proxy.example.com:8080

# 功能配置
ENABLE_GATEWAY=true
AUTO_SEED=true
SCHEDULED_REFRESH=true
RANDOM_TOKEN=true
HISTORY_DISABLED=false
```

---

## 使用指南

### 1. Token 管理

#### 访问 Token 管理页面

- **不带前缀**：`http://your-domain:5005/tokens`
- **带前缀**：`http://your-domain:5005/your_prefix/tokens`

#### 上传 Token

1. 访问 Token 管理页面
2. 在输入框中输入 `AccessToken` 或 `RefreshToken`（每行一个）
3. 点击"上传 Tokens"按钮

#### Token 获取方法

##### AccessToken 获取
1. 在浏览器登录 [https://chatgpt.com](https://chatgpt.com)
2. 打开 [https://chatgpt.com/api/auth/session](https://chatgpt.com/api/auth/session)
3. 复制 `accessToken` 字段的值

##### RefreshToken 获取
此处不提供获取方法，请自行研究。

#### 查看和清空 Tokens

- **查看**：访问 Token 管理页面即可看到当前 Token 数量
- **清空**：点击"清空 Tokens"按钮

---

### 2. 逆向 API 使用

#### 基础请求示例

```bash
curl --location 'http://127.0.0.1:5005/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_TOKEN' \
--data '{
  "model": "gpt-3.5-turbo",
  "messages": [{"role": "user", "content": "你好，请介绍一下自己"}],
  "stream": true
}'
```

#### 带前缀的请求

如果设置了 `API_PREFIX=myapi`：

```bash
curl --location 'http://127.0.0.1:5005/myapi/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_TOKEN' \
--data '{
  "model": "gpt-4",
  "messages": [{"role": "user", "content": "你好"}],
  "stream": false
}'
```

#### 支持的模型

| 模型名称 | 说明 | 需要 Token |
|---------|------|-----------|
| `gpt-3.5-turbo` | GPT-3.5 模型 | 否（免登录可用） |
| `gpt-4` | GPT-4 模型 | 是 |
| `gpt-4o` | GPT-4o 模型 | 是 |
| `gpt-4o-mini` | GPT-4o Mini 模型 | 是 |
| `o1-preview` | O1 预览版 | 是 |
| `o1-mini` | O1 Mini 版 | 是 |
| `o1-pro` | O1 Pro 版 | 是 |
| `o3-mini` | O3 Mini 版 | 是 |
| `o3-mini-high` | O3 Mini 高性能版 | 是 |
| `gpt-4-gizmo-g-*` | GPTs 模型 | 是 |

#### 使用授权码轮询 Tokens

如果设置了 `AUTHORIZATION=sk-test123`，可以使用授权码代替具体 Token：

```bash
curl --location 'http://127.0.0.1:5005/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer sk-test123' \
--data '{
  "model": "gpt-4",
  "messages": [{"role": "user", "content": "你好"}]
}'
```

系统将自动从后台 Token 池中选择可用 Token。

#### Team 账号使用

##### 方式一：Header 传入

```bash
curl --location 'http://127.0.0.1:5005/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_TOKEN' \
--header 'ChatGPT-Account-ID: YOUR_TEAM_ACCOUNT_ID' \
--data '{...}'
```

##### 方式二：Token 中传入

```bash
--header 'Authorization: Bearer YOUR_TOKEN,YOUR_TEAM_ACCOUNT_ID'
```

#### 上传图片示例

```bash
curl --location 'http://127.0.0.1:5005/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer YOUR_TOKEN' \
--data '{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这张图片里有什么？"},
        {
          "type": "image_url",
          "image_url": {
            "url": "https://example.com/image.jpg"
          }
        }
      ]
    }
  ]
}'
```

---

### 3. 官网镜像使用

#### 启用网关模式

设置环境变量 `ENABLE_GATEWAY=true`。

#### 访问登录页面

访问 `http://your-domain:5005/login`

#### 登录方式

##### 方式一：在登录页面输入 Token

1. 访问 `/login`
2. 输入 `RefreshToken` 或 `AccessToken`
3. 点击登录

##### 方式二：URL 直接登录

```
http://your-domain:5005/?token=YOUR_TOKEN
```

##### 方式三：使用 Seed Token（随机账号）

```
http://your-domain:5005/?token=seed_any_random_string
```

系统将从后台 Token 池中随机选择一个账号。

#### 功能支持

- ✅ 支持 O3-mini/high、O1/mini/Pro、GPT-4/4o/mini
- ✅ 支持 GPTs 商店
- ✅ 支持 DeepResearch、Canvas 等官网独有功能
- ✅ 支持切换各国语言
- ✅ 会话隔离（不同 SeedToken 会话互不干扰）

---

## 进阶配置

### 配置代理

#### 全局代理（用于访问 ChatGPT）

```bash
# HTTP 代理
PROXY_URL=http://username:password@proxy.example.com:8080

# SOCKS5 代理
PROXY_URL=socks5://proxy.example.com:1080

# 多个代理（轮询）
PROXY_URL=http://proxy1.com:8080,http://proxy2.com:8080
```

#### 出口代理（防止 IP 泄漏）

```bash
EXPORT_PROXY_URL=http://export-proxy.example.com:8080
```

### 配置多个网关地址

```bash
CHATGPT_BASE_URL=https://chatgpt.com,https://mirror1.example.com,https://mirror2.example.com
```

### 定时刷新 Token

```bash
SCHEDULED_REFRESH=true
```

- 每次启动程序：全部非强制刷新一次
- 每 4 天晚上 3 点：全部强制刷新一次

### 自定义端口

修改 `app.py` 最后一行：

```python
uvicorn.run("app:app", host="0.0.0.0", port=8080)  # 改为 8080
```

或使用 Docker 映射：

```bash
docker run -d -p 8080:5005 lanqian528/chat2api:latest
```

### 启用 HTTPS

修改 `app.py`：

```python
uvicorn.run("app:app", host="0.0.0.0", port=5005, 
            ssl_keyfile="key.pem", 
            ssl_certfile="cert.pem")
```

确保 `key.pem` 和 `cert.pem` 文件存在。

---

## 常见问题

### 1. 错误代码说明

| 错误代码 | 原因 | 解决方案 |
|---------|------|---------|
| 401 | IP 不支持免登录或身份验证失败 | 更换 IP 或设置 `PROXY_URL` |
| 403 | 请求被拒绝 | 查看日志具体原因，可能需要配置代理 |
| 429 | 请求频率超限 | 等待或更换 IP |
| 500 | 服务器内部错误 | 查看日志排查问题 |
| 502 | 网关错误或网络不可用 | 检查网络连接或更换网络 |

### 2. IP 限制问题

**问题**：日本 IP 很多不支持免登

**解决方案**：
- 使用美国 IP
- 配置 `PROXY_URL` 使用代理
- 使用 WARP 部署方案

### 3. Token 失效问题

**问题**：Token 过期或无效

**解决方案**：
- 重新获取 AccessToken
- 使用 RefreshToken（自动刷新）
- 启用 `SCHEDULED_REFRESH=true`

### 4. 无法访问服务

**检查步骤**：

```bash
# 1. 检查服务是否运行
docker ps | grep chat2api

# 2. 查看日志
docker logs chat2api

# 3. 检查端口是否开放
netstat -tlnp | grep 5005

# 4. 测试本地访问
curl http://localhost:5005/
```

### 5. 数据持久化

**Docker 挂载数据目录**：

```bash
docker run -d \
  --name chat2api \
  -p 5005:5005 \
  -v $(pwd)/data:/app/data \
  lanqian528/chat2api:latest
```

这样 Token 和配置数据将保存在 `./data` 目录。

### 6. 更新到最新版本

#### Docker 部署

```bash
docker pull lanqian528/chat2api:latest
docker stop chat2api
docker rm chat2api
docker run -d --name chat2api -p 5005:5005 lanqian528/chat2api:latest
```

#### Docker Compose 部署

如果使用了 watchtower，会自动更新。手动更新：

```bash
docker-compose pull
docker-compose up -d
```

#### 直接部署

```bash
cd chat2api
git pull
pip install -r requirements.txt
python app.py
```

---

## 安全建议

### 1. 设置 API 前缀

```bash
API_PREFIX=my_secret_prefix_$(date +%s)
```

这样 API 地址变为 `/my_secret_prefix_xxx/v1/chat/completions`，防止被扫描。

### 2. 使用强授权码

```bash
AUTHORIZATION=sk-$(openssl rand -hex 32)
```

### 3. 配置 AUTH_KEY

如果是私人网关：

```bash
AUTH_KEY=your_very_secret_key
```

所有请求需要添加 Header：

```bash
--header 'auth_key: your_very_secret_key'
```

### 4. 限制访问 IP

使用 Nginx 反向代理并配置 IP 白名单：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        allow 192.168.1.0/24;
        deny all;
        
        proxy_pass http://localhost:5005;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 5. 使用 HTTPS

强烈建议在生产环境使用 HTTPS，可以通过：
- Nginx/Caddy 反向代理 + Let's Encrypt
- Cloudflare CDN
- 直接在应用中配置 SSL 证书

### 6. 定期更换 Token

- 定期更新 AUTHORIZATION 授权码
- 定期刷新 AccessToken
- 监控异常请求

### 7. 日志监控

```bash
# 实时查看日志
docker logs -f chat2api

# 查看最近 100 行日志
docker logs --tail 100 chat2api

# 导出日志到文件
docker logs chat2api > chat2api.log 2>&1
```

---

## 维护和监控

### 查看运行状态

```bash
# Docker 部署
docker ps | grep chat2api
docker stats chat2api

# Docker Compose 部署
docker-compose ps
docker-compose top
```

### 重启服务

```bash
# Docker 部署
docker restart chat2api

# Docker Compose 部署
docker-compose restart

# 直接部署
pkill -f app.py
python app.py
```

### 备份数据

```bash
# 备份 data 目录
tar -czf chat2api-backup-$(date +%Y%m%d).tar.gz data/

# 恢复数据
tar -xzf chat2api-backup-20251006.tar.gz
```

### 性能优化

#### 使用 Gunicorn（生产环境推荐）

修改 Dockerfile：

```dockerfile
CMD ["gunicorn", "app:app", "-w", "4", "-k", "uvicorn.workers.UvicornWorker", "-b", "0.0.0.0:5005"]
```

添加到 `requirements.txt`：

```
gunicorn
```

#### 使用 Redis 缓存（可选）

可以集成 Redis 缓存 Token 和会话数据，提升性能。

---

## 技术支持

- **交流群**：[https://t.me/chat2api](https://t.me/chat2api)
- **GitHub Issues**：[https://github.com/LanQian528/chat2api/issues](https://github.com/LanQian528/chat2api/issues)
- **项目文档**：[README.md](README.md)

提问时请提供：
1. 启动日志截图（敏感信息打码）
2. 报错的日志信息
3. 接口返回的状态码和响应体
4. 环境信息（操作系统、部署方式、版本号）

---

## 附录

### A. 完整 Docker Compose 配置示例

```yaml
version: '3'

services:
  chat2api:
    image: lanqian528/chat2api:latest
    container_name: chat2api
    restart: unless-stopped
    ports:
      - '5005:5005'
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs  # 可选，挂载日志目录
    environment:
      # 时区
      - TZ=Asia/Shanghai
      
      # 安全配置
      - API_PREFIX=my_secret_api
      - AUTHORIZATION=sk-test123,sk-test456
      - AUTH_KEY=my_auth_key
      
      # 请求配置
      - CHATGPT_BASE_URL=https://chatgpt.com
      - PROXY_URL=http://proxy.example.com:8080
      - EXPORT_PROXY_URL=http://export-proxy.example.com:8080
      
      # 功能配置
      - ENABLE_GATEWAY=true
      - AUTO_SEED=true
      - SCHEDULED_REFRESH=true
      - RANDOM_TOKEN=true
      - HISTORY_DISABLED=false
      - RETRY_TIMES=3
      - ENABLE_LIMIT=true
      
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5005/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --cleanup --interval 300 chat2api
```

### B. Nginx 反向代理配置

```nginx
server {
    listen 80;
    server_name chat2api.example.com;
    
    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name chat2api.example.com;
    
    # SSL 证书配置
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    # 日志
    access_log /var/log/nginx/chat2api_access.log;
    error_log /var/log/nginx/chat2api_error.log;
    
    # 反向代理配置
    location / {
        proxy_pass http://localhost:5005;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket 支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # 超时配置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
    
    # 限流配置（可选）
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
    limit_req zone=api_limit burst=20 nodelay;
}
```

### C. Systemd 服务配置（直接部署）

创建 `/etc/systemd/system/chat2api.service`：

```ini
[Unit]
Description=Chat2API Service
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/chat2api
Environment="PATH=/opt/chat2api/venv/bin"
ExecStart=/opt/chat2api/venv/bin/python app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

启用和启动服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable chat2api
sudo systemctl start chat2api
sudo systemctl status chat2api
```

---

## 更新日志

- **2025-10-06**：创建初始部署文档
- 基于版本：1.8.8-beta2

---

## 许可证

本项目采用 MIT 许可证。详见 [LICENSE](LICENSE) 文件。

---

**祝使用愉快！如有问题，欢迎加入 Telegram 交流群讨论。**
