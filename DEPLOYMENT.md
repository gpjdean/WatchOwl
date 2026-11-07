# WatchOwl 部署指南（三种方式）

支持三种部署方式，按场景选择：

| 方式 | 适用场景 | 功能完整度 |
|------|---------|-----------|
| **docker-compose** | 单机部署 / 小团队 / 快速交付 | 核心功能 100%（含自监控）；云鸮终端仅 SSH/kubeconfig 模式 |
| **K8s + Helm** | 生产环境 / 集群化运维 | 100%，唯一全自动编排云鸮终端 Pod 的方式 |
| **docker 单容器** | 快速试用 / 开发调试 | 需外部 MySQL+Redis，其余同 compose |

## 0. 镜像仓库

所有镜像已发布到阿里云 ACR（公共可拉取，私有化时先 `docker login`）：

```bash
docker login registry.cn-beijing.aliyuncs.com   # 用户名/密码自备
```

| 组件 | 镜像 |
|------|------|
| 后端主服务 | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:service` |
| 前端 web | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:web` |
| 云鸮终端 agent | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:terminal` |
| MySQL | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:mysql` |
| Redis | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:redis` |
| SkyWalking BanyanDB | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:banyandb` |
| SkyWalking OAP | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:skywalking-oap` |

> 注意：以上 tag 是**滚动最新**。生产建议在 compose/.env 或 values 里锁定具体版本 tag。

---

## 1. docker-compose 部署（推荐起步）

编排文件位于 `backend/deploy/compose/`，包含全套 7 个服务（含内置 SkyWalking 自监控栈）。

### 步骤

```bash
cd backend/deploy/compose

# 1) 准备环境变量(必填 4 项口令/密钥)
cp .env.example .env
vi .env
#   MYSQL_ROOT_PASSWORD / WATCHOWL_ADMIN_PASSWORD
#   WATCHOWL_JWT_SIGNKEY / WATCHOWL_DATASOURCE_ENCRYPTION_KEY
#   密钥建议用 openssl rand -hex 32 生成

# 2) (可选)按需修改
vi conf/config.yaml      # externalUrl 改成实际访问地址(通知链接用)
vi docker-compose.yaml   # 端口/资源/是否需要某服务

# 3) 启动
docker compose up -d
docker compose ps          # 等待全部 healthy/running
docker compose logs -f watchowl-service

# 4) 访问
# http://<主机IP>:8080  账号 admin, 密码 = .env 里的 WATCHOWL_ADMIN_PASSWORD
```

### 拓扑

```
用户 -> watchowl-web(:8080, nginx) -> watchowl-service(:9001)
                                        ├-> watchowl-mysql   (业务库)
                                        ├-> watchowl-redis   (缓存/事件)
                                        └-> watchowl-oap ─> watchowl-banyandb
                                              ↑ service 通过 skywalking-go agent 自动上报
```

### 只起核心栈（不要自监控）

```bash
docker compose up -d --scale watchowl-oap=0 --scale watchowl-banyandb=0
```

后端检测到 `SELF_SKYWALKING_OAP` 不可达时自动跳过自监控 seed，不影响其它功能（/skywalking 页选择其它 SkyWalking 数据源仍可用）。

### 数据与备份

数据全部落 docker volume：`mysql-data` / `redis-data` / `banyandb-data`。

```bash
# 备份业务库(最关键)
docker exec watchowl-mysql sh -c 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" watchowl' > watchowl-$(date +%F).sql
# 恢复
docker exec -i watchowl-mysql sh -c 'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" watchowl' < watchowl-2026-10-06.sql
```

### 升级

```bash
# 改 .env / compose 里的镜像 tag 后:
docker compose pull && docker compose up -d
```

---

## 2. docker 单容器部署（试用/开发）

后端是静态编译的单二进制，只需外挂 MySQL + Redis：

```bash
# 1) 准备外部 MySQL(空库 watchowl)与 Redis, 网络可达即可

# 2) 启动后端
docker run -d --name watchowl-service \
  -p 9001:9001 \
  -e TZ=Asia/Shanghai \
  -e WATCHOWL_DB_PASSWORD='<MySQL口令>' \
  -e WATCHOWL_ADMIN_PASSWORD='<初始admin密码>' \
  -e WATCHOWL_JWT_SIGNKEY="$(openssl rand -hex 32)" \
  -e WATCHOWL_DATASOURCE_ENCRYPTION_KEY="$(openssl rand -hex 32)" \
  registry.cn-beijing.aliyuncs.com/deanmr/watchowl:service
# 数据库/Redis 地址默认 watchowl-mysql/watchowl-redis, 外部部署时需挂自定义配置:
#   -v ./config.yaml:/app/config/config.yaml:ro   (模板见 compose/conf/config.yaml)

# 3) 启动前端(反代到后端)
docker run -d --name watchowl-web \
  -p 8080:80 \
  -v ./watchowl.conf:/etc/nginx/conf.d/default.conf:ro \
  registry.cn-beijing.aliyuncs.com/deanmr/watchowl:web
