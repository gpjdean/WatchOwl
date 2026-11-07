<p align="center">
  <h1 align="center">🌐 WatchOwl</h1>
  <p align="center"><b>云原生一体化运维平台 · 监控告警 × 集群管理 × 智能运维</b></p>
</p>


<p align="center">
<a href="https://github.com/watch-owl/WatchOwl/graphs/contributors">
  <img alt="GitHub contributors" src="https://img.shields.io/github/contributors-anon/watch-owl/WatchOwl"/></a>
<img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/watch-owl/WatchOwl">
<img alt="GitHub forks" src="https://img.shields.io/github/forks/watch-owl/WatchOwl">
<img alt="GitHub Repo issues" src="https://img.shields.io/github/issues/watch-owl/WatchOwl">
<img alt="GitHub Repo issues closed" src="https://img.shields.io/github/issues-closed/watch-owl/WatchOwl">
<img alt="License" src="https://img.shields.io/badge/license-Apache--2.0-blue"/>
<img alt="Arch" src="https://img.shields.io/badge/arch-amd64%20%7C%20arm64-critical"/>
</p>


> 💡 提示：仓库地址以 `watch-owl/WatchOwl` 作为占位，请按你实际的 GitHub 组织/仓库替换 README 与 CI 中的镜像命名空间和仓库链接。

## 💎 WatchOwl 是什么？

**WatchOwl 是一个开箱即用的云原生一体化运维平台**——把「看得见」（监控告警）、
「管得住」（集群与主机管理）、「修得快」（剧本编排 / 自愈 / 云鸮终端）放进同一个控制台，
让运维不再在监控、终端、审批流、CMDB 之间来回切换。

一句话概括它的三层能力：

| 层次               | 能做什么                                                     | 典型场景                         |
| ------------------ | ------------------------------------------------------------ | -------------------------------- |
| 🔍 **可观测与告警** | Metrics / Logs / Traces / 事件 / 拨测全链路采集评估，告警升级、静默抑制、值班排班、多渠道通知 | 线上告警 5 分钟内找到值班人      |
| 🖥️ **基础设施管理** | 多 Kubernetes 集群纳管（工作负载/网络/存储/RBAC/拓扑）、主机管理、数据库与中间件管理、多集群 kubectl 终端 | 一个页面巡检全部集群和数据库     |
| 🤖 **自动化与治理** | Ansible 剧本编排、定时任务、自愈规则、高危操作执行审批、全量审计 | 自愈磁盘告警，敏感操作走审批留痕 |

在此之上，**AI 全程加持**：告警内容深度语义解析、根因推测与修复建议自动生成，
让每一次告警都「言之有物」，让每一次变更都「有据可查」。

![架构图](assets/architecture.png)

## 🧩 全面兼容主流可观测技术栈

|   监控类型    | 支持的数据源                                                 |
| :-----------: | ------------------------------------------------------------ |
|    Metrics    | Prometheus、AWS CloudWatch                                   |
|     Logs      | Loki、ElasticSearch、VictoriaLogs、ClickHouse、SLS（阿里云日志服务）、TLS（火山云日志服务，开发中）、CLS（腾讯云日志服务，开发中） |
|    Traces     | Jaeger                                                       |
|    Events     | Kubernetes 事件监控                                          |
| 数据库/中间件 | MySQL、PostgreSQL、SQL Server、MongoDB、Redis（单机 / 哨兵 / Cluster）、Kafka、RabbitMQ、RocketMQ、Etcd |
|   网络拨测    | HTTP、ICMP、TCP、SSL                                         |
|   通知渠道    | 飞书、钉钉、企业微信、邮件、自定义 Webhook、Slack、短信、电话加急 |

## 🔍 核心亮点

🧠 **AI 智能分析**

- 基于 AI 技术对告警内容进行深度语义解析，自动识别异常模式
- 提供根因推测、排查建议与修复思路，让每一次告警都"言之有物"

🕰️ **完善的值班机制**

- 支持轮班排班、节假日调整、值班交接等场景
- 告警通知精准匹配责任人，确保第一时间响应

⚡ **告警升级机制**

- 多级告警策略配置：从首次触发到升级通知，层层保障不漏报
- 支持超时重试、通知升级、负责人转接等功能，保障告警闭环处理

📊 **Namespace 级告警分类**

