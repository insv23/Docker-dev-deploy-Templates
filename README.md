# Docker-dev-deploy-Templates

## 文件介绍
- `Dockerfile.dev` - 专门用于开发环境
- `Dockerfile` - 用于生产环境
- `compose.dev.yaml` - 开发环境的服务编排
- `compose.yaml` - 生产环境的服务编排

## 开发

### 构建并启动容器
```bash
docker compose -f compose.dev.yaml build && docker compose -f compose.dev.yaml watch
```
构建应用程序的所有 Docker 镜像，启动 Compose Watch 模式，监听代码更改并自动更新相应的服务。
在开发过程中修改代码时，Compose Watch 会自动重新构建或更新受影响的容器，而无需手动执行 `docker compose up --build` 或 `docker compose restart` 等命令。

### 停止容器
使用 `ctrl c` 只会停止 watch 服务，而容器依然在运行。使用如下命令停止所有容器:
```bash
docker compose -f compose.dev.yaml down
```

### 单独重启某个容器
在开发过程中，遇见因为某个致命错误导致前端或后端服务器彻底卡死的情况并不罕见，以重启 `frontend` 服务并保持 watch 监听为例:
```bash
docker compose -f compose.dev.yaml restart frontend && docker compose -f compose.dev.yaml watch
```

如果需要重新单独构建 `frontend`:
```bash
docker compose -f compose.dev.yaml up -d --build frontend && docker compose -f compose.dev.yaml watch
```

## 部署
启动:
```bash
docker compose up -d
```

不依赖缓存构建后启动:
```bash
docker compose build --no-cache; docker compose up -d
```

停止:
```bash
docker compose down
```
