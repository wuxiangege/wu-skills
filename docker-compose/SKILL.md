---
name: docker-compose
description: >-
  根据用户指定的配方输出可直接保存为 docker-compose.yml 的标准 Compose 片段。
  支持：mysql5、mysql8、postgresql、rabbitmq、rocketmq、xxl-job。
  适用于用户 @docker-compose、提到 docker-compose.yml、Docker Compose、本地数据库/中间件编排时。
---

# Skill: Docker Compose 片段

## Role

当用户通过 `@docker-compose` 并附带配方名请求编排文件时，输出**完整、可运行**的 `docker-compose.yml` 内容，不输出长篇理论。

---

## Goal

1. 默认输出**可直接保存为项目根目录 `docker-compose.yml`** 的 YAML。
2. 使用 **Compose 规范**写法（不写已废弃的顶层 `version` 字段），兼容 Docker Compose V2。
3. 配方名**不区分大小写**；别名：`postgresql` 等同于 `postgres`、`pg`；`xxl-job` 等同于 `xxljob`。
4. 在 YAML 后附上**最短**的启动与连接示例命令（`docker compose up -d`、可选 `docker compose ps`）。

---

## Rules

1. **mysql5**：官方 `mysql:5.7`，`3306`，utf8mb4，`mysql_native_password`，健康检查与命名卷。
2. **mysql8**：官方 `mysql:8.0`，`3306`，utf8mb4，健康检查与命名卷（默认认证插件为 MySQL 8 原生方式）。
3. **postgresql**：官方 `postgres:16-alpine`，`5432`，`POSTGRES_*` 环境变量，健康检查与命名卷。
4. **rabbitmq**：官方 `rabbitmq:3-management-alpine`，`5672`（AMQP）、`15672`（管理 UI），健康检查与命名卷。
5. **rocketmq**：`apache/rocketmq:5.2.0`，NameServer `9876`，Broker `10909`/`10911`/`10912`；使用 Compose `configs` 内嵌 `broker.conf`（`brokerIP1=127.0.0.1` 便于宿主机客户端经端口映射访问）。
6. **xxl-job**：`mysql:8.0`（仅集群内访问、不映射宿主机端口）+ `xuxueli/xxl-job-admin:2.4.1`；初始化挂载官方 `xxl_job` 建表 SQL；Admin `8080`。
7. 环境变量给出**本地开发可用**的默认值；在回复中**用一两句话提醒**生产环境务必改为强密码或 `.env`。
8. 代码块使用 `yaml` 语言标签；若用户未指定文件名，说明保存为 `docker-compose.yml`；若指定路径，按用户路径说明。

---

## 配方：mysql5（MySQL 5.7）

用户说 `@docker-compose mysql5` 或等价表述时，**原样输出**以下 compose（仅允许根据用户要求改 `container_name`、端口映射或数据库名；否则保持默认）：

```yaml
services:
  mysql:
    image: mysql:5.7
    container_name: mysql5
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app
      MYSQL_USER: app
      MYSQL_PASSWORD: app
    ports:
      - "3306:3306"
    volumes:
      - mysql5_data:/var/lib/mysql
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default-authentication-plugin=mysql_native_password
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

volumes:
  mysql5_data:
```

**启动：**

```bash
docker compose up -d
```

**本机连接示例：**

```bash
mysql -h 127.0.0.1 -P 3306 -uapp -papp app
```

（root：`mysql -h 127.0.0.1 -P 3306 -uroot -proot`。）

---

## 配方：mysql8（MySQL 8.0）

用户说 `@docker-compose mysql8` 时，**原样输出**：

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql8
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app
      MYSQL_USER: app
      MYSQL_PASSWORD: app
    ports:
      - "3306:3306"
    volumes:
      - mysql8_data:/var/lib/mysql
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

volumes:
  mysql8_data:
```

**启动：**

```bash
docker compose up -d
```

**本机连接示例：**

```bash
mysql -h 127.0.0.1 -P 3306 -uapp -papp app
```

---

## 配方：postgresql（PostgreSQL 16）

用户说 `@docker-compose postgresql`、`postgres` 或 `pg` 时，**原样输出**：

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres16
    restart: unless-stopped
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 20s

volumes:
  postgres_data:
```

**启动：**

```bash
docker compose up -d
```

**本机连接示例：**

```bash
psql "postgresql://app:app@127.0.0.1:5432/app"
```

