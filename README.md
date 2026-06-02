# 🚀 wu-skills

个人 AI 技能库 | Personal Cursor Agent Skills  
包含：`hello-world`、`task-desc-generator`、`pua-response`、`business-flow-mapper`

---

## ✨ Skills

| 技能名 | 描述 |
| --- | --- |
| 🧪 hello-world | 生成最简单 hello world 示例。 |
| 📝 task-desc-generator | 根据 Git 分支改动生成可直接填入任务系统的中文任务描述。 |
| 🛡️ pua-response | 拆解情感操控、职场打压等经历，并给出被动反制与生存方案。 |
| 🗺️ business-flow-mapper | 梳理零散业务信息，输出流程图、时序图、状态图等 Mermaid 图表，快速理清链路。 |

---

## 📦 安装

Cursor 会从本机技能目录加载子文件夹里的 `SKILL.md`。  
本仓库已放在个人技能路径下时，无需额外配置。

### 📁 把整个库克隆到技能目录

把本仓库克隆到 `~/.cursor/skills/` 下任意文件夹名（例如 `wu-skills`），保证目录结构类似：

```text
~/.cursor/skills/wu-skills/
├── README.md
├── hello-world/
│   └── SKILL.md
├── task-desc-generator/
│   └── SKILL.md
├── pua-response/
│   └── SKILL.md
└── business-flow-mapper/
    └── SKILL.md
```

示例命令（若目录已存在请先备份或换名）：

```bash
mkdir -p ~/.cursor/skills
git clone https://github.com/wuxiangege/wu-skills.git ~/.cursor/skills/wu-skills
```

安装后重启 Cursor 或新开对话，便于技能列表刷新。

---

## 🛠 使用

### 💬 在对话里 @ 技能（推荐）

在 Agent / Chat 输入框中输入 `@`，  
在列表里选择技能名称（与各 `SKILL.md` frontmatter 里的 `name` 一致），再补充你的具体需求。

---

### 🧪 示例：hello-world

```text
@hello-world 给我一份 Go 的 hello world，要带文件名和运行命令
```

---

### 📝 示例：task-desc-generator

```text
@task-desc-generator 根据当前分支相对 origin/main 的改动生成任务描述
```

```text
@task-desc-generator feature/TS-48761
```

---

### 🛡️ 示例：pua-response

```text
@pua-response 我最近的经历是……
```

```text
@pua-response 帮我拆解这段经历，并给我被动反制和生存方案
```

---

### 🗺️ 示例：business-flow-mapper

```text
@business-flow-mapper 用户下单后调用支付，支付成功才扣库存，失败就取消订单。还有支付回调。
```

```text
@business-flow-mapper 根据这段 PRD 帮我画流程图和时序图：[粘贴需求片段]
```

```text
@business-flow-mapper 结合 order/handler 和 payment 相关代码，梳理下单支付链路
```

---

## 🧱 维护说明

- 📂 个人技能根路径：`~/.cursor/skills/`
- ⚠️ 勿与 Cursor 内置目录 `~/.cursor/skills-cursor/` 混用
- 📦 每个技能一个文件夹
- 📄 文件夹内至少包含 `SKILL.md`
- 🧩 可选：
  - `metadata.json`
  - `examples/`
  - `templates/`

---

## 📌 未来计划

- [ ] 增加更多工程化技能
- [ ] 支持多语言代码生成
- [ ] 增加 Git 工作流辅助技能
- [ ] 增加文档自动化生成技能

---

## 📄 License

MIT