- 支持以命名空间（Namespace）为单位进行告警分组管理
- 清晰分类，快速定位，大幅提升故障处理效率

🔇 **静默与抑制**

- 灵活的静默规则，避免在维护窗口期产生噪音告警

## 🏗️ 架构概览

WatchOwl 采用前后端分离的云原生架构：

```
┌───────────────┐      ┌──────────────────────────────────────────────┐      ┌─────────────┐
│   浏览器/终端   │◀────▶│  Web 前端 (React + Ant Design)                │      │             │
└───────────────┘      │  - 仪表盘 / 规则 / 故障中心 / 值班 / 拨测 ...      │      │             │
                       └──────────────────────────────────────────────┘      │             │
                                    │ /api 代理 (Nginx)                       │  Prometheus │
                                    ▼                                         │  Loki/ES ...│
                       ┌──────────────────────────────────────────────┐      │  Jaeger     │
                       │  后端服务 (Go + Gin)                           │◀────▶│  Kubernetes │
                       │  - 告警评估引擎 (Metrics/Logs/Traces/拨测)      │      │  拨测目标     │
                       │  - 告警升级 / 静默 / 值班 / 通知调度            │      └─────────────┘
                       │  - AI 分析 / 审计 / 多租户 / RBAC              │
                       └──────────────────────────────────────────────┘
                                    │           │
                          ┌─────────▼──┐   ┌───▼─────┐
                          │  MySQL 8.0 │   │  Redis  │
                          │ (持久化)    │   │ (缓存)   │
                          └────────────┘   └─────────┘
```

## 🚀 技术栈

- **后端**：Go ≥ `1.23`、Gin、Viper、Gorm、JWT、go-zero
- **前端**：Node.js ≥ `v18.20.3`、Yarn ≥ `1.22.22`、React、Ant Design、CodeMirror
- **基础设施**：MySQL 8.0、Redis

## 🐳 多架构容器镜像

WatchOwl 的前后端镜像同时支持 **linux/amd64** 与 **linux/arm64** 两种架构，可在 x86 服务器与 ARM（如鲲鹏、Apple Silicon、树莓派）环境上统一部署。

| 组件 | 镜像                             | 架构                     |
| ---- | -------------------------------- | ------------------------ |
| 后端 | `watch-owl/watch-owl:latest`     | linux/amd64, linux/arm64 |
| 前端 | `watch-owl/watch-owl-web:latest` | linux/amd64, linux/arm64 |

> 镜像命名空间 `watch-owl/` 为示例，请按你自己的 Docker Hub 账号替换（CI 中的 `DOCKERHUB_USER` / `DOCKERHUB_TOKEN` Secret 同步更新）。

多架构镜像通过 GitHub Actions + Docker Buildx 自动构建：

- 打 tag（`v*`）触发 [release-ci](./backend/.github/workflows/release-ci.yml) / [前端 release-ci](./frontend/.github/workflows/release-ci.yml)，推送 `latest` 与版本号镜像。
- 推送到 `master`/`main` 分支触发 test-ci，推送以分支名命名的镜像。

后端 Dockerfile 使用 `--platform=${BUILDPLATFORM}` + `TARGETARCH` 让 Go 工具链在构建平台原生运行，通过 `GOARCH` 交叉编译目标架构，避免 QEMU 模拟整个 `go build`，arm64 构建提速明显。

### 手动多架构构建

```bash
# 后端
cd backend
docker buildx build --platform linux/amd64,linux/arm64 \
  --build-arg VERSION=v1.0.0 \
  -t watch-owl/watch-owl:latest -t watch-owl/watch-owl:v1.0.0 \
  --push .

# 前端（Nginx 发布镜像）
cd frontend
docker buildx build --platform linux/amd64,linux/arm64 \
  -t watch-owl/watch-owl-web:latest \
  -f Dockerfile-nginx --push .
```

仅构建本机架构镜像用于本地验证：

```bash
docker build -t watch-owl/watch-owl:local .          # 后端
docker build -f Dockerfile-nginx -t watch-owl/watch-owl-web:local .   # 前端
```

## ⚡ 快速开始（Docker Compose）

**官方镜像**（阿里云容器镜像服务，公网可拉取）：

| 组件           | 镜像                                                        |
| -------------- | ----------------------------------------------------------- |
| 后端服务       | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:service`  |
| 前端 Web       | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:web`      |
| 云鸮终端 Agent | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:terminal` |
| 内置 MySQL     | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:mysql`    |
| 内置 Redis     | `registry.cn-beijing.aliyuncs.com/deanmr/watchowl:redis`    |

