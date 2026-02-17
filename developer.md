# Feline Developer Guide

本文档面向 Feline 项目的开发者，提供架构设计、开发指南、API 规范和未来规划。

## 目录

- [架构概述](#架构概述)
- [开发环境设置](#开发环境设置)
- [代码结构](#代码结构)
- [核心模块详解](#核心模块详解)
- [数据流与通信](#数据流与通信)
- [API 规范](#api-规范)
- [当前问题与改进计划](#当前问题与改进计划)
- [未来规划](#未来规划)

---

## 架构概述

### 整体架构

Feline 采用客户端-服务端架构，支持本地模式和云端模式两种部署方式：

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              User's Machine                                  │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        VS Code Extension (feline-plugin)               │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐    │  │
│  │  │   feline    │  │    iris     │  │         whisker             │    │  │
│  │  │ (Extension  │  │  (Chat UI)  │  │    (Local Backend)          │    │  │
│  │  │   Host)     │  │   React     │  │  Python + FastAPI           │    │  │
│  │  └──────┬──────┘  └──────┬──────┘  └──────────────┬──────────────┘    │  │
│  └─────────┼────────────────┼────────────────────────┼───────────────────┘  │
│            │                │                        │                       │
│            │   postMessage  │    HTTP/SSE            │                       │
│            └────────────────┴────────────────────────┘                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ HTTP/SSE (云端模式)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Cloud Infrastructure                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    feline-server (API Gateway)                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  PostgreSQL  │  │    Redis     │  │ feline-monitor│  │ LLM Providers│    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 运行模式

| 模式 | 数据流 | 适用场景 |
|------|--------|----------|
| **本地模式** | Extension → Whisker → LLM Provider | 开发者自带 API Key，数据存储在本地 |
| **云端模式** | Extension → feline-server → LLM Provider | 企业用户，统一管理和计费 |

---

## 开发环境设置

### 环境要求

| 工具 | 版本 | 用途 |
|------|------|------|
| Node.js | >= 18 | 前端构建、扩展开发 |
| Python | >= 3.10 | 后端服务 |
| pnpm/npm | 最新 | 包管理 |
| Docker | 最新 | 容器化部署 |

### 安装依赖

```bash
# 克隆仓库
git clone https://github.com/phanhom/feline.git
cd feline

# 安装所有依赖
npm run install:all

# 或单独安装
cd feline-plugin && npm install
cd feline-server && pip install -r requirements.txt
```

### 开发命令

```bash
# feline-plugin
npm run dev:feline   # 扩展开发（需要 F5 启动调试）
npm run dev:iris     # UI 开发（热重载）
npm run dev:whisker  # 后端开发

# 构建
npm run build        # 构建所有包
npm run build:iris   # 仅构建 UI
npm run build:feline # 仅构建扩展
npm run package:vsix # 打包 .vsix 文件

# feline-server
cd feline-server
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

### IDE 配置

推荐使用 VS Code，并安装以下扩展：

- Python (Microsoft)
- Pylance
- ESLint
- TypeScript Vue Plugin (Volar)
- Tailwind CSS IntelliSense

---

## 代码结构

### feline-plugin

```
feline-plugin/
├── packages/
│   ├── feline/                    # VS Code 扩展宿主
│   │   ├── src/
│   │   │   └── extension.ts       # 扩展入口，Webview 管理，消息处理
│   │   ├── assets/                # 静态资源
│   │   ├── package.json           # 扩展配置（命令、快捷键、菜单）
│   │   └── webpack.config.js      # 构建配置
│   │
│   ├── iris/                      # 聊天 UI (Webview)
│   │   ├── src/
│   │   │   ├── App.tsx            # 主组件
│   │   │   ├── components/        # UI 组件
│   │   │   │   ├── ChatList.tsx   # 消息列表
│   │   │   │   ├── ChatInput.tsx  # 输入框
│   │   │   │   ├── ActionPanel.tsx# 工具结果面板
│   │   │   │   └── Settings/      # 设置页面
│   │   │   ├── hooks/
│   │   │   │   └── useVSCode.ts   # VS Code 通信 Hook
│   │   │   └── types/
│   │   │       └── index.ts       # TypeScript 类型定义
│   │   ├── vite.config.ts
│   │   └── tailwind.config.js
│   │
│   └── whisker/                   # 本地后端
│       ├── app/
│       │   ├── api/
│       │   │   └── endpoints.py   # API 路由
│       │   ├── core/
│       │   │   ├── config.py      # 配置管理
│       │   │   ├── system_prompt.py# 系统提示词
│       │   │   └── workspace.py   # 工作区管理
│       │   ├── schemas/
│       │   │   └── chat.py        # 数据模型
│       │   └── services/
│       │       ├── llm/           # LLM 服务
│       │       │   ├── providers/ # 各 Provider 实现
│       │       │   └── factory.py # Provider 工厂
│       │       └── tools/         # 工具定义与执行
│       └── main.py                # 入口文件
│
├── scripts/
│   └── package.sh                 # 打包脚本
└── feline.config.json             # 全局配置
```

### feline-server

```
feline-server/
├── src/
│   ├── api/                       # API 路由
│   │   ├── auth.py                # 认证（注册/登录）
│   │   ├── inference.py           # LLM 推理接口
│   │   ├── models.py              # 模型列表
│   │   ├── providers.py           # Provider 列表
│   │   ├── subscriptions.py       # 订阅管理
│   │   ├── usage.py               # 用量统计
│   │   └── users.py               # 用户管理
│   │
│   ├── core/                      # 核心模块
│   │   ├── config.py              # 配置管理
│   │   ├── database.py            # 数据库连接
│   │   ├── security.py            # 安全工具（JWT、密码哈希）
│   │   └── middleware.py          # 中间件
│   │
│   ├── llm/                       # LLM 客户端
│   │   ├── base.py                # 基类
│   │   ├── anthropic.py           # Claude 实现
│   │   ├── openai.py              # OpenAI 实现
│   │   └── client.py              # 统一客户端
│   │
│   ├── models/                    # 数据库模型
│   │   ├── user.py
│   │   ├── subscription.py
│   │   └── usage.py
│   │
│   ├── services/                  # 业务逻辑
│   │   ├── user.py
│   │   ├── subscription.py
│   │   └── usage.py
│   │
│   └── main.py                    # 应用入口
│
├── utils/                         # 工具函数
│   ├── metrics.py                 # Prometheus 指标
│   ├── redis.py                   # Redis 客户端
│   └── monitoring.py              # 监控工具
│
├── config.toml                    # 配置文件
└── requirements.txt
```

---

## 核心模块详解

### feline (Extension Host)

**职责：** VS Code 扩展宿主，管理 Webview 生命周期、处理用户交互、与 Whisker/feline-server 通信。

**关键代码：** [extension.ts](feline-plugin/packages/feline/src/extension.ts)

**核心功能：**

1. **Webview 管理**
   - `FelineChatViewProvider`：管理侧边栏 Webview
   - 加载 Iris UI（React 应用）
   - 处理主题切换

2. **消息通信**
   - `postMessage`：Extension ↔ Webview 双向通信
   - 支持的消息类型：`getSettings`、`saveSettings`、`sendMessage`、`stopGeneration` 等

3. **Whisker 进程管理**
   - 自动启动/重启 Whisker 后端
   - 端口冲突检测与自动切换
   - 健康检查机制

4. **会话持久化**
   - 使用 `globalState` 存储会话历史
   - 支持会话切换、重命名、回溯

### iris (Chat UI)

**职责：** 聊天 UI，运行在 Webview 中，负责消息渲染、用户交互、工具展示。

**关键代码：** [App.tsx](feline-plugin/packages/iris/src/App.tsx)、[useVSCode.ts](feline-plugin/packages/iris/src/hooks/useVSCode.ts)

**核心组件：**

| 组件 | 职责 |
|------|------|
| `ChatList` | 消息列表渲染，支持流式更新、回溯按钮 |
| `ChatInput` | 输入框，支持模式切换（agent/chat）、快捷键 |
| `ActionPanel` | 工具结果面板，显示文件变更、任务列表 |
| `AskUserCard` | 用户确认卡片，用于 Plan Mode 和工具确认 |
| `SettingsPage` | 设置页面，配置 Provider、API Key 等 |

### whisker (Local Backend)

**职责：** 本地后端服务，提供 LLM 接口、工具执行、上下文压缩。

**关键代码：** [endpoints.py](feline-plugin/packages/whisker/app/api/endpoints.py)

**核心 API：**

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/health` | GET | 健康检查 |
| `/api/chat` | POST | 聊天接口（SSE 流式） |

**工具系统：**

```python
# 工具定义位置
feline-plugin/packages/whisker/app/services/tools/
├── definitions.py   # 工具定义（read_file, write_file, run_bash, etc.）
├── executor.py      # 工具执行器
└── base.py          # 工具基类
```

### feline-server (Cloud API)

**职责：** 云服务后端，提供统一 LLM 接口、用户认证、订阅管理、用量统计。

**关键代码：** [inference.py](feline-server/src/api/inference.py)

**核心 API：**

| 端点 | 方法 | 描述 |
|------|------|------|
| `/v1/auth/sign_up` | POST | 用户注册 |
| `/v1/auth/sign_in` | POST | 用户登录 |
| `/v1/chat/completions` | POST | Chat 推理 |
| `/v1/completions` | POST | 文本补全 |
| `/v1/models` | GET | 模型列表 |
| `/v1/subscriptions` | GET | 订阅信息 |
| `/v1/usage/summary` | GET | 用量统计 |

---

## 数据流与通信

### Extension ↔ Webview 通信

```
┌─────────────┐                      ┌─────────────┐
│  Extension  │                      │    Iris     │
│  (feline)   │                      │  (Webview)  │
└──────┬──────┘                      └──────┬──────┘
       │                                    │
       │  webview.postMessage({             │
       │    command: 'loadSettings',        │
       │    settings: {...}                 │
       │  })                                │
       │ ─────────────────────────────────► │
       │                                    │
       │                                    │  window.addEventListener('message')
       │                                    │
       │  window.addEventListener('message')│
       │ ◄───────────────────────────────── │
       │                                    │
       │  {                                 │
       │    command: 'sendMessage',         │
       │    messages: [...],                │
       │    text: '...'                     │
       │  }                                 │
       │                                    │
```

### SSE 流式响应

```
┌─────────────┐    POST /api/chat     ┌─────────────┐
│  Extension  │ ────────────────────► │   Whisker   │
│             │                       │   (Local)   │
│             │ ◄──────────────────── │             │
│             │   SSE Stream          │             │
│             │                       │             │
│             │   data: {"content":..│             │
│             │   data: {"thinking":.│             │
│             │   data: {"tool_call":│             │
│             │   data: {"tool_result│             │
│             │   data: [DONE]       │             │
└─────────────┘                       └─────────────┘
```

### SSE 事件类型

| 事件 | 字段 | 描述 |
|------|------|------|
| 内容流 | `content` | 文本内容增量 |
| 思考过程 | `thinking` | 模型思考过程（DeepSeek-R1 等） |
| 工具调用 | `tool_call` | 工具调用请求 |
| 工具结果 | `tool_result` | 工具执行结果 |
| 系统事件 | `system_event` | 系统级事件（压缩、重试等） |
| 完成 | `done: true` | 响应结束 |

---

## API 规范

### Chat 请求格式

```typescript
interface ChatRequest {
  messages: Message[];
  settings: {
    base_url: string;
    api_key: string;
    model_name: string;
    max_context_tokens?: number;
    tool_timeout_seconds?: number;
  };
  workspace_root: string;
  plan_mode: boolean;
  mode: 'agent' | 'chat';
  attachments?: MentionAttachment[];
  session_id: string;
  interaction_id: string;
  current_file_path?: string;
  current_file_selection?: string;
}
```

### Chat 响应格式 (SSE)

```
data: {"content": "Hello"}
data: {"thinking": "Let me think..."}
data: {"tool_call": {"id": "xxx", "name": "read_file", "arguments": {...}}}
data: {"tool_result": {"id": "xxx", "success": true, "output": "..."}}
data: {"done": true}
data: [DONE]
```

---

## 当前问题与改进计划

### feline-plugin 问题

| 类别 | 问题 | 影响 | 优先级 |
|------|------|------|--------|
| 架构 | whisker 是独立 Python 进程，需额外启动 | 部署复杂，资源占用高 | 中 |
| 配置 | Profile/API Key 存本地，无法跨设备同步 | 换设备需重新配置 | 高 |
| 安全 | API Key 明文存储 | 安全风险 | 高 |
| 功能 | 无 Token 用量统计 | 用户无法了解使用情况 | 中 |
| Thinking | 不支持 thinking/reasoning 模式 | 无法使用 DeepSeek-R1 等 | 中 |
| 重试 | 无自动重试和 Provider 切换 | 单点故障，可靠性低 | 高 |

### feline-server 问题

| 类别 | 问题 | 影响 | 优先级 |
|------|------|------|--------|
| 集成 | 未与 feline-plugin 对接 | 两套独立系统 | 高 |
| 订阅 | Subscription 表独立，未与 Usage 关联 | 无法自动扣费 | 高 |
| 限制 | 无 Token 使用限制检查 | 用户可超额使用 | 高 |
| 计费 | 无 Stripe 支付集成 | 无法商业化 | 中 |
| 实时 | 无 WebSocket 支持 | 无法实时推送消息 | 低 |
| 安全 | API Key 存配置文件，未加密 | 安全风险 | 高 |
| 限流 | 无速率限制 | 易被滥用 | 高 |
| 运维 | 无监控、日志聚合、告警 | 难以排查问题 | 中 |
| 迁移 | 无 Alembic 数据库迁移 | 表结构变更困难 | 中 |

---

## 未来规划

### 短期目标 (Q1)

1. **统一模型配置**
   - 移除 feline-plugin 中的用户模型配置
   - 所有 LLM 请求统一通过 feline-server
   - 用户只需配置 feline-server 的认证信息

2. **Plugin-Server 集成**
   - feline-plugin 支持连接 feline-server
   - 实现云端认证和会话同步
   - 支持云端订阅和用量统计

3. **安全加固**
   - API Key 加密存储
   - 实现 JWT 认证
   - 添加速率限制

### 中期目标 (Q2)

1. **订阅系统完善**
   - Subscription 与 Usage 关联
   - 自动扣费和额度检查
   - Stripe 支付集成

2. **监控与运维**
   - Prometheus 指标完善
   - 日志聚合（ELK/Loki）
   - 告警系统

3. **数据库迁移**
   - 引入 Alembic
   - 版本化管理数据库变更

### 长期目标 (Q3+)

1. **实时通信**
   - WebSocket 支持
   - 实时推送消息和状态

2. **多语言支持**
   - 国际化 UI
   - 多语言文档

3. **企业功能**
   - 团队管理
   - 权限控制
   - 审计日志

---

## 贡献指南

### 提交代码

1. Fork 仓库
2. 创建功能分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add amazing feature'`)
4. 推送分支 (`git push origin feature/amazing-feature`)
5. 创建 Pull Request

### 代码规范

- TypeScript: ESLint + Prettier
- Python: Black + isort + mypy
- 提交信息: Conventional Commits

### 分支策略

- `main`: 稳定版本
- `develop`: 开发版本
- `feature/*`: 功能分支
- `hotfix/*`: 紧急修复

---

## 联系方式

- GitHub Issues: [https://github.com/phanhom/feline/issues](https://github.com/phanhom/feline/issues)
- Email: feline@example.com
