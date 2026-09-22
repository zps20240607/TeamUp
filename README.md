# 学校智能化组队系统 - Docker 部署包

## 一、包内容说明

```
team/
├── docker-compose.yml      # Docker Compose 编排文件
├── backend/                # 后端镜像构建
│   ├── Dockerfile
│   └── application.yml
├── frontend/               # 前端镜像构建
│   ├── Dockerfile
│   └── nginx.conf          # Nginx 反向代理配置
├── mysql/
│   └── init.sql            # school_teamup 数据库初始化脚本
└── redis/
    └── redis.conf          # Redis 配置
```

## 二、服务器部署要求

- 已安装 Docker Engine 24.0+
- 已安装 Docker Compose 2.20+
- 服务器 3306、6379、8080、80 端口未被占用

## 三、快速部署步骤

### 1. 上传项目目录

将整个 `team/` 目录上传到服务器，例如 `/opt/team/`：

```bash
cd /opt/team
```

### 2. 配置环境变量（必做）

```bash
cp .env.example .env
vim .env    # 填写 MYSQL_ROOT_PASSWORD / REDIS_PASSWORD / JWT_SECRET / LANGCHAIN_API_KEY
```

`.env` 已被 `.gitignore` 忽略，不会进入版本库。compose 文件中所有口令都写成
`${VAR:?提示}` 形式：变量缺失时 `docker compose` 会直接报错并打印提示，
不会用空值或弱口令把服务启动起来。

### 3. 构建并启动服务

```bash
docker compose build
docker compose up -d
```

### 4. 验证服务

```bash
# 查看容器状态
docker compose ps

# 测试后端接口
curl http://localhost:8080/api/auth/captcha

# 测试前端页面
curl http://localhost/
```

## 四、NATAPP 内网穿透配置（可选）

如需将服务暴露到公网，可购买 NATAPP Web 类型隧道并绑定自己的二级域名（例如 `your-tunnel.natapp1.cc`），将服务器 `80` 端口映射到该域名：

```bash
# 在 NATAPP 控制台获取隧道 authtoken 后
./natapp -authtoken=你的authtoken
```

## 五、安全注意事项

1. **仓库内不含任何口令**。MySQL / Redis 口令、JWT 密钥、大模型 API Key 全部通过 `.env` 注入，
   `.env` 已被 `.gitignore` 忽略；仓库只提供 `.env.example` 模板。
2. **数据库与 Redis 不对宿主机发布端口**（`docker-compose.yml`）。后端与它们同处 `team-network`，
   通过服务名 `team-mysql` / `team-redis` 访问，无需端口映射。
   在云服务器上把 3306 / 6379 映射出去等于把数据库直接暴露到公网——
   如确需从宿主机连库调试，请使用 `docker-compose.local.yml`（已绑定 `127.0.0.1`）。
3. **Redis 不再把口令写进 `redis.conf`**，改由 compose 以 `--requirepass ${REDIS_PASSWORD}` 注入，
   并保持 `protected-mode yes`。
4. **`mysql/init.sql` 是演示用初始化脚本**，其中的账号、手机号、邮箱均为虚构的种子数据，
   仅用于让本地环境开箱可跑；请在正式环境替换或清空。
5. 构建产物（后端 jar、前端 dist、镜像压缩包）不纳入版本控制。
6. 数据库持久化数据保存在 Docker 命名卷中，如需迁移请额外备份卷数据。

## 六、常用命令

```bash
# 停止服务
docker compose down

# 停止并删除数据卷（会清空数据库和缓存，谨慎使用）
docker compose down -v

# 查看日志
docker compose logs -f team-backend
docker compose logs -f team-mysql
docker compose logs -f team-redis
docker compose logs -f team-frontend

# 进入 MySQL 容器
docker exec -it team-mysql mysql -uroot -p123456 school_teamup
```