> 五个镜像同仓库同发版，按组件用语义 tag 区分；国内环境无需配置任何镜像加速即可拉取。
> Compose / Helm / 裸 K8s YAML 三种部署方式默认均使用上述镜像，开箱即用。

```bash
cd backend/deploy/docker-compose

# 口令与密钥统一走环境变量注入, 不要写进 config.yaml
# (config.yaml 在 Helm 部署下会渲染成 ConfigMap, 明文存在里面等于公开)
cat > .env <<'EOF'
WATCHOWL_DB_PASSWORD=换成你自己的数据库口令
WATCHOWL_ADMIN_PASSWORD=换成 admin 的初始口令
WATCHOWL_JWT_SIGNKEY=用 openssl rand -hex 32 生成一个随机串
EOF

docker compose up -d
```

启动后访问 `http://<服务器IP>` 进入 Web 界面，首次访问需初始化 admin 密码。
后端 API 默认监听 `9001`，Web 默认监听 `80`。

> `WATCHOWL_DB_PASSWORD` 与 `WATCHOWL_ADMIN_PASSWORD` 为必填项；缺少数据库口令时 Compose 会拒绝启动，
> 新安装缺少管理员初始口令时后端会终止启动。`WATCHOWL_JWT_SIGNKEY` 留空时进程每次重启都会换一把
> 临时密钥（已登录用户全部掉线），生产环境也必须显式设置。

| 环境变量                  | 作用                                                       | 优先级                                |
| ------------------------- | ---------------------------------------------------------- | ------------------------------------- |
| `WATCHOWL_DB_PASSWORD`    | 数据库连接口令，同时也是 compose 里 MySQL 的 root 口令     | 高于 `config.yaml` 的 `Database.pass` |
| `WATCHOWL_REDIS_PASSWORD` | Redis 连接口令，内置 Redis 未设 requirepass，默认留空      | 高于 `config.yaml` 的 `Redis.pass`    |
| `WATCHOWL_ADMIN_PASSWORD` | 内置超管 admin 的初始口令，仅在库中还没有 admin 用户时生效 | —                                     |
| `WATCHOWL_JWT_SIGNKEY`    | JWT 签名密钥，整套鉴权体系的信任根，建议不少于 32 位       | 高于 `config.yaml` 的 `Jwt.signKey`   |

## 🔧 配置说明

后端配置文件位于 `backend/config/config.yaml`：

```yaml
Server:
  port: "9001"
  mode: "release"        # release / debug / test
  # 允许跨域访问的前端来源白名单, 留空表示只允许同源
  allowedOrigins: []
  # pprof 会暴露内存对象与 goroutine 栈, 仅排障时临时打开, 且只绑回环地址
  enablePprof: false

Database:
  type: mysql            # mysql 或 sqlite
  host: watch-owl-mysql
  port: 3306
  user: root
  pass: ""               # 留空, 由环境变量 WATCHOWL_DB_PASSWORD 注入
  dbName: watchowl
  timeout: 10s
  # SQLite 配置 (type 为 sqlite 时使用)
  path: data/watchowl.db

Redis:
  host: watch-owl-redis
  port: 6379
  pass: ""               # 留空, 需要口令时用 WATCHOWL_REDIS_PASSWORD 注入
  database: 0

Jwt:
  expire: 18000
  signKey: ""            # 留空, 由环境变量 WATCHOWL_JWT_SIGNKEY 注入

# 登录失败限流, 挡住针对口令的在线暴力破解
Login:
  maxFailures: 5         # 连续失败次数阈值
  lockMinutes: 15        # 达到阈值后的锁定时长(分钟)
```

> 口令与密钥类配置一律留空、改用 `WATCHOWL_` 开头的环境变量注入：Helm 部署时 `config.yaml` 会被渲染成
> ConfigMap（明文存储，凡有 `get configmap` 权限的人都能读到），而环境变量可以由 Secret 提供。
> Helm 侧的配置方式见下方「Helm 部署」一节。

### 两个需要显式收口的配置项

**`Server.metricsToken`（默认留空 = `/metrics` 不鉴权）**

