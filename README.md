# AIPan网盘搜索应用部署指南

本文档提供了AIPan网盘搜索应用的部署说明，包括系统要求、安装步骤和配置选项。

## 系统要求

- Docker (20.10.0或更高版本)
- Docker Compose (v2或更高版本)
- 至少2GB RAM
- 至少10GB可用磁盘空间
- 互联网连接（用于拉取Docker镜像）

## 快速部署

### 1. 下载部署脚本

```bash
curl -O https://raw.githubusercontent.com/unilei/aipan.me/main/deploy.sh
chmod +x deploy.sh
```

### 2. 运行部署脚本

```bash
./deploy.sh
```

### 3. 选择部署选项

脚本将显示以下选项：

```
===== AIPan网盘搜索部署脚本 =====

  1) 完整部署 (首次部署或重新部署)
  2) 更新配置 (仅修改配置并重启服务)
  3) 运行数据库迁移 (初始化或更新数据库结构)
  4) 重新生成 Prisma 客户端 (解决 Prisma 相关问题)
  5) 退出
```

对于首次部署，选择选项`1`。

## 部署过程详解

### 完整部署

选择选项`1`后，脚本将执行以下操作：

1. 检查Docker和Docker Compose是否已安装
2. 创建配置文件（.env和docker-compose.yml）
3. 生成随机管理员凭据（用户名、密码、JWT密钥）
4. 拉取所需的Docker镜像
5. 启动应用服务

部署完成后，应用将在以下地址可用：
- Web应用：http://localhost:3000
- WebSocket：ws://localhost:3002

> **重要提示**：管理员凭据将保存在`admin_credentials.txt`文件中，请妥善保管此文件！

### 更新配置

如需修改配置，选择选项`2`。此选项允许您：

1. 编辑.env文件
2. 重启服务以应用新配置

### 运行数据库迁移

如需初始化或更新数据库结构，选择选项`3`。此选项将：

1. 检查PostgreSQL容器是否运行
2. 执行Prisma数据库迁移
3. 运行数据库种子脚本

### 重新生成Prisma客户端

如遇到Prisma相关问题，选择选项`4`。此选项将：

1. 重新生成Prisma客户端
2. 可选择重启应用以应用更改

## 配置选项

### 端口配置

默认情况下，应用使用以下端口：
- 应用端口：3000
- WebSocket端口：3002

您可以在部署过程中自定义这些端口。

### 环境变量

主要环境变量说明：

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| ADMIN_USER | 管理员用户名 | 随机生成 |
| ADMIN_PASSWORD | 管理员密码 | 随机生成 |
| ADMIN_EMAIL | 管理员邮箱 | 随机生成 |
| JWT_SECRET | JWT密钥 | 随机生成 |
| APP_PORT | 应用端口 | 3000 |
| WS_PORT | WebSocket端口 | 3002 |
| POSTGRES_USER | PostgreSQL用户名 | postgres |
| POSTGRES_PASSWORD | PostgreSQL密码 | postgres |
| POSTGRES_DB | PostgreSQL数据库名 | aipan |

### GitHub配置（可选）

如需启用GitHub集成，请在.env文件中设置以下变量：

```
NUXT_PUBLIC_GITHUB_OWNER="您的GitHub用户名或组织名"
NUXT_PUBLIC_GITHUB_REPO="您的GitHub仓库名"
NUXT_PUBLIC_GITHUB_TOKEN="您的GitHub个人访问令牌"
NUXT_PUBLIC_GITHUB_BRANCH="您的GitHub分支名"
```

### Quark配置（可选）

如需启用Quark集成，请在.env文件中设置以下变量：

```
NUXT_PUBLIC_QUARK_COOKIE="您的Quark Cookie"
```

## 常见问题

### 端口冲突

如果默认端口已被占用，脚本将自动寻找可用端口。

### 数据库认证失败

如果遇到数据库认证失败的错误（如 `Authentication failed against database server, the provided database credentials for 'aipan' are not valid`），可能是因为：

1. 自定义数据库用户未正确创建或缺少权限
2. 数据库连接字符串配置错误

**解决方法**：

- 运行部署脚本选项 `3`（运行数据库迁移），这将自动修复用户权限
- 或者手动执行以下命令创建用户并授权：

```bash
docker exec aipan-postgres psql -U postgres -c "CREATE USER aipan WITH PASSWORD 'aipan123';"
docker exec aipan-postgres psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE aipan TO aipan;"
```

### 表权限错误

如果遇到表权限错误（如 `permission denied for table ForumCategory`），表示数据库用户没有足够的权限访问表。

**解决方法**：

- 运行部署脚本选项 `3`（运行数据库迁移），这将自动更新表权限
- 或者手动执行以下命令授予表权限：

```bash
docker exec aipan-postgres psql -U postgres -d aipan -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO aipan;"
docker exec aipan-postgres psql -U postgres -d aipan -c "GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO aipan;"
```

### 数据持久化

应用数据存储在Docker卷中：
- PostgreSQL数据：postgres-data
- Redis数据：redis-data

### 查看日志

要查看应用日志，请运行：

```bash
docker-compose logs -f aipan-netdisk-search
```

### 停止服务

要停止所有服务，请运行：

```bash
docker-compose down
```

### 重启服务

要重启所有服务，请运行：

```bash
docker-compose restart
```

## 高级用法

### 手动编辑配置

您可以直接编辑.env文件来修改配置，然后重启服务：

```bash
nano .env
docker-compose restart
```

### 清理Docker资源

在部署过程中，您可以选择清理无用的Docker资源，包括：
- 停止的容器
- 未使用的网络
- 未使用的数据卷

## 技术支持

如遇到问题，请提交GitHub Issue或联系管理员。
