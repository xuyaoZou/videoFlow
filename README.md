# 🎬 Luciano

> **ComfyUI for Video** — 节点化 AI 视频创作引擎

Luciano 是一个基于节点画布的 AI 视频创作平台，支持将多个 AI 模型（Kling、Seedance 等）的能力组合成可视化工作流。

![Flow Canvas Preview](docs/images/flow-canvas-preview.jpg)

## ✨ 特性

- **节点画布** — ComfyUI 式的可视化工作流编辑器，拖拽节点、连线、组合能力
- **多模型适配器** — 统一接口对接 Kling AI、火山方舟 Seedance 等模型，一个适配器接入，全局可用
- **DAG 执行引擎** — 拓扑排序 + 逐层并行执行 + 端口数据流自动传递
- **双模式创作** — Agent 对话式创作（小白友好）+ 专业节点画布（精准控制）
- **实时反馈** — SSE 推送执行进度，节点完成即时显示结果
- **媒体资产管理** — 本地/S3 双存储后端，自动下载落地，前端 Blob URL 缓存

## 🏗️ 架构概览

```
┌──────────────────────────────────────────────────┐
│                    前端 (Nuxt 3)                    │
│   Vue Flow 画布 │ Tailwind CSS │ SSE 实时通信       │
└────────────────────┬─────────────────────────────┘
                     │ REST + SSE
┌────────────────────┴─────────────────────────────┐
│                  后端 (Spring Boot)                │
│  ┌──────────┐  ┌──────────┐  ┌────────────────┐   │
│  │Flow 引擎  │  │适配器层   │  │媒体系统         │   │
│  │DAG 拓扑   │  │Registry  │  │本地+S3 双存储   │   │
│  │端口数据流  │  │9+ 能力   │  │代理下载         │   │
│  └──────────┘  └──────────┘  └────────────────┘   │
└────────────────────┬─────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Kling AI    火山方舟       PostgreSQL
              Seedance        + Flyway
```

## 🚀 快速开始

### 前置要求

- Docker & Docker Compose（推荐）
- 或：JDK 17+、Node.js 20+、PostgreSQL 16+、pnpm

### 方式一：Docker Compose 一键启动

```bash
# 1. 克隆仓库
git clone https://github.com/xuyaoZou/videoflow.git
cd luciano

# 2. 配置环境变量
cp .env.example .env
# 编辑 .env，填入你的 API Key（至少需要一个模型）

# 3. 一键启动
docker compose up -d

# 4. 访问
# 前端: http://localhost:3000
# 后端 API: http://localhost:8090
```

### 方式二：本地开发模式

```bash
# 1. 启动 PostgreSQL
docker run -d --name luciano-pg \
  -e POSTGRES_DB=luciano \
  -e POSTGRES_USER=luciano \
  -e POSTGRES_PASSWORD=luciano_dev \
  -p 5432:5432 \
  postgres:16-alpine

# 2. 启动后端
cd luciano_backend
cp .env.adapters.example .env.adapters
# 编辑 .env.adapters，填入 API Key
mvn spring-boot:run

# 3. 启动前端
cd ../luciano-web
pnpm install
pnpm dev
```

### API Key 申请

| 模型 | 申请地址 | 文档 |
|------|---------|------|
| Kling AI | https://klingai.kuaishou.com/ | 国内版用 api-beijing.klingai.com |
| 火山方舟 Seedance | https://www.volcengine.com/product/ark | Bearer Token 认证 |

## 🧩 核心概念

### 节点画布

每个能力（文生视频、图生视频、文生图等）是一个节点，节点之间通过**端口**连线，数据自动流动。

```
[文本输入] ──prompt──→ [文生图] ──image──→ [图生视频] ──video──→ [预览]
```

### 适配器层

实现 `ModelAdapter` 接口，`@Component` 注解自动注册。新增模型只需：

```java
@Component
public class YourAdapter implements ModelAdapter {
    // 实现 submit/poll/download 等方法
}
```

### 端口类型系统

15 种 PortType，支持类型兼容检查和自动转换（如 IMAGE → REFERENCE）。

## 📊 支持的能力

| 能力 | Kling | Seedance |
|------|:-----:|:--------:|
| 文生视频 (T2V) | ✅ | ✅ |
| 图生视频 (I2V) | ✅ | ✅ |
| 视频续写 (V2V) | ✅ | ✅ |
| 运镜控制 (Camera) | ✅ | ✅ |
| 文生图 (T2I) | ✅ | ✅ |
| 图片编辑 (I2I) | ✅ | — |
| 背景移除 | ✅ | — |
| Omni 模型 | ✅ | — |

## 📁 项目结构

```
luciano/
├── luciano_backend/          # Spring Boot 后端
│   ├── src/main/java/com/luciano/
│   │   ├── adapter/          # 多模型适配器层
│   │   │   ├── kling/        # Kling AI 适配器
│   │   │   └── seedance/     # 火山方舟适配器
│   │   ├── flow/             # Flow 引擎 (DAG + 端口系统)
│   │   ├── controller/       # REST API
│   │   ├── service/         # 业务逻辑
│   │   └── config/          # 配置 (Security, JWT, S3)
│   └── src/main/resources/
│       ├── db/migration/    # Flyway 迁移脚本 (V1-V17)
│       └── application.yml  # 配置 (环境变量驱动)
│
├── luciano-web/              # Nuxt 3 前端
│   ├── components/
│   │   └── flow/             # Vue Flow 画布组件
│   ├── composables/          # useAuth, useMediaLoader, useTheme 等
│   └── pages/
│
└── docker-compose.yml        # 一键启动
```

## ⚠️ 当前状态

**Work in Progress** — 核心框架已完成，正在持续迭代中。

### 已完成
- ✅ 适配器层架构 + Kling/Seedance 适配器
- ✅ Flow 引擎 (DAG 拓扑 + 端口系统 + 数据流通)
- ✅ Vue Flow 节点画布 (暗色主题 + 右键菜单 + 属性面板)
- ✅ 工作流持久化 (保存/加载/执行)
- ✅ SSE 实时执行反馈
- ✅ 媒体系统 (本地/S3 存储 + 代理下载)
- ✅ JWT 认证体系
- ✅ Agent 对话模式

### 进行中
- 🔄 E2E 数据流验证
- 🔄 工作流模板体系
- 🔄 执行结果持久化与恢复

### 规划中
- 🔲 更多模型适配器 (Runway, Pika 等)
- 🔲 TTS / 音频生成
- 🔲 多角色一致性生成
- 🔲 模板市场

## 🛠️ 技术栈

**后端**: Java 17 · Spring Boot 3.3 · MyBatis-Plus · PostgreSQL · Flyway
**前端**: Nuxt 3 · Vue 3.5 · Vue Flow · Tailwind CSS
**存储**: 本地文件系统 · S3 兼容 (TOS/MinIO/AWS)
**AI 模型**: Kling AI · 火山方舟 Seedance

## 📝 License

MIT License — 随意使用，欢迎 PR。

## 🙋 关于

一个人做的 AI 视频工作流引擎。如果你也在做 AI 视频相关的东西，欢迎交流。

Issue / PR / Star 都欢迎。