`enableMetrics: true` 时 `/metrics` 会暴露服务自身的运行指标。默认值留空意味着**不做任何鉴权**，
任何能访问到 `9001` 端口的人都能读取。指标本身不含业务数据，但会泄露实例数量、规则规模、
接口调用量等运行侧信息，可用于摸清系统规模。处置方式二选一：

```yaml
Server:
  # 方式一: 显式设置令牌, 抓取端需携带 Authorization: Bearer <token>
  metricsToken: "换成一段随机长字符串"
```

```yaml
# 方式二: 保持留空, 但在网关/反代层限制 /metrics 的来源网段, 只放行 Prometheus
```

只要服务不是纯内网可达，就必须选一种；两种都不做等于把 `/metrics` 挂在公网上。
令牌比对走 `crypto/subtle` 的常量时间比较，不存在按响应时间逐字节猜测的问题。

**Prometheus 服务发现接口的 `apiKey` 查询参数**

`GET /api/watchowl/prometheus/targets` 是给采集端直连的接口，凭据按
`X-API-Key` 请求头 → `Authorization` 请求头 → `apiKey` 查询参数的顺序读取。
`apiKey` 走 query 只是为了兼容不支持自定义请求头的旧版 Prometheus（`http_headers` 需 2.51+），
**代价是密钥会落到访问日志里** —— nginx 的 `$request_uri`、Ingress Controller 日志、
APM 链路追踪的 URL 字段都会原样记下它，任何能看日志的人都能捡到一枚长期有效的密钥。

因此：

- Prometheus 2.51 及以上一律用请求头，不要用 query 参数：

  ```yaml
  http_sd_configs:
    - url: https://watchowl.example.com/api/watchowl/prometheus/targets?tenantId=<租户ID>
      http_headers:
        X-API-Key:
          value: <API 密钥>
  ```

- 确实只能走 query 参数时，在反代层把它从日志中摘掉，例如 nginx 用
  `map` 改写 `$request_uri` 后再写入 `log_format`，并给该密钥设置较短的有效期、定期在
  「API 密钥」页面轮换（`rotate` 接口支持原地轮换，不需要重建）。

该接口除校验密钥有效性外，还会核对密钥持有人是否属于 URL 中声明的 `tenantId`，
所以一枚泄露的密钥不会连带读到其他租户的采集目标，但它自己那个租户的目标清单仍会全部暴露。

### Alertmanager 管理

在“告警管理 → Alertmanager”中可添加实例地址、测试连接、查看活动告警，并创建或删除 Alertmanager 静默。实例按当前租户隔离。

出于 SSRF 防护，默认只允许访问公网地址，且不跟随重定向。若 Alertmanager 部署在 Kubernetes 或内网，请在 `backend/config/config.yaml` 明确放行所需主机：

```yaml
Alertmanager:
  allowedHosts:
    - alertmanager.monitoring.svc.cluster.local
  allowPrivateNetworks: true
  requestTimeout: 10s
```

`allowedHosts` 支持精确域名和 `*.cluster.local` 这类后缀通配符。请只放行实际使用的 Alertmanager 域名；不要为了方便配置为宽泛公网域名。

### Webhook 适配器

如果你要把告警统一发到钉钉、飞书、企业微信或卡片消息，建议让 `webhook_configs` 统一指向集群内的 `webhook-adapter`，再由适配器按平台组装不同参数。

```yaml
- name: 'platform-webhook'
  webhook_configs:
    - url: 'http://webhook-adapter.monitoring.svc.cluster.local:8080/api/v1/notify'
      send_resolved: true
      http_config:
        headers:
          X-Alert-Platform: 'dingtalk'
          X-Alert-Channel: 'ops'
          X-Alert-Card: 'markdown'
```

在前端里也可以直接用“通知对象 → WebHook”或“Alertmanager → 编辑配置”里的预设按钮生成这段配置。常用平台约定如下：

- 钉钉：`X-Alert-Platform=dingtalk`
- 飞书：`X-Alert-Platform=feishu`
- 企业微信：`X-Alert-Platform=wecom`
- 卡片：`X-Alert-Card=card` 或 `interactive`

## 📖 功能使用教程

WatchOwl 是一套多租户的监控告警平台，核心链路是「数据源 → 规则 → 告警 → 通知」。
下面按日常使用顺序介绍各模块。

### 1. 登录与初始配置

