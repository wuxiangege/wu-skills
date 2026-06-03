# 技术栈检测签名速查

Agent 扫描时按「依赖 → 目录 → import → 配置 → 调用点」顺序匹配。多条命中则置信度 ✅。

---

## Go 生态

### go-zero

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/zeromicro/go-zero` in `go.mod` |
| API 定义 | `*.api` 文件，`syntax = "v1"`，`@server`、`@handler`、`service api` |
| 生成代码 | `internal/handler/`、`internal/logic/`、`internal/svc/servicecontext.go` |
| 配置 | `etc/*-api.yaml`，含 `Name`、`Host`、`Port` |
| 启动 | `*.go` 中 `rest.MustNewServer`、`conf.MustLoad` |

### GORM + MySQL

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `gorm.io/gorm`、`gorm.io/driver/mysql` |
| 扩展 | `gitlab.cdfsunrise.com/library/gormx`、`gormextension` |
| 代码 | `*gorm.DB`、`db.WithContext(ctx)`、`AutoMigrate` |
| Repository | `ddd/repository/*repository/mysql.go` |

### shopspring/decimal

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/shopspring/decimal` |
| import | `"github.com/shopspring/decimal"` |
| 用法 | `decimal.NewFromString`、`decimal.NewFromInt`、金额字段类型 `decimal.Decimal` |

### go-linq

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/ahmetb/go-linq/v3` |
| import | `linq "github.com/ahmetb/go-linq/v3"` 或 `"github.com/ahmetb/go-linq/v3"` |
| 用法 | `linq.From(slice).Where(...).SelectT(...).ToSlice(&dest)` |

### Google Wire

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/google/wire` |
| 文件 | `wire.go`（`//go:build wireinject`）、`wire_gen.go` |
| 用法 | `wire.Build(...)`、`wire.NewSet` |

### cron（本地定时）

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/robfig/cron/v3` |
| 用法 | `cron.New()`、`cron.AddFunc`、`cron.Remove` |

### XXL-Job

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/xxl-job/xxl-job-executor-go` |
| 目录 | `internal/xxljob/` |
| 用法 | 实现 `xxl.TaskFunc`、`executor` 注册 |

### RocketMQ

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/apache/rocketmq-client-go/v2` |
| 目录 | `ddd/infrastructure/rocketmq/` |
| 用法 | `consumer`、`producer`、`PushConsumer` |

### gRPC

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `google.golang.org/grpc` |
| 代码 | `.pb.go`、`grpc.Dial`、`*Client` stub |

### Redis

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/redis/go-redis/v9` |
| 配置 | yaml 中 `Redis:` 段 |
| 用法 | `redis.NewClient`、`rdb.Get/Set` |

### Apollo 配置中心

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/apolloconfig/agollo/v4` 或项目内 apollo 封装 |
| 配置 | yaml 中 `Apollo:` |

### 单元测试栈

| 信号 | 位置/模式 |
|------|-----------|
| goconvey | `github.com/smartystreets/goconvey/convey`，`Convey`/`PatchConvey` |
| mockey | `github.com/bytedance/mockey`，`Mock(...).Return(...).Build()` |
| miniredis | `github.com/alicebob/miniredis/v2` |

---

## 架构与模式

### DDD 分层

| 信号 | 典型目录 |
|------|----------|
| 领域层 | `ddd/domain/model/`、`ddd/domain/service/`、`ddd/domain/valueobj/` |
| 应用层 | `ddd/app/` |
| 仓储 | `ddd/repository/` |
| 基础设施 | `ddd/infrastructure/clients/`、`ddd/infrastructure/rocketmq/` |
| 枚举 | `ddd/enum/` |

### RESTful API

| 信号 | 说明 |
|------|------|
| 路径前缀 | `prefix: api/v1` in `.api` |
| 资源化路径 | `/settlementbill/`、`/settlementbilldetails/` |
| HTTP 动词 | `get`/`post`/`put`/`delete` in `.api` |
| 注意 | 很多查询接口用 `post .../query`，属「POST 承载查询体」的类 REST 风格 |

### WebSocket

| 信号 | 位置/模式 |
|------|-----------|
| 依赖 | `github.com/gorilla/websocket`、`nhooyr.io/websocket`、`github.com/coder/websocket` |
| Go 标准库 | `net/http` + `Upgrade: websocket`；`golang.org/x/net/websocket`（较旧） |
| 前端 | `new WebSocket(url)`、`socket.io-client`（封装层，底层可能仍是 WS） |
| 路由 | `/ws`、`/websocket` 路径；`.api` 中少见，多在 handler 手动注册 |
| 框架 | go-zero `rest` 自定义路由；Gin `GET("/ws", wsHandler)`；Spring `@ServerEndpoint` |
| 特征 | `Upgrader`、`ReadMessage`/`WriteMessage`、`Conn`、`Ping`/`Pong` |

### SSE（Server-Sent Events）

| 信号 | 位置/模式 |
|------|-----------|
| 响应头 | `Content-Type: text/event-stream`、`Cache-Control: no-cache`、`Connection: keep-alive` |
| 写法 | `fmt.Fprintf(w, "data: %s\n\n", payload)`、`event:` / `id:` / `retry:` 字段 |
| 前端 | `EventSource(url)`、`onmessage` |
| 框架 | Gin `c.Stream`；Spring `SseEmitter` / `Flux` + `TEXT_EVENT_STREAM` |
| 与 WebSocket 区分 | SSE **单向**（服务端→客户端）、基于 HTTP、自动重连；适合推送通知、AI 流式输出 |
| 特征 | 长连接 HTTP GET；无 `Upgrade` 头 |

### HTTP 客户端（服务间通讯）

| 信号 | 位置/模式 |
|------|-----------|
| 目录 | `ddd/infrastructure/clients/*/http.go` |
| 用法 | `http.Client`、`resty`、项目封装 `HttpClient` |
| 对比 gRPC | 同目录下可能有 `grpc` 子包 |

### Repository 模式

| 信号 | 位置/模式 |
|------|-----------|
| 接口 | `ddd/repository/*/repository.go` 定义 `Repository interface` |
| 实现 | `mysql.go`、`mongo.go` |
| 命名 | `NewMySQLRepository`、`Query(ctx, options)` |

### State Machine（状态机）

| 信号 | 位置/模式 |
|------|-----------|
| 命名 | `*StateMachine`、`UpdateApplyTaskStatus` |
| 文件 | `*statemachine.go` |
| 配合 | `ddd/enum/` 中状态枚举 |

### Factory / Builder

| 信号 | 位置/模式 |
|------|-----------|
| Factory | `NewXxxFactory`、`CreateXxx` |
| Builder | `*builder.go`、`Build()` 链式组装 |

### Strategy

| 信号 | 位置/模式 |
|------|-----------|
| 接口 + 多实现 | 同接口多个 `impl`/`mode` 子包（如 `standard_mode`、`detailed_mode`） |
| 选择 | switch 或 map 按类型选实现 |

---

## 其他语言（按需扩展）

### Java / Spring

`pom.xml` / `build.gradle`：`spring-boot-starter-web`、`mybatis-spring-boot-starter`

### Node / TypeScript

`package.json`：`express`、`@nestjs/core`、`typeorm`、`prisma`

### Python

`requirements.txt` / `pyproject.toml`：`fastapi`、`django`、`sqlalchemy`

---

## 扫描命令参考（Agent 内部使用）

```bash
# 依赖
rg -l "zeromicro/go-zero|gorm.io/gorm|shopspring/decimal|go-linq" go.mod

# 目录
ls ddd/ api/ internal/

# import 统计
rg "github.com/shopspring/decimal" --glob "*.go" | head

# API 路由
rg "@(get|post|put|delete)" api/apifiles/*.api
```
