# Media Hub

Media Hub 是一套面向个人媒体库管理、Emby 兼容访问和 115 网盘资源整理的综合工具。它把用户、卡密、工单、Emby 实例、MoviePilot、HDHive、115 网盘、播放风控、媒体库替换、资源转存、频道订阅和通知审计放到一个统一后台里，帮助个人用户减少重复维护，并让关键链路可追踪、可回滚、可验证。

本项目由 GPT-5.5 协助开发。

本目录只提供 Docker 部署文件和项目说明，不包含后端或前端源码。

## 使用声明

本项目仅限个人学习与技术研究使用。不得将本项目、Docker 镜像、部署包或基于本项目改造后的服务进行二次售卖、倒卖、分发销售，或作为收费服务对外提供。

使用中遇到问题可以加入 TG 群交流：

```text
https://t.me/+5FhoSbS2j48yMmI1
```

## Docker 镜像

```text
liushuangxidocker/media-hub:latest
```

统一镜像内置：

- API 服务
- Worker 服务
- 前端静态页面
- 数据库迁移文件
- Emby 兼容网关和 115 相关后台能力

## 功能概览

### 用户与会员

- 独立系统账号体系，使用服务端 Session Cookie 登录。
- RBAC 权限控制，角色、菜单、按钮动作、API 策略和后台导航统一由授权数据驱动。
- 用户列表、用户详情、会员状态、到期时间、角色、授权实例和操作记录。
- 用户中心支持个人资料、会员状态、续费、工单、115 账号绑定、车长模式和车队大厅。

### 卡密与续费

- 注册卡、续费卡、订单、退款、购买限制和流水管理。
- 支持 Stripe Checkout 和 webhook。
- 支持链动小铺作为外部商城入口：打开新窗口、复制链接、二维码购买。
- 注册卡展示邀请链接，支持卡密生命周期跟踪和删除。

### Emby 管理与替换模式

- 支持多 Emby 实例配置、健康检查、模板用户、用户导入和策略同步。
- 支持 Emby 替换兼容网关，提供纯替换验证能力。
- 支持兼容排查包、接口缺口统计、客户端日志、播放调试和替换 readiness。
- 兼容路径覆盖首页、Views、Items、PlaybackInfo、stream、图片、剧集季集等常用 Emby API。

### 115 网盘与媒体库

- 支持 115 扫码登录、账号池、单盘模式、大盘挂小盘模式。
- 支持 115 目录缓存：默认读本地缓存，手动刷新才访问 115。
- 支持按方案维护扫描目录，CID 来自缓存目录树。
- 支持扫描编排器、增量扫描、进度展示、扫描后通知和可选元数据增强。
- 支持 115 分享链接转存到指定网盘目录。
- 支持播放账号池、车长模式、车队大厅、乘客绑定关系和账号二选一规则。
- 支持 Telegram bot 订阅频道，识别 115 分享链接并进入可转存资源池。

### 播放链路与风控

- 支持 115 直链 302、播放缓存、直链探针、失败候选冷却和自动刷新。
- 支持播放记录、播放点击计数、同播计数、局域网同播限制、全局和用户级限制。
- 支持记录播放 IP，并解析省、市、县等地区信息。
- 支持 115 播放稳健内核：账号冷却、IP 窗口、singleflight 合并、negative cache、风控状态卡。
- 普通诊断和日志不展示真实 302、115 cookie、agent token、Emby token 或完整签名 query。

### 工单、求片与资源处理

- 用户可提交简化工单，问题类型和媒体支持搜索选择，媒体信息可自动回填。
- 用户端和管理端采用卡片式工单展示，详情通过弹窗查看。
- 求片选择媒体后展示媒体详情、总季数、总集数、库内已有季集和可用图片。
- 工单可推送到 MoviePilot，将关联媒体创建为 MP 订阅任务。
- 工单可通过 HDHive 查询和解锁资源，并记录解锁后的资源链接。
- 已对接 115 分享转存时，管理员可选择转存目录并从工单处理资源。

### 通知、审计与运维

- 支持邮件、Telegram、企业微信、飞书等通知渠道。
- 支持扫描资源后通知、订单通知、退款通知、工单通知、MP 推送通知和失败重试。
- 后台关键操作写入审计日志。
- 运营中心汇总系统待办、依赖健康、异常任务和关键模块状态。

## 快速部署

### 1. 准备目录

```bash
mkdir -p media-hub
cd media-hub
```

把本仓库里的文件放到该目录：

```text
docker-compose.yml
.env.example
README.md
```

### 2. 创建配置

```bash
cp .env.example .env
```

至少修改这些值：

```env
PUBLIC_BASE_URL=http://YOUR_SERVER_IP:8080
POSTGRES_PASSWORD=CHANGE_ME_DB_PASSWORD
DATABASE_URL=postgres://media_hub:CHANGE_ME_DB_PASSWORD@postgres:5432/media_hub?sslmode=disable
BOOTSTRAP_ADMIN_PASSWORD=CHANGE_ME_ADMIN_PASSWORD
BOOTSTRAP_ADMIN_EMAIL=admin@example.com
SETTINGS_ENCRYPTION_KEY=CHANGE_ME_64_HEX_OR_LONG_RANDOM
CARD_PLAINTEXT_ENCRYPTION_KEY=CHANGE_ME_64_HEX_OR_LONG_RANDOM
```

生成随机密钥示例：

```bash
openssl rand -hex 32
```

### 3. 启动

```bash
docker compose pull
docker compose up -d
docker compose logs -f api
```

### 4. 验证

```bash
curl -fsS http://YOUR_SERVER_IP:8080/api/health
```

正常响应类似：

```json
{
  "status": "ok",
  "service": "media-hub-api",
  "environment": "production",
  "database": "configured",
  "started_at": "2026-05-26T00:00:00Z"
}
```

浏览器访问：

```text
http://YOUR_SERVER_IP:8080
```

首次登录使用 `.env` 中的：

- `BOOTSTRAP_ADMIN_USERNAME`
- `BOOTSTRAP_ADMIN_PASSWORD`

## 数据目录

默认数据保存在当前目录下：

```text
data/postgres
data/redis
data/config
data/media-images
```

备份时至少备份：

- PostgreSQL 数据目录或数据库 dump
- `data/config`
- `data/media-images`
- `.env`

`data/config` 中会保存兼容媒体库生成产物、播放映射、图片索引等运行时文件。115 账号密钥、运行时配置、扫描数据、工单、用户、订单等以数据库为主。

## 常用命令

```bash
docker compose ps
docker compose logs -f api
docker compose logs -f worker
docker compose pull
docker compose up -d
docker compose down
```

升级镜像：

```bash
docker compose pull
docker compose up -d
```

## 部署后建议配置

1. 登录后台，检查系统设置和公网入口。
2. 配置通知渠道，例如 SMTP、Telegram、企业微信或飞书。
3. 配置 Emby 实例和替换模式。
4. 配置 115 账号池、客户模式和扫描方案。
5. 刷新 115 目录缓存，从缓存树选择扫描目录。
6. 执行扫描并生成 Emby 兼容媒体库。
7. 用 VidHub、网易爆米花等客户端验证首页、详情、图片、播放和续播。
8. 根据实际需要配置卡密、续费、工单、MoviePilot、HDHive、TG 频道订阅和风控策略。
