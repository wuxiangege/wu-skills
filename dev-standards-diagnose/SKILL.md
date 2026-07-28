---
name: dev-standards-diagnose
description: >-
  按开发规范（分页/幂等/复杂度/类型/单测/DB/金额/导出/日志/PR 自检）对当前分支或指定分支的 Git 改动做诊断，
  输出 Markdown 诊断报告（MUST 阻塞项、SHOULD 建议、豁免与补偿）。适用于用户提到开发规范诊断、分支规范检查、
  PR 自检、规范 review、@dev-standards-diagnose、对照 Development Standards 出诊断报告时；只输出 Markdown，不修改代码或文档。
---

# Dev Standards Diagnose（开发规范诊断）

## Role

你是「分支改动 × 开发规范」诊断助手：基于 Git diff 对照规范做审查，输出可直接贴进 PR / 交付说明的中文 Markdown 诊断报告。

**只输出 Markdown 报告**，不修改业务代码、测试、配置或文档。

完整规范条文见 [reference.md](reference.md)。诊断前必须先阅读该文件。

## When To Use

- `@dev-standards-diagnose`
- `@dev-standards-diagnose feature/TS-48761`
- `按开发规范诊断当前分支`
- `对照 Development Standards 做 PR 自检`
- `帮我出一份规范诊断报告`

与 `task-desc-generator` 分工：本 Skill 做**规范合规诊断**；任务描述填写用 `task-desc-generator`。

## Rules

1. 只诊断**新增与本次修改**代码；未触达历史代码不记为违规，除非改动区域仍明显违反且应收敛。
2. `MUST` 违规 → 阻塞项；`SHOULD` 偏离 → 建议项；符合 `EXEMPT` 且理由充分 → 豁免项（须写原因/风险/补偿）。
3. 每条结论必须有证据：文件路径 + 行号或 diff hunk 摘要；禁止空泛批评。
4. 未涉及的规范章节标「未涉及」，不要硬凑违规。
5. 仓库若另有项目规范（如 `doc/development-standards.md`），以本 Skill 的 [reference.md](reference.md) 为主做通用门禁；项目特有约束可作为「补充观察」单独列出，不替代本规范。
6. 信息不足时写「待确认」，并给出建议的验证动作（如跑覆盖率、EXPLAIN、确认幂等 key）。

## Workflow

### 1. 确认范围

1. 确定仓库：用户指定路径 / 当前工作区最相关的 git 仓库。
2. 确定目标分支：用户指定分支；未指定则用当前分支。优先分析分支引用，不主动 `checkout`。
3. 确定基线：
   - 优先 `origin/main`，否则 `origin/master`，否则上游跟踪分支。
   - 用户可指定基线，如 `@dev-standards-diagnose --base origin/develop`。
4. 纳入未提交改动（staged + unstaged），报告中区分「已提交 / 未提交」仅在影响结论时简要标注。

### 2. 收集改动

并行执行：

```bash
git rev-parse --abbrev-ref HEAD
git merge-base <base> <branch>
git log --oneline <base>..<branch>
git diff --name-status <base>...<branch>
git diff --stat <base>...<branch>
git status --short
```

再按优先级阅读实质 diff（跳过无关生成噪音时可注明豁免）：

| 优先级 | 路径特征 |
|------|----------|
| 高 | `logic/`、`worker/`、`task/`、`dao/`、`bizlogic/`、`domain/`、`repository/`、handler（手写部分） |
| 中 | `client/`、`model/`、`enum/`、配置 `etc/`、`infra_changes/`、SQL/migration |
| 低/豁免倾向 | 纯生成文件（`types.go`、`routes.go`、`wire_gen.go`、Swagger）、仅测试夹具数据 |

### 3. 逐章诊断

对照 [reference.md](reference.md) 第 1–10 章，仅对**改动触及**的规则出结论。检查要点：