- 默认管理员账号为 `admin`，初始口令由环境变量 `WATCHOWL_ADMIN_PASSWORD` 注入；
  新安装时未配置该变量，服务会拒绝创建默认管理员并终止启动。
- 登录后进入「系统设置」配置基础参数：认证方式（本地 / LDAP / OIDC）、JWT 过期时间、
  登录失败限流（连续 5 次失败锁定 15 分钟）。
- 顶部租户切换器选择当前工作租户，所有数据按租户隔离。

### 2. 租户与用户管理

- **租户**：「租户管理」创建租户后，在租户详情中添加成员并分配角色。
  预置三种角色：超级管理员（内置，不落表）、运维（业务全权，不含人员/租户/系统设置）、
  只读（仅查看类接口）。角色权限可在「角色管理」中按需调整。
- **用户**：「人员管理」创建用户、重置密码、启用/停用值班。用户可属于多个租户，
  不同租户下可分配不同角色。
- **API 密钥**：「个人中心 → API 密钥」签发长期密钥，供 Prometheus 服务发现、
  自动化脚本等非交互场景使用。密钥支持 `rotate` 原地轮换，泄露后立即轮换即可作废。

### 3. 数据源接入

「数据源管理」支持 Prometheus、VictoriaMetrics、Loki、Elasticsearch、AWS CloudWatch 等。
添加时填写地址、认证信息（Basic Auth / Bearer Token），点「测试连接」验证连通性。

- 数据源按租户隔离，只在当前租户下可见。
- Prometheus 类数据源可在「Prometheus 查询」页面直接写 PromQL 做临时排查。

除可观测性数据源外，WatchOwl 还内置了 **数据库与中间件** 数据源，用于把「业务 / 组件自身的指标」也纳入告警：

| 数据源     | 部署模式              | 采集方式                                                     |
| ---------- | --------------------- | ------------------------------------------------------------ |
| MySQL      | 单机 / 集群（多节点） | 执行 SQL 取标量（首行首列数值）                              |
| PostgreSQL | 单机 / 集群（多节点） | 执行 SQL 取标量                                              |
| SQL Server | 单机                  | 执行 SQL 取标量（`encrypt=disable`，连接 / 查询超时 5s）     |
| Redis      | 单机 / 哨兵 / Cluster | `INFO` 指标 key；Cluster 另支持 `cluster_*` 专项指标         |
| MongoDB    | 单机 / 副本集         | `serverStatus` 点号路径（如 `connections.current`）+ `replset.*` 副本集专项 |
| Kafka      | 集群                  | Metadata：`brokers` / `topics` / `partitions` / `offline_partitions` |
| RabbitMQ   | 单机                  | Management API `/api/overview`：`messages` / `connections` / `queues` 等 |
| RocketMQ   | 集群                  | NameServer TCP 探活：`nameservers_alive` / `nameservers_total` |
| Etcd       | 集群                  | `/health` 健康探测 + `/metrics` 文本解析，指标按 Prometheus 指标名选取 |

配置集群 / 哨兵 / 副本集时，地址按「多行文本」逐行填写（如 `host1:3306`、`host2:3306`）；
Redis 哨兵模式需额外填写 `masterName`。添加完成后可在「指标预览」直接拉取该数据源的
**全部可用指标 key**，无需手工记忆指标名。

### 4. 告警规则

规则分「规则组 → 规则」两级管理：

- **规则组**：一组规则的集合，对应 Prometheus 的 `rule_groups`。可选择数据源、
  设置评估间隔、所属租户。
- **规则**：PromQL 表达式 + 持续时长（`for`）+ 告警等级 + 标签 + 注解。
  支持从模板批量导入（「规则模板」维护可复用的规则片段）。
- 规则变更后 WatchOwl 会把规则下发到对应数据源（Prometheus 通过 `/api/v1/rules`
  的 reload，或写入规则文件后触发热加载）。

#### 中间件与数据库规则

除 PromQL 规则外，规则类型里还提供 MySQL / PostgreSQL / SQL Server / MongoDB / Kafka /
RabbitMQ / RocketMQ / Etcd / Redis 等「中间件指标」规则，评估语义与 Zabbix 一致：
**阈值表达式 + 连续达标次数**。

以 MySQL 为例：

- **订单失败率**：写一段返回标量的 SQL，例如
  `SELECT ROUND(SUM(status='failed')/COUNT(*)*100, 2) FROM orders`；
  阈值表达式填 `>= 5`、评估周期 60s、连续 2 次 —— 连续两个周期失败率 ≥ 5% 才触发。
