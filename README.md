# Traefik 网关配置示例

项目是根据  https://github.com/traefik/traefik  v3.6.14 版本的源码，进行阅读整理出来的配置示例，包含了 HTTP、TCP、UDP 代理，动态路由，中间件，访问日志，健康检查等能力的演示和维护。

这是一个基于 Docker Compose 的 Traefik 网关配置仓库，用于演示和维护 HTTP、TCP、UDP 代理、动态路由、中间件、访问日志和健康检查等能力。

## 项目结构

```text
.
├── docker-compose.yml                 # Traefik 容器启动配置
├── etc/traefik.toml                   # 本地/示例环境静态配置
├── providers/files/                   # 本地/示例环境动态配置
├── prod/
│   ├── traefik.toml                   # 生产环境静态配置
│   └── providers/                     # 生产环境动态配置
├── whoami/                            # whoami 测试服务
├── mysql/                             # MySQL TCP 代理测试服务
├── docs/                              # 架构图、访问日志说明、Traefik 学习笔记
└── clear.sh                           # 清理日志脚本
```

## 核心概念

Traefik 的请求链路可以理解为：

```text
EntryPoints -> Routers -> Middlewares -> Services
```

- `EntryPoints`：监听端口，例如 `web=:80`、`websecure=:443`、`mqtt=:8883`、`mysql=:3366`。
- `Routers`：根据 `Host`、`PathPrefix`、`ClientIP`、`HostSNI` 等规则匹配请求。
- `Middlewares`：在转发前后处理请求，例如限流、IP 白名单、路径裁剪、压缩、认证、重试。
- `Services`：真实后端服务地址，支持负载均衡、权重、健康检查、粘性会话等。
- `Providers`：配置来源，本项目主要使用 `file` provider，也保留 Docker provider 示例。

## 快速启动

### 1. 启动 Traefik

```bash
docker compose up -d
```

根目录 `docker-compose.yml` 使用 host 网络模式，并挂载以下路径：

- `etc/traefik.toml` -> `/etc/traefik/traefik.toml`
- `providers/` -> `/providers`
- `log/` -> `/var/log`
- `/var/run/docker.sock` -> `/var/run/docker.sock`

### 2. 启动测试服务

```bash
docker compose -f whoami/docker-compose.yml up -d
docker compose -f whoami/docker-compose-myapp.yml up -d
```

测试服务会分别暴露：

- `whoami`: `802 -> 3000`
- `myapp`: `8024 -> 3000`

### 3. 查看健康检查

```bash
curl http://127.0.0.1/ping
```

正常返回：

```text
OK
```

### 4. 访问控制面板

当前配置开启了 `api.insecure = true`，Traefik 会暴露 Dashboard 和 API：

```text
http://127.0.0.1:8080/dashboard/
http://127.0.0.1:8080/api/overview
```

注意：`insecure = true` 只适合本地或内网测试，生产环境应关闭或加认证、白名单保护。

## 配置说明

### 静态配置

静态配置在 Traefik 启动时加载，主要包括入口点、日志、API、Provider 等。

本地/示例环境：

```text
etc/traefik.toml
```

生产环境：

```text
prod/traefik.toml
```

常用入口点：

| 名称 | 端口 | 说明 |
| --- | --- | --- |
| `web` | `80` | HTTP 入口 |
| `websecure` | `443` | HTTPS 入口 |
| `mqtt` | `8883` | MQTT/TCP 示例入口 |
| `mysql` | `3366` | 生产配置中的 MySQL TCP 入口 |

### 动态配置

动态配置通过 `providers.file` 加载，支持热更新。

本地/示例环境：

```toml
[providers.file]
directory = "/providers/files"
watch = true
```

生产环境：

```toml
[providers.file]
directory = "/providers"
watch = true
```

重要规则：`providers.file` 目录中只要有一个动态配置文件存在语法或字段错误，Traefik 可能会拒绝加载整个目录中的动态配置。修改配置后建议立即查看 Dashboard、日志或 `/api/rawdata`。

## HTTP 代理示例

生产环境中的 whoami 路由示例：

```toml
[http.routers.whoami-api]
entryPoints = ["web"]
rule = "Host(`10.0.1.7`)&& PathPrefix(`/whoami`)"
middlewares = ["m-internal-ip-allow","m-ip-rate-limit","m-strip-perfix"]
service = "whoami-api"
```

对应后端服务：

```toml
[http.services.whoami-api.loadBalancer]
strategy = "wrr"
passHostHeader = true

[[http.services.whoami-api.loadBalancer.servers]]
url = "http://10.0.1.7:802"
weight = 2
preservePath = false

[[http.services.whoami-api.loadBalancer.servers]]
url = "http://10.0.1.6:802"
weight = 1
preservePath = false
```