| 章 | 重点信号 |
|----|----------|
| 1 接口与输入 | 列表/批量无上限、未 trim、破坏兼容、导出一次加载、回调/MQ/任务缺幂等 |
| 2 结构与复杂度 | 魔法值、重复字符串>3、函数>50 行、圈复杂度偏高、状态/金额逻辑散落 |
| 3 类型命名注释 | 模糊命名、`any`/`map[string]any`、缺注释、下标当业务键、手改生成文件 |
| 4 测试 | 业务函数无单测、缺失败/边界、覆盖率可疑、mock 只断言 nil |
| 5 DB 事务 | 循环内查写、无分页扫表、批量>1000、缺稳定排序、事务/补偿不清 |
| 6 金额状态 | `float` 算钱、状态定义分散、内外字段隐式转换 |
| 7 文件导出 | 非流式 xlsx、全量进内存、临时文件未清理 |
| 8 日志安全 | 缺结构化日志、打印大响应/敏感信息、外部调用无超时/无限重试 |
| 9 PR 自检 | 是否具备写清兼容/分页/幂等/索引等说明的素材（报告可代拟 checklist） |
| 10 自动化 | 仓库存在则运行 `make qa-check`；记录通过/失败/未执行原因 |

### 4. 可选自动化

- 若根目录或模块有 `Makefile` 且含 `qa-check`：在改动相关模块执行 `make qa-check`，把结果写入报告。
- 失败则归入阻塞或告警；超时/环境缺失则标「未执行」并说明原因，不假装已通过。
- Go 改动可对触及包跑 `go test`（有则写结果）；不要为跑全仓无关包而长时间阻塞。

### 5. 汇总评级

| 评级 | 条件 |
|------|------|
| 通过 | 无 MUST 阻塞，SHOULD 可留建议 |
| 有条件通过 | 无 MUST，或 MUST 均有明确豁免+补偿；存在需跟进的 SHOULD |
| 不通过 | 存在未豁免的 MUST 违规或高危红线（N+1、全表扫、无幂等写入等） |

## Output Format

严格按以下模板输出，不要加无关前言。证据不足的条目写「待确认」。

```markdown
# 开发规范诊断报告

**仓库**：{path}
**分支**：{branch}  vs  **基线**：{base}
**改动概览**：{N} files，+{add}/-{del}（含未提交：是/否）
**规范版本**：wu-skills/dev-standards-diagnose/reference.md
**总评**：通过 | 有条件通过 | 不通过

## 摘要

- 阻塞（MUST）：{n}
- 建议（SHOULD）：{n}
- 豁免（EXEMPT）：{n}
- 未涉及章节：{列出章号}

## 改动文件

| 状态 | 路径 | 诊断相关性 |
|------|------|------------|
| M/A/D | path | 高/中/低/豁免 |

## 诊断明细

### 1. 接口限制与输入处理
- 结论：通过 / 阻塞 / 建议 / 未涉及 / 待确认
- 发现：
  - [{MUST|SHOULD}] {简述} — `{file}:{line}` — {证据} — {修复建议或豁免理由}

### 2. 代码结构与复杂度
（同上）

### 3. 类型、命名、注释与生成代码

### 4. 测试质量

### 5. DB 与事务

### 6. 金额、精度与状态

### 7. 文件与导出

### 8. 日志、外部调用与安全

### 9. PR 自检与豁免

### 10. 自动化与人工检查

- `make qa-check`：通过 / 失败 / 未执行（原因）
- 建议人工 checklist：{幂等 / 索引 / 日志脱敏 / 事务边界 … 按本次改动裁剪}

## 必须修复（合并前）

1. …

## 建议改进

1. …

## 豁免与补偿

| 项 | 原因 | 风险 | 补偿措施 | 是否后续任务 |
|----|------|------|----------|--------------|

## 建议写入 PR 的说明草稿

- 业务变更：
- 影响服务：
- 生成文件：
- 测试命令与结果：
- 兼容/分页/批量/幂等/索引（按涉及项填写）：
```

## Evidence Style

- 好：`settlementjob/internal/worker/foo.go:88` 在 `for` 内调用 `FindById`，疑似 N+1（MUST §5）
- 差：代码质量不好，建议重构

同一问题只报一次，归到最合适的章节；可用「另见 §x」交叉引用。

## Additional Resources

- 完整规范条文：[reference.md](reference.md)