- **连接数占比**：
  `SELECT ROUND(COUNT(*)/@@max_connections*100, 2) FROM information_schema.processlist`，
  阈值 `> 80`。
- **Redis**：填 `INFO` 中的指标 key（如 `connected_clients`、`used_memory`、`keyspace_hits`），
  阈值表达式与连续达标次数同上。集群模式还可用专项指标：`cluster_state`（ok=1）、
  `cluster_slots_assigned` / `cluster_slots_ok` / `cluster_slots_pfail` / `cluster_slots_fail`（槽位分配）、
  `cluster_known_nodes` / `cluster_size`（集群规模）、`cluster_nodes_failed`（异常节点数）；
  在非集群数据源上访问这些 key 会明确报「仅集群模式支持」。
- **MongoDB**：支持 `connections.current` 之类的点号路径，以及 `replset.members_healthy` /
  `replset.members_unhealthy` 等副本集健康度指标。

评估周期与「连续达标次数」共同决定告警灵敏度：周期内达标才累加计数、未达标即清零，
避免瞬时抖动造成误报。

### 5. 通知对象

「通知对象」定义告警发到哪里：

- **邮件**：SMTP 配置在「系统设置」，通知对象只填收件人。
- **钉钉 / 飞书 / 企业微信**：填 Webhook 地址与加签密钥，飞书支持交互式卡片。
- **WebHook**：自定义 HTTP 回调，可在请求头中指定平台类型，由 `webhook-adapter`
  统一适配（见上文 Webhook 适配器小节）。

通知对象按租户隔离，可在规则注解中通过 `notice_id` 指定接收人。

### 6. 静默与抑制

- **静默（Silence）**：临时屏蔽某段时间内的告警，按标签匹配，到期自动恢复。
  在「告警管理 → 静默」创建，或在告警详情页一键静默。
- **抑制规则（Inhibit）**：高级别告警触发时自动抑制低级别告警，避免告警风暴。
  例如「实例宕机」抑制该实例上所有「服务异常」告警。

### 7. 值班管理

- **值班组**：定义一组值班人员与排班规则（按天 / 按周轮换）。
- **值班日历**：可视化查看每日值班人，支持手动调整。
- 告警通知时会自动找到当前值班人，按其联系方式发送。

### 8. 拨测（Probing）

主动探测目标可用性，无需 Prometheus 即可覆盖 HTTP / TCP / ICMP 等协议：

- 「拨测管理」创建探测任务，设置目标地址、间隔、超时、失败阈值。
- 探测失败自动生成告警，走与规则告警相同的通知链路。

### 9. 订阅

「订阅管理」让用户按标签订阅自己关心的告警，只收到匹配的通知，无需管理员介入。
支持按告警等级、标签键值对过滤。

### 10. 仪表盘

- **自定义仪表盘**：拖拽式编排 PromQL 图表，支持折线图、表格、单值、状态面板。
  仪表盘按租户隔离，可设为公共或私有。
- **故障中心**：汇总当前所有未恢复告警，按等级、服务、租户筛选，支持批量处理。

### 11. 审计日志

「审计日志」记录所有写操作（创建 / 更新 / 删除 / 登录 / 改密等），含操作人、IP、
租户、操作对象、变更前后快照。口令等敏感字段在日志中自动脱敏。审计日志不可删除，
仅超级管理员可查。

### 12. 运维管理

「运维管理」是面向一线运维的作业面，覆盖 Kubernetes、主机、数据库与中间件：

- **集群管理**：KubeConfig 方式纳管多个 Kubernetes 集群。多集群总览一屏看清
  各集群版本、节点 / Pod / 工作负载规模与告警数；单集群提供概览、节点、工作负载、
  Pod、服务与网络、配置与存储、RBAC、弹性伸缩（HPA/VPA/KEDA）、CRD、资源拓扑、
  命名空间、事件等完整管理能力，工作负载向导式创建（提交审批后自动下发）。
- **集群终端**：多集群总览的「终端」按钮从底部弹出 kubectl 终端面板（可放大全屏），
  会话锁定目标集群；高危操作执行审批、终端会话录制均沿用平台既有安全设施。