# nginx 配置模板见 compose/conf/watchowl.conf, 把 upstream 改成你的后端地址

# 4) (可选)接自监控: 再跑 oap + banyandb 两个容器, 并给后端加:
#   -e SELF_SKYWALKING_OAP=http://<oap>:12800 \
#   -e SW_AGENT_NAME=watchowl-service \
#   -e SW_AGENT_REPORTER_GRPC_BACKEND_SERVICE=<oap>:11800 \
```

> 端口 9001 是后端 API（前端反代目标），80 是前端。

---

## 3. K8s + Helm 部署（生产推荐）

编排文件位于 `backend/deploy/helmchart/`。

### 步骤

```bash
# 1) 前置 Secret
kubectl create ns watchowl
kubectl -n watchowl create secret generic watchowl-local-secrets \
  --from-literal=mysql-root-password='<MySQL口令>' \
  --from-literal=adminPassword='<初始admin密码>' \
  --from-literal=datasourceEncryptionKey='<随机串>'
kubectl -n watchowl create secret generic watchowl-jwt-signkey \
  --from-literal=jwtSignKey="$(openssl rand -hex 32)"

# 2) 使用 ACR 镜像: 修改 values(-kind) 中各 image 的 repository/tag
#    service.image.repository: registry.cn-beijing.aliyuncs.com/deanmr/watchowl
#    service.image.tag: service
#    (web/terminal/mysql/redis/skywalking 同理)

# 3) 安装/升级
cd backend/deploy/helmchart
helm install watchowl . -f values.yaml -f <你的环境values> -n watchowl
# 本地 kind 环境示例:
helm upgrade watchowl . -f values.yaml -f values-kind.yaml -f values-loki-kind.yaml -n watchowl

# 4) 等待就绪并访问 NodePort(默认 30593)
kubectl -n watchowl get pods
```

### 关键 values 开关

| 配置项 | 说明 |
|--------|------|
| `skywalking.selfMonitoring.enabled` | 内置 SkyWalking 子栈(OAP+BanyanDB) + 后端自动埋点上报 + 自动 seed 自监控数据源/规则。`false` 时整套不起 |
| `service.terminal.agent.enabled` / `clusterAdmin.enabled` | 云鸮终端：agent Pod 编排 + 管理集群 kubectl 直连（高权限，生产慎开） |
| `web.service.nodePort` | 前端入口端口 |
| `service.externalUrl` | 对外地址（通知跳转链接） |
| `service.outbound.*` / `alertmanager.*` | SSRF 出站白名单（对接内网监控组件时配置） |

### 首次启动自动完成

建表（AutoMigrate）→ 创建 admin（密码取 Secret）→ 预置通知模板/Runbook → 自监控数据源与 2 条预置规则（开启时）。全部幂等（SeedMarker 标记，只建不改，删除不复活）。

---

## 附：云鸮终端在非 K8s 环境的使用

终端 agent 与后端解耦（WebSocket + 共享 Token），三种会话模式：

| 模式 | K8s 部署 | compose/裸机 |
|------|---------|--------------|
| kubectl 连管理集群（in-cluster SA） | ✅（helm 自动编排，SA→cluster-admin） | ❌ 无意义 |
| 指定 kubeconfig 的 kubectl（按会话注入，0600 临时文件，会话结束即删） | ✅ | ✅ |
| SSH 远程主机 | ✅ | ✅ |

compose 下启用：`docker compose --profile terminal up -d`，并在后端与 agent 两侧配置
`WATCHOWL_TERMINAL_AGENT_ADDR / _TOKEN / _URL`（详见 `internal/types/settings.go` 注释）。

## 附：常见问题

| 现象 | 处置 |
|------|------|
| 后端起不来, 日志 `配置读取失败` | config.yaml 必须挂载在 `/app/config/config.yaml`（相对 WORKDIR /app 的 config/config.yaml） |
| agent 静默不上报 | 检查 env 名是 `SW_AGENT_REPORTER_GRPC_BACKEND_SERVICE`（不是 Java 风格的 `SW_AGENT_COLLECTOR_BACKEND_SERVICES`） |
| agent 日志报 `LogWrapper.ChangeLogger` panic | 二进制不是标准产物，必须用 `make build-linux-arm-instrumented` 构建（-tags skywalking + -a + 文件列表含 agent_boot.go） |
| BanyanDB 容器退出且无日志 | 0.8.x 镜像默认入口是 help，必须显式 `command: /banyand standalone --*-root-path ...`（见 compose 文件） |
| OAP 反复重启 | BanyanDB 未就绪的正常现象，起好后 `docker compose restart watchowl-oap` |
| 忘记 admin 密码 | K8s: 改 Secret `watchowl-local-secrets.adminPassword` 重启；compose: 改 `WATCHOWL_ADMIN_PASSWORD` 重建容器（仅首次建号生效，之后走平台 UI 或直改库 bcrypt） |
