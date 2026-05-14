# docker-compose 技能

在 Cursor 对话里通过 `@docker-compose` 引用本技能，并附上配方名，即可让助手输出**可直接使用**的 `docker-compose.yml` 内容。

## 用法

```text
@docker-compose mysql5
```

将助手给出的 YAML 保存为项目根目录的 `docker-compose.yml`，然后执行：

```bash
docker compose up -d
```

## 当前支持的配方

| 配方 | 说明 |
| --- | --- |
| `mysql5` | MySQL 5.7，端口 `3306`，utf8mb4，带健康检查与数据卷 |
| `mysql8` | MySQL 8.0，端口 `3306`，utf8mb4，带健康检查与数据卷 |
| `postgresql` | PostgreSQL 16（Alpine），端口 `5432`，别名 `postgres`、`pg` |
| `rabbitmq` | RabbitMQ 3 + 管理插件，`5672` / `15672` |
| `rocketmq` | Apache RocketMQ 5.2.0，NameServer `9876`，Broker 端口 `10909`/`10911`/`10912` |
| `xxl-job` | XXL-JOB Admin 2.4.1 + MySQL 8，Admin `8080`；需同目录放置建表 SQL |

## 默认账号（仅本地开发）

### MySQL（mysql5 / mysql8）

| 变量 | 默认值 |
| --- | --- |
| root 密码 | `root` |
| 数据库名 | `app` |
| 应用用户 / 密码 | `app` / `app` |

### PostgreSQL

| 变量 | 默认值 |
| --- | --- |
| 用户 / 密码 / 库名 | `app` / `app` / `app` |

### RabbitMQ

| 变量 | 默认值 |
| --- | --- |
| 用户 / 密码 | `app` / `app` |

### XXL-JOB

| 项 | 默认值 |
| --- | --- |
| MySQL root 密码 | `root` |
| 库名 | `xxl_job`（由 SQL 与 `MYSQL_DATABASE` 一致） |
| Admin 地址 | `http://127.0.0.1:8080/xxl-job-admin` |
| Admin 登录 | 一般为 `admin` / `123456`（以镜像说明为准） |

**请勿**在生产环境使用上述默认密码；请改为 `.env` 或密钥管理方案。

## 仓库内模板

与本技能一致的参考文件（可复制为 `docker-compose.yml`，或与 `SKILL.md` 中片段一致）：

- `templates/mysql5.yml`
- `templates/mysql8.yml`
- `templates/postgresql.yml`
- `templates/rabbitmq.yml`
- `templates/rocketmq.yml`
- `templates/xxl-job.yml`（需与同目录下的 `xxl-job-tables.sql` 一并使用）
- `templates/xxl-job-tables.sql`（官方 v2.4.1 建表脚本）

使用 `xxl-job` 模板时，请在**存放 `xxl-job.yml` 的目录**下同时保留 `xxl-job-tables.sql`，或修改 compose 中的卷挂载路径。

## 依赖

- 已安装 [Docker](https://docs.docker.com/get-docker/) 与 [Docker Compose V2](https://docs.docker.com/compose/)（`docker compose` 子命令）。
- `rocketmq` 配方使用 Compose `configs` 内嵌 Broker 配置，需较新的 Docker Compose（支持 spec 中的 `configs.content`）。