- **主机管理**：SSH 主机资产纳入管理，支持凭据托管、批量命令、批量改密、
  电源操作（BMC），并可作为剧本编排的执行目标。
- **剧本编排**：Ansible Playbook 剧本库，选主机执行；支持高危剧本的定时任务与补投。
- **中间件管理**：MySQL、Redis、PostgreSQL、Oracle、MongoDB、Nacos、Kafka-ZK
  统一纳管——SQL 查询（只读 / 写模式）、慢会话治理、MySQL 用户权限管理（建号 /
  授权 / 回收，系统账号保护）、Redis 键管理等。
- **执行审批**：所有高危操作（剧本执行、负载下发、数据库写操作、Redis 写、
  集群运维、管理删除、平台配置变更等）可在「系统设置 → 执行审批策略」中逐项开关；
  开启即「先审批后自动执行」，关闭则免审直行并记审计。

### 常见操作速查

| 操作          | 入口                              |
| ------------- | --------------------------------- |
| 改自己密码    | 个人中心 → 修改密码               |
| 重置他人密码  | 人员管理 → 重置密码（需权限）     |
| 轮换 API 密钥 | 个人中心 → API 密钥 → 轮换        |
| 切换租户      | 顶部租户切换器                    |
| 导入规则      | 规则组 → 导入（支持 YAML / JSON） |
| 查看告警历史  | 告警管理 → 历史告警               |

## 🛠️ 从源码构建

### 后端

```bash
cd backend
# 直接运行
make run
# 编译当前平台
make build
# 交叉编译 Linux amd64 / arm64
make build-linux       # 含 upx 压缩
make build-linux-arm
```

### 前端

```bash
cd frontend
yarn install
yarn start             # 开发模式，默认 http://localhost:3000（kind 集群访问入口为 http://localhost:30593）
yarn build             # 生产构建，产物在 build/
```

## ☸️ Kubernetes / Helm 部署

> **不想用 Helm？** 项目提供了与 chart 语义等价的单文件原生清单
> [backend/deploy/yaml/watchowl-deploy.yaml](./backend/deploy/yaml/watchowl-deploy.yaml)，
> 包含 Namespace / Secret / ConfigMap / MySQL / Redis / 后端 / 前端全套资源，
> 修改其中 Secret 的占位口令后一条命令即可部署：
>
> ```bash
> kubectl apply -f backend/deploy/yaml/watchowl-deploy.yaml
> # 访问入口: http://<节点IP>:30593 (NodePort, 与 Helm 部署一致)
> ```
>
> 部署后务必把 `watchowl-secrets` 里的占位密钥换成自己的随机值，或改用
> sealed-secrets / external-secrets / vault 接管（该清单不能与下方 Helm 部署混用在同一集群）。

### 1. 创建命名空间与密钥

统一使用 `watchowl` 命名空间部署：

```bash
kubectl create namespace watchowl

# 口令与密钥先放进 Secret, 不要写在 values.yaml 里
# (values.yaml 的 config 段会渲染成 ConfigMap, 明文存在里面等于公开)
kubectl -n watchowl create secret generic watchowl-secrets \
  --from-literal=mysql-root-password="$(openssl rand -hex 16)" \
  --from-literal=adminPassword="$(openssl rand -hex 12)" \
  --from-literal=jwtSignKey="$(openssl rand -hex 32)" \
  --from-literal=datasourceEncryptionKey="$(openssl rand -hex 32)"
```

| Secret key                | 用途                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `mysql-root-password`     | 内置 MySQL root 口令，同时也是 WatchOwl 连库口令（单点真源） |
| `adminPassword`           | admin 初始口令，仅库中还没有 admin 用户时生效                |
| `jwtSignKey`              | JWT 签名密钥，整套鉴权体系的信任根，建议不少于 32 位         |
| `datasourceEncryptionKey` | 数据源口令等敏感字段落库的加密密钥（可选，建议配置）         |

### 2. 安装

```bash
cd backend/deploy/helmchart

# 按需修改 values.yaml 中的镜像仓库、副本数、ingress 域名等, 再指向上面的 Secret
helm install watchowl . -n watchowl \
  --set mysql.auth.existingSecret.name=watchowl-secrets \
  --set service.adminPasswordSecret.name=watchowl-secrets \
  --set service.jwtSignKeySecret.name=watchowl-secrets \
  --set service.datasourceEncryptionKeySecret.name=watchowl-secrets
```

