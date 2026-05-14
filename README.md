# wu-skills

个人 AI 技能库 | Personal Cursor Agent Skills（`hello-world`、`task-desc-generator`）

## Skills

| 技能名 | 描述 |
| --- | --- |
| hello-world | 生成最简单 hello world 示例。 |
| task-desc-generator | 根据 Git 分支改动生成可直接填入任务系统的中文任务描述。 |

## 安装

Cursor 会从本机技能目录加载子文件夹里的 `SKILL.md`。本仓库已放在个人技能路径下时，无需额外配置。

### 把整个库克隆到技能目录

把本仓库克隆到 `~/.cursor/skills/` 下任意文件夹名（例如 `wu-skills`），保证目录结构类似：

```text
~/.cursor/skills/wu-skills/
├── README.md
├── hello-world/
│   └── SKILL.md
└── task-desc-generator/
    └── SKILL.md
```

示例命令（若目录已存在请先备份或换名）：

```bash
mkdir -p ~/.cursor/skills
git clone https://github.com/wuxiangege/wu-skills.git ~/.cursor/skills/wu-skills
```

安装后重启 Cursor 或新开对话，便于技能列表刷新。

## 使用

### 在对话里 @ 技能（推荐）

在 Agent / Chat 输入框中输入 `@`，在列表里选择技能名称（与各 `SKILL.md`  frontmatter 里的 `name` 一致），再补充你的具体需求。

**示例：hello-world**

```text
@hello-world 给我一份 Go 的 hello world，要带文件名和运行命令
```

**示例：task-desc-generator**

```text
@task-desc-generator 根据当前分支相对 origin/main 的改动生成任务描述
```

```text
@task-desc-generator feature/TS-48761
```

## 维护说明

- 个人技能根路径：`~/.cursor/skills/`（勿与 Cursor 内置的 `~/.cursor/skills-cursor/` 混用）。
- 每个技能一个文件夹，文件夹内至少包含 `SKILL.md`；可选 `metadata.json`、`examples/`、`templates/` 等。
