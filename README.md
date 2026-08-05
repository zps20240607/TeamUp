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

### 2. 构建并启动服务

```bash
docker compose build
docker compose up -d
```

### 3. 验证服务

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

1. **默认密码仅用于本地开发**。MySQL、Redis 等默认密码（见 `docker-compose.yml`）以及 JWT 密钥，生产环境部署前必须通过环境变量覆盖。
2. **LangChain API Key 通过环境变量注入**：后端从 `LANGCHAIN_API_KEY` 读取（见 `backend/application.yml`），实际值放在本地 `.env` 中，该文件已加入 `.gitignore`，不会提交到 Git。
3. 构建产物（后端 jar、前端 dist、镜像压缩包）不纳入版本控制。
4. 数据库持久化数据保存在 Docker 命名卷中，如需迁移请额外备份卷数据。

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