### 3. 查看状态与访问

```bash
kubectl -n watchowl get pods        # 等待全部 Running
kubectl -n watchowl get svc         # 查看各组件 Service

# 安装完成后 helm 会输出访问方式, 也可手动获取 NodePort 入口:
export NODE_PORT=$(kubectl -n watchowl get svc watchowl-web -o jsonpath="{.spec.ports[0].nodePort}")
export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
echo "http://${NODE_IP}:${NODE_PORT}"
```

默认 `web.service.type=NodePort`（端口随机分配）。可按需调整：

```bash
# 固定 NodePort
--set web.service.nodePort=30080

# 或走 Ingress
--set web.ingress.enabled=true \
--set web.ingress.className=nginx \
--set web.ingress.hosts[0].host=watchowl.example.com
```

### 4. 组件说明

Helm Chart 会自动拉起以下组件，数据库表结构由后端启动时自动迁移完成，无需额外初始化步骤：

| 组件                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| `watchowl-service`  | 后端 API（Gin），支持多副本                                  |
| `watchowl-web`      | 前端（Nginx），对外访问入口                                  |
| `watchowl-mysql`    | 内置 MySQL 8.0（StatefulSet，数据落 PVC）                    |
| `watchowl-redis`    | 内置 Redis（告警事件缓存、限流等）                           |
| `watchowl-terminal` | 云鸮终端 Agent（可选，`service.terminal.agent.enabled` 控制） |

### 5. 升级与卸载

```bash
# 升级（修改 values 后重新应用）
helm upgrade watchowl . -n watchowl -f values.yaml

# 查看历史 / 回滚
helm history watchowl -n watchowl
helm rollback watchowl <revision> -n watchowl

# 卸载（Secret/PVC 不会被删除, 需要的话手动清理）
helm uninstall watchowl -n watchowl
```

关键值见 [values.yaml](./backend/deploy/helmchart/values.yaml)；本地 Kind 测试可参考 [values-kind.yaml](./backend/deploy/helmchart/values-kind.yaml)（镜像为阿里云 ACR 仓库 `registry.cn-beijing.aliyuncs.com/deanmr/watchowl`，命名空间同样为 `watchowl`）。

口令相关的值有一套统一约定，可用 `helm template` 先确认渲染结果里没有明文：

| values 项                                               | 说明                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| `mysql.auth.password`                                   | 内置 MySQL 的 root 口令，**同时也是 WatchOwl 连库的口令**（单点真源）。默认是仅供本地体验的弱口令 |
| `mysql.auth.existingSecret.name`                        | 填了它之后 `mysql.auth.password` 不再生效，后端 Deployment、MySQL StatefulSet、SQL 导入 Job **三处同时**改走 `secretKeyRef` |
| `service.dbPassword` / `service.dbPasswordSecret`       | 对接外部数据库时用，优先级高于 `mysql.auth`；两项都留空且启用了内置 MySQL 时自动复用 `mysql.auth` |
| `service.redisPassword` / `service.redisPasswordSecret` | Redis 口令，内置 Redis 未设 requirepass，默认留空            |
| `service.adminPassword` / `service.adminPasswordSecret` | admin 的初始口令，仅在库中还没有 admin 用户时生效            |
| `service.enableOpenRegister`                            | 免鉴权的注册接口，**默认关闭**，正常建号请走「人员管理」页面 |

## 📚 项目文档

- 功能介绍：https://watch-owl.github.io/docs/
- 快速部署：https://watch-owl.github.io/docs/install/docker.html

## 🎉 项目预览

- 演示环境：https://watchowl.deanit.cn
- 演示账号：`demo`
- 演示密码：`WatchOwl@2026`

> 演示环境数据每日重置，仅提供只读级别体验，请勿在其中接入生产数据源。

## 🎸 支持

- 如果你觉得 WatchOwl 还不错，可以通过 Star 来表示你的喜欢
- 在公司或个人项目中使用 WatchOwl，并帮忙推广给伙伴使用

## 🤝 贡献

- 贡献指南见 [CONTRIBUTING.md](./CONTRIBUTING.md)

## 🧑‍💻 交流渠道

- [点击我](https://watch-owl.github.io/docs/#%E4%BA%A4%E6%B5%81%E7%BE%A4-%E8%81%94%E7%B3%BB%E6%88%91)

## 📄 License

- [Apache-2.0](