---

## 配方：rabbitmq（RabbitMQ 3 + 管理插件）

用户说 `@docker-compose rabbitmq` 时，**原样输出**：

```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: rabbitmq3
    restart: unless-stopped
    environment:
      RABBITMQ_DEFAULT_USER: app
      RABBITMQ_DEFAULT_PASS: app
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 40s

volumes:
  rabbitmq_data:
```

**启动：**

```bash
docker compose up -d
```

**连接与管理界面：**

- AMQP：`amqp://app:app@127.0.0.1:5672/`
- 管理 UI：`http://127.0.0.1:15672/`（账号 `app` / 密码 `app`）

---

## 配方：rocketmq（RocketMQ 5.2）

用户说 `@docker-compose rocketmq` 时，**原样输出**：

```yaml
services:
  rmqnamesrv:
    image: apache/rocketmq:5.2.0
    container_name: rmqnamesrv
    restart: unless-stopped
    ports:
      - "9876:9876"
    volumes:
      - rmq_namesrv_logs:/home/rocketmq/logs
    command: sh mqnamesrv

  rmqbroker:
    image: apache/rocketmq:5.2.0
    container_name: rmqbroker
    restart: unless-stopped
    ports:
      - "10909:10909"
      - "10911:10911"
      - "10912:10912"
    volumes:
      - rmq_broker_logs:/home/rocketmq/logs
      - rmq_broker_store:/home/rocketmq/store
    depends_on:
      rmqnamesrv:
        condition: service_started
    command: sh mqbroker -n rmqnamesrv:9876 -c /run/rocketmq/broker.conf
    configs:
      - source: rmq_broker_conf
        target: /run/rocketmq/broker.conf

configs:
  rmq_broker_conf:
    content: |
      brokerClusterName=DefaultCluster
      brokerName=broker-a
      brokerId=0
      deleteWhen=04
      fileReservedTime=48
      brokerRole=ASYNC_MASTER
      flushDiskType=ASYNC_FLUSH
      brokerIP1=127.0.0.1
      namesrvAddr=rmqnamesrv:9876
      listenPort=10911
      autoCreateTopicEnable=true

volumes:
  rmq_namesrv_logs:
  rmq_broker_logs:
  rmq_broker_store:
```

**启动：**

```bash
docker compose up -d
```

**客户端地址：** NameServer `127.0.0.1:9876`；Broker 经映射端口访问（如 `10911`）。若仅在 **Docker 网络内**其他容器访问，需将 `brokerIP1` 改为对客户端可达的地址并同步调整配置。

---

## 配方：xxl-job（调度中心 + MySQL）

用户说 `@docker-compose xxl-job` 或 `xxljob` 时，输出以下 compose，并**单独说明**：须将官方建表 SQL 与本文件放在**同一目录**（或按用户要求修改卷路径）。仓库内与模板一致的文件名为 `xxl-job-tables.sql`（XXL-JOB v2.4.1）。

```yaml
services:
  xxl-mysql:
    image: mysql:8.0
    container_name: xxl-job-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: xxl_job
    volumes:
      - xxl_job_mysql_data:/var/lib/mysql
      - ./xxl-job-tables.sql:/docker-entrypoint-initdb.d/00-xxl-job.sql:ro
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 15
      start_period: 60s

  xxl-job-admin:
    image: xuxueli/xxl-job-admin:2.4.1
    container_name: xxl-job-admin
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      PARAMS: >-
        --spring.datasource.url=jdbc:mysql://xxl-mysql:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true
        --spring.datasource.username=root
        --spring.datasource.password=root
    depends_on:
      xxl-mysql:
        condition: service_healthy

volumes:
  xxl_job_mysql_data:
```

**启动：**

```bash
docker compose up -d
```

**访问：** `http://127.0.0.1:8080/xxl-job-admin`（默认登录 `admin` / `123456`，以镜像内默认配置为准）。

**说明：** 首次启动需等待 MySQL 执行完 `docker-entrypoint-initdb.d` 脚本后再访问 Admin；若数据卷已存在但库未初始化，需清空对应 volume 或手动导入 SQL。

---

## 未支持的配方

若用户请求的配方不在 **mysql5、mysql8、postgresql、rabbitmq、rocketmq、xxl-job** 中，明确列出当前支持的配方名，并询问是否需要沿用其一或补充目标镜像与端口。
