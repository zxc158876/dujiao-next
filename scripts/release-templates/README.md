# Dujiao-Next Fullstack Binary

这个包包含 Dujiao-Next 的全功能单二进制（admin + user + api 一体）。

## 快速部署

### 1. 解压

```bash
tar -xzf dujiao-all_*.tar.gz
cd <解压后的目录>
```

### 2. 复制配置文件

```bash
cp config.yml.example config.yml
```

### 3. 必改字段

打开 `config.yml` 编辑：

- **`jwt.secret`** / **`user_jwt.secret`**：必改，生成强随机字符串。例如 `openssl rand -hex 32`
- **`web.admin_path`**：**强烈建议**改成不易猜测的字符串以降低自动化扫描风险。例如 `/dj-mgmt-7x9k2` 或 `/console-private`
- **`redis.host`** / **`redis.port`**：默认 `127.0.0.1:6379` 与本包附带的 `docker-compose.yml` 对应，不改即可
- **`database.driver`** / **`database.dsn`**：默认 SQLite 即可起步；生产建议 PostgreSQL
- 域名、邮件、支付等按需调整

### 4. 启动 Redis

本包附带最小 Redis 容器配置：

```bash
docker compose up -d redis
```

如果你已有 Redis 服务，直接修改 `config.yml` 的 `redis.host` 和 `redis.port` 即可。

### 5. 启动服务

```bash
./dujiao-server
```

二进制运行后会自动创建 `db/`、`uploads/`、`logs/` 目录。

### 6. 访问

- 用户端：`http://<your-ip>:8080`
- 管理端：`http://<your-ip>:8080<web.admin_path>`（默认 `/admin`，建议改）

## 反代示例（Nginx）

把单个域名 `https://shop.example.com` 转发到二进制的 8080 端口：

```nginx
server {
    listen 443 ssl http2;
    server_name shop.example.com;
    # ssl_certificate ...

    client_max_body_size 50m;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 升级

1. 停止服务（Ctrl+C 或 systemctl stop）
2. 备份 `config.yml`、`db/`、`uploads/` 目录
3. 解压新版本 tar.gz，覆盖 `dujiao-server` 二进制
4. 重启服务：`./dujiao-server`

数据库会在启动时自动迁移，无需手动操作。

## 系统服务（systemd 示例）

`/etc/systemd/system/dujiao.service`：
```ini
[Unit]
Description=Dujiao-Next Fullstack
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/dujiao
ExecStart=/opt/dujiao/dujiao-server
Restart=on-failure
User=dujiao
Group=dujiao

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now dujiao
```

## 常见问题

详见在线文档：<https://dujiao-next.github.io/deploy/binary>
