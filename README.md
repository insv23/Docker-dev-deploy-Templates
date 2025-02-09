# Docker-dev-deploy-Templates

## 文件介绍
- `Dockerfile.dev` - 专门用于开发环境
- `Dockerfile` - 用于生产环境
- `compose.dev.yaml` - 开发环境的服务编排
- `compose.yaml` - 生产环境的服务编排

## 开发
核心策略是: **通过 `compose.dev.yaml` 中的 `develop` 设置将宿主机文件变动 sync 到容器内**

### 构建并启动容器
```bash
docker compose -f compose.dev.yaml build && docker compose -f compose.dev.yaml watch
```
构建应用程序的所有 Docker 镜像，启动 Compose Watch 模式，监听代码更改并自动更新相应的服务。

在开发过程中修改代码时，Compose Watch 会自动重新构建或更新受影响的容器，而无需手动执行 `docker compose up --build` 或 `docker compose restart` 等命令。

### 安装依赖
以 `npm install xxx` 为例

**策略是宿主机安装依赖，package.json 变动，触发容器的 rebuild。**

<details>
  <summary>⁉️ 为什么不通过 <code>docker compose -f compose.dev.yaml exec nextjs npm install ws</code> 这种方式直接在容器内安装依赖?</summary>
  <ul>
    <li>
        需要使用 volumes 功能，将容器内的文件与宿主机彻底同步，有可能导致容器内某些操作反向影响宿主机的文件，并不安全。<br>
        我更倾向于容器只是宿主机文件的新鲜备份。<br>
        (如果使用 VS Code 的 Dev Container 功能，就相当于直接进入容器内，当前设置并不会将在容器内的修改同步到宿主机文件。<br>
        目前这个 Dev Container 的功能我还没摸明白，有时间根据官方指引，找个最小的 Python 环境试一下)
    </li>
    <li>
      即使通过 volumes 功能将容器的文件(主要是 package*.json 这两文件)同步到宿主机，但宿主机上并没有安装对应依赖，编辑器会提示找不到对应的依赖。
    </li>
    </ul>
</details>

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
