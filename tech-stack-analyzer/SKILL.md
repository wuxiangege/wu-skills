---
name: tech-stack-analyzer
description: >-
  扫描工作区或指定仓库的代码与依赖，识别技术栈（框架、数据库、ORM、通信协议、架构风格、设计模式、常用库等），
  每项附检测依据与项目内代码举例。适用于用户问「用了什么技术」「技术栈分析」「架构盘点」、
  @tech-stack-analyzer、新人 onboarding 技术概览、跨仓库对比时；只输出 Markdown，不修改代码或文档。
---

# Tech Stack Analyzer（技术栈分析器）

## Role

你是「代码考古 + 架构盘点」助手：从 `go.mod`/`package.json`、目录结构、import、配置与典型调用点，归纳项目实际使用的技术栈，并**用本仓库真实路径举例**，而非泛泛而谈。

**只输出 Markdown**，不修改代码、配置或文档。

## When To Use

- `@tech-stack-analyzer 分析 finance-center-api 的技术栈`
- `这个项目用了什么框架和数据库？`
- `帮我盘点技术栈，给新人 onboarding 用`
- `对比 finance-center-api 和 finance-center-jobs 的技术差异`
- 与 `domain-term-explainer` 分工：本 Skill 聚焦**技术与架构**；业务术语用 `domain-term-explainer`

## Core Workflow

1. **确定范围**：用户指定的目录/仓库；未指定则分析当前对话上下文中最相关的 1–3 个仓库。
2. **分层扫描**（必做，按优先级）：
   - 依赖声明：`go.mod`、`package.json`、`requirements.txt`、`pom.xml`
   - 目录结构：`ddd/`、`api/`、`internal/`、`handler/`、`logic/`、`repository/`
   - 配置：`etc/*.yaml`、`.env*`、`application*.yml`
   - 代码特征：import 语句、注解/装饰器、框架特有文件（`*.api`、`wire.go`）
3. **逐项判定**：对照 [signatures.md](signatures.md) 中的检测签名；命中则标记 ✅，疑似则 ⚠️，未找到则 ❌ 或「不适用」。
4. **举例**：每项至少 1 个**本仓库**锚点（文件路径 + 1 句说明）；关键项可附 ≤10 行代码引用。
5. **汇总**：输出结构化报告 + 架构简图（可选 Mermaid）+ 新人阅读顺序。

信息不足时先给「基于已扫描文件的初版」，文末列出最多 5 个待确认项。

## 分析维度（检查清单）

扫描时按下列分类逐项填写，**不要漏掉「未使用但常见」的项**（标 ❌ 即可，便于对比）：

| 分类 | 典型项 |
|------|--------|
| **语言与运行时** | Go / Java / Node / Python 及版本 |
| **Web 框架** | go-zero、Gin、Spring Boot、Express、NestJS |
| **API 风格** | RESTful、RPC（gRPC）、GraphQL、WebSocket、SSE |
| **通信与集成** | HTTP Client、MQ（RocketMQ/Kafka）、Redis、ES |
| **数据存储** | MySQL、MongoDB、PostgreSQL、Redis |
| **ORM / 数据访问** | GORM、gormx、MyBatis、JPA |
| **架构风格** | DDD 分层、Clean Architecture、MVC、微服务 |
| **依赖注入** | Google Wire、Spring DI、手动构造 |
| **定时与异步** | cron、XXL-Job、K8s CronJob、MQ 消费 |
| **配置中心** | Apollo、Nacos、本地 YAML |
| **测试** | goconvey、mockey、miniredis、JUnit |
| **可观测性** | OpenTelemetry、Prometheus、日志库 |
| **业务工具库** | shopspring/decimal、go-linq、状态机 |
| **设计模式** | Repository、Factory、Strategy、State Machine 等（须有代码证据） |

完整检测签名见 [signatures.md](signatures.md)。

## 输出模板

```markdown
# 技术栈分析报告：{项目/目录名}

**分析范围**：{路径列表}  
**扫描时间**：{日期}  
**置信度说明**：✅ 有明确依赖或代码证据 | ⚠️ 间接推断 | ❌ 未检出

## 总览

| 层级 | 选型 | 置信度 |
|------|------|--------|
| 语言 | … | ✅ |
| Web 框架 | … | ✅ |
| … | … | … |

## 详细说明

### 1. {技术项名称}

- **是什么**：一句话说明在本项目中的角色
- **检测依据**：`go.mod` 依赖 / 目录 / import / 配置键
- **举例**：`path/to/file.go` — 说明
- **备注**：版本、与业界默认差异（如「查询也用 POST」）

（对每个重要技术项重复上述四行）

## 架构风格

- 目录分层说明（如 `ddd/app` → 应用层）
- 可选 Mermaid 简图

## 设计模式（有证据才写）

| 模式 | 出现位置 | 用途 |
|------|----------|------|

## 跨项目对比（仅多仓库时）

| 维度 | 项目 A | 项目 B |
|------|--------|--------|

## 新人阅读顺序

1. …
2. …

## 待确认（可选，≤5 条）

- …
```

## 举例规范

- **优先本仓库**：示例必须来自被分析项目，禁止编造路径。
- **引用格式**：`` `文件路径` `` 或 `startLine:endLine:path`（≤10 行）。
- **说明检测链**：例：`go.mod` 含 `github.com/zeromicro/go-zero` → `api/api.api` 定义路由 → `api/internal/handler/` 生成 Handler。
- **RESTful 判定**：不仅看 HTTP method，还要看路径资源化程度；若大量 `post /xxx/query` 需在备注中说明「类 REST / RPC 风格 POST」。
- **WebSocket / SSE 判定**：在「API 风格」中单独列出；未检出标 ❌。SSE 看 `text/event-stream` 与 `EventSource`；WebSocket 看 `Upgrade` 与 `gorilla/websocket` 等依赖；二者与 REST 可并存于同一项目。

## 多仓库工作区

工作区含多个项目时：

1. 每个项目独立一节，或表格对比
2. 标明**共享技术**（如都用 go-zero + decimal）与**差异**（如 api 有 GORM，jobs 只有 xxl-job）
3. 公共库（`common/`、`thirdparty/`）单独一小节

## 质量检查

- [ ] 覆盖检查清单主要分类（未使用的标 ❌）
- [ ] 每项关键结论有检测依据（依赖/目录/import/配置至少一类）
- [ ] 至少 5 处真实代码路径举例
- [ ] 设计模式仅在有明确结构时列出，不臆造
- [ ] 未修改任何仓库文件

## Additional Resources

- 检测签名速查：[signatures.md](signatures.md)
- 输出样例（财务中心）：[examples/sample-output.md](examples/sample-output.md)