访问示例：

```bash
curl http://10.0.1.7/whoami
```

## TCP 代理示例

`providers/files/tcp-routers.toml` 中包含 MySQL TCP 代理示例：

```toml
[tcp.routers.mysql-router]
entryPoints = ["mysql"]
rule = "HostSNI(`*`)"
service = "mysql-service"
middlewares = ["m-mysql-ip-allow"]

[tcp.routers.mysql-router.tls]
passthrough = true
```

适用于 MySQL、Redis、SSH、PostgreSQL、MQTT 等四层 TCP 流量代理。TCP 路由通常使用 `HostSNI`、`ClientIP`、`IPMatch` 等规则。

注意：根目录 `etc/traefik.toml` 默认没有启用 `mysql` 入口点；如需直接测试该 TCP 示例，请参考 `prod/traefik.toml` 增加：

```toml
[entryPoints.mysql]
address = ":3366"
```

## 常用中间件

本项目已准备多类中间件配置：

| 中间件 | 作用 |
| --- | --- |
| `m-internal-ip-allow` | 只允许内网或指定 IP 段访问 |
| `m-ip-rate-limit` | 按 IP 限流，默认 `100 req/s`，突发 `50` |
| `m-strip-perfix` / `m-strip-pefix` | 删除路径前缀后再转发 |
| `m-add-pefix` | 给转发路径增加前缀 |
| `m-basic-auth` | 基础认证 |
| `m-oidc-auth` | ForwardAuth 外部认证 |
| `m-compress` / `m-rep-compress` | 响应压缩 |
| `retry-3x` | 请求失败自动重试 |
| `m-upload-100mb` | 大文件上传缓冲限制 |

提示：当前示例配置中存在 `pefix`、`perfix` 这类历史命名，使用时以具体配置文件里的实际名称为准。

## 日志

日志挂载到宿主机 `log/` 目录：

```text
log/traefik.log
log/access-log.log
```

访问日志使用 JSON 格式，并保留了常用字段：

- 客户端 IP
- 请求方法、域名、路径
- 状态码
- 路由名称、入口点、服务名称
- 请求耗时
- 重试次数
- `User-Agent`、`Referer`

清理日志：

```bash
./clear.sh
```

Windows PowerShell 下可手动执行：

```powershell
Remove-Item -Force log/access-log.log, log/traefik.log
```

## 生产配置使用建议

`prod/` 目录已经按生产配置拆分出更细的路由、服务和中间件文件，例如：

```text
prod/providers/http-routers.toml
prod/providers/http-services.toml
prod/providers/http-middlewares-ip-allow.toml
prod/providers/http-middlewares-ip-rate-limit.toml
prod/providers/http-routers-myapp.toml
prod/providers/http-services-myapp.toml
```

建议生产环境按以下方式维护：

1. 一个业务服务对应独立的 router/service 文件。
2. 通用中间件单独维护，业务路由只引用名称。
3. 新增或修改动态配置后，先在测试环境验证语法和 Dashboard 状态。
4. 关闭或保护 `api.insecure`、`debug` 等调试入口。
5. 对外服务默认启用 IP 白名单、限流、访问日志。

## 排障清单

### 路由没有生效

- 检查 `providers.file.directory` 是否挂载正确。
- 检查动态配置目录中是否有任意一个 TOML 文件语法错误。
- 检查 Dashboard 中 router、middleware、service 是否为绿色状态。
- 检查 router 的 `entryPoints`、`rule`、`service` 名称是否匹配。

### 请求返回 404

- 可能没有任何 router 命中。
- 可能被 `ipAllowList.rejectStatusCode = 404` 拒绝。
- 检查 `Host`、`Path`、`PathPrefix` 是否和实际请求一致。

### 后端访问失败

- 检查 service 中的 `url` 或 TCP `address` 是否能从 Traefik 所在主机访问。
- 检查健康检查路径是否存在，例如 `/api/health`。
- 检查 `stripPrefix`、`addPrefix`、`preservePath` 是否改变了后端实际收到的路径。

### 日志没有写入

- 检查宿主机是否存在 `log/` 目录。
- 检查容器是否正确挂载 `$PWD/log:/var/log`。
- 检查路由是否开启：

```toml
[http.routers.<router-name>.observability]
accessLogs = true
```

## 参考文档

- `docs/介绍.md`
- `docs/docs.md`
- `docs/accesslog.md`
- `docs/4层与7层.md`
- `docs/架构图.png`
- `docs/请求架构.png`
- `docs/请求时序图.png`
