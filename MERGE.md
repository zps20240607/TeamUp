# 分卷文件合并说明

## 分卷文件列表

- `team-images.tar.gz.part.aa`（450MB）
- `team-images.tar.gz.part.ab`（148MB）

## 在服务器上合并

将上述两个分卷文件上传到服务器同一目录（例如 `/opt/team/`），然后执行：

```bash
cd /opt/team

# 合并分卷
cat team-images.tar.gz.part.aa team-images.tar.gz.part.ab > team-images.tar.gz

# 校验 SHA256（可选）
sha256sum team-images.tar.gz
```

正确的 SHA256 校验值应为：

```
6016525ce8f31c5991eaf619aee32eb1c3fb8c81bee84c840d91b3e4a24e1d80  team-images.tar.gz
```

## 加载镜像并启动

```bash
cd /opt/team

# 加载 Docker 镜像
gunzip -c team-images.tar.gz | docker load

# 启动服务
docker compose up -d
```

## 注意事项

- 分卷必须按顺序合并：`part.aa` + `part.ab`
- 合并后的 `team-images.tar.gz` 大小应为 598MB
- 若校验值不一致，请重新上传分卷
