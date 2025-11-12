---
layout: default
title: Docker Compose使用
excerpt: 从入门到多环境编排的完整实践指南
tags: [Docker, Compose]
---

## 概述

Docker Compose 用于定义和运行多容器应用。通过一个 `compose.yml` 文件即可描述服务、网络、卷与依赖关系，便于本地开发与多环境一致化部署。

## 安装与快速开始

### 安装

macOS 与 Windows 的 Docker Desktop 默认内置 Compose；Linux 可通过包管理器安装 `docker compose` 子命令。

### 最小示例

```yaml
services:
  web:
    image: nginx:1.25
    ports:
      - "8080:80"
```

运行与查看状态：

```bash
docker compose up -d
docker compose ps
docker compose logs -f web
```

## `compose.yml` 结构与关键字段

常用字段：`image`、`build`、`ports`、`environment`、`volumes`、`depends_on`、`networks`、`healthcheck`。

```yaml
services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=development
      - DB_URL=${DB_URL}
    ports:
      - "3000:3000"
    volumes:
      - ./api:/app
      - node_modules:/app/node_modules
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
    healthcheck:
      test: ["CMD-SHELL","pg_isready -U postgres"]
      interval: 10s
      timeout: 3s
      retries: 5
volumes:
  node_modules:
networks:
  default:
    name: demo-net
```

## 多环境配置

使用 `.env` 与多个 Compose 文件叠加实现环境差异：

```bash
docker compose -f compose.yml -f compose.prod.yml up -d
```

示例：

```yaml
# compose.prod.yml
services:
  api:
    environment:
      - NODE_ENV=production
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
```

## 卷与数据持久化

- 绑定卷：`./data:/var/lib/postgresql/data`
- 匿名/命名卷：`dbdata:/var/lib/postgresql/data`

备份与恢复：

```bash
docker compose exec db pg_dump -U postgres app > backup.sql
docker compose exec -T db psql -U postgres app < backup.sql
```

## 网络与服务发现

Compose 默认创建隔离网络，服务间可用服务名互相访问：`api` 访问 `db:5432`。

自定义网络可设置 `name` 以与其他栈共享。

## 健康检查与依赖

通过 `healthcheck` 与 `depends_on` 的 `condition: service_healthy` 保证依赖顺序与稳定启动。

## 常用命令速查

```bash
docker compose up -d
docker compose down
docker compose build --no-cache
docker compose logs -f api
docker compose exec api sh
docker compose pull
docker compose restart api
```

## 生产环境建议

- 将敏感配置置于环境变量或密钥管理，不写入镜像
- 固化版本：`image: nginx:1.25.4`，避免 `latest`
- 资源限制与健康检查必配
- 使用反向代理与证书自动续期（如 Traefik/Caddy）

## 故障排查

日志、事件与容器状态结合定位：

```bash
docker compose logs --tail=200 api
docker events --since 1h
docker ps -a
```

## 完整示例

```yaml
services:
  web:
    image: caddy:2.8
    volumes:
      - ./site:/srv
      - ./Caddyfile:/etc/caddy/Caddyfile
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      api:
        condition: service_started
  api:
    build:
      context: ./api
    environment:
      - PORT=3000
    ports:
      - "3000:3000"
    healthcheck:
      test: ["CMD","curl","-f","http://localhost:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 5
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=secret
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```

## 总结

以 Compose 为核心的本地与准生产编排可以保持环境一致性、提升交付效率，并为后续迁移至编排系统（如 Swarm/Kubernetes）打好基础。
