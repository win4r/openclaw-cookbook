# 🦞 OpenClaw Cookbook

**你的 AI 智能体网关，运行在你的机器上，通过任何渠道对话，使用你选择的模型。**

OpenClaw 是一个自托管的多渠道 AI 智能体网关。这本 Cookbook 提供生产级的配方，帮你从零搭建完整的 AI 智能体系统 —— 可以直接复制的配置、可以直接运行的 docker-compose、可以直接上线的场景方案。

> **这不是文档。** [官方文档](https://github.com/anthropics/openclaw)覆盖了每个参数和选项。这本 Cookbook 解决的是"怎么真正把东西做出来" —— 模式、踩坑、以及在生产环境验证过的配方。

[English Version](./README.md)

---

## 你能用它做什么

```
┌──────────────────────────────────────────────────────────────────────┐
│                          你的用户和设备                                │
│  Telegram · WhatsApp · Discord · Slack · Twitch · iMessage · Web     │
└──────────────┬──────────────────────────────────────────┬────────────┘
               │                                          │
               ▼                                          ▼
┌──────────────────────────────────────────────────────────────────────┐
│                          OpenClaw 网关                                │
│                                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                          │
│  │  智能体A  │  │  智能体B  │  │  智能体C  │  路由与绑定 · 鉴权       │
│  │  (客服)   │  │  (编程)   │  │  (研究)   │                          │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                          │
│       │              │              │                                 │
│  ┌────┴──────────────┴──────────────┴─────┐                         │
│  │          模型供应商池 (35+)               │                         │
│  │  Claude · GPT · Gemini · DeepSeek       │                         │
│  │  MiniMax · Kimi · Ollama · vLLM         │                         │
│  └────────────────────────────────────────┘                         │
│                                                                      │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌───────────┐ │
│  │ 记忆  │  │ 工具  │  │ 调度  │  │ 插件  │  │ 节点  │  │ 控制面板   │ │
│  │LanceDB│  │ exec │  │ cron │  │ npm  │  │iOS/An│  │  Web UI   │ │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘  └───────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

**一个网关。多个智能体。任意模型。所有渠道。你的设备。**

---

## Cookbook 模块

| # | 模块 | 你将构建 | 时间 |
|---|------|---------|------|
| **01** | [快速开始](./01-quickstart/) | 一行命令运行 OpenClaw 实例 | 15 分钟 |
| **02** | [渠道接入](./02-channels/) | Telegram、Discord、Slack、WhatsApp、Twitch、iMessage | 每个 30 分钟 |
| **03** | [模型供应商](./03-model-providers/) | 多模型配置 + 降级链 + 路由策略 | 20 分钟 |
| **04** | [多智能体](./04-multi-agent/) | 专业化智能体、绑定路由 + ClawTeam 蜂群协调 | 45 分钟 |
| **05** | [记忆系统](./05-memory/) | LanceDB 长期记忆 + 语义召回 | 30 分钟 |
| **06** | [工具系统](./06-tools/) | 自定义工具、exec 审批、MCP 集成 | 40 分钟 |
| **07** | [安全加固](./07-security/) | SOUL.md 防护、鉴权、exec 白名单 | 30 分钟 |
| **08** | [工作空间](./08-workspace/) | AGENTS.md、TOOLS.md、IDENTITY.md — 人格编程 | 30 分钟 |
| **09** | [生产部署](./09-production/) | 监控、日志、多用户隔离、成本控制 | 45 分钟 |
| **10** | [实战配方](./10-recipes/) | 端到端可运行的完整场景 | 每个 30 分钟 |
| **11** | [定时任务](./11-scheduler/) | Cron 作业、提醒、心跳、自动化任务 | 25 分钟 |
| **12** | [设备节点](./12-nodes/) | iOS、Android 和 macOS 设备集成 | 30 分钟 |
| **13** | [控制面板](./13-control-ui/) | 浏览器 Web 管理面板 | 15 分钟 |

**全部掌握约需 8 小时。** 或者直接跳到[实战配方](#实战配方)，边做边学。

---

## 5 分钟体验

### 安装

```bash
# 安装 OpenClaw
npm install -g openclaw

# 验证
openclaw --version
```

### 配置第一个智能体

```bash
# 初始化工作空间
openclaw setup

# 添加模型供应商密钥
echo 'ANTHROPIC_API_KEY=sk-ant-...' >> ~/.openclaw/.env

# 启动网关
openclaw gateway
```

### 开始对话

```bash
# CLI 模式
openclaw message send "你好，你能做什么？"

# 或者接入 Telegram（详见 02-channels/）
openclaw plugins enable telegram
openclaw config set telegram.bots.main.token "你的BOT_TOKEN"
openclaw gateway --restart
```

搞定。你的私有 AI 智能体已经在运行了。接下来让它变得有用 —— 选一个模块开始，或者直接跳到配方。

---

## 实战配方

完整、可运行的端到端场景。每个配方包含所有配置文件和验证清单。

| 配方 | 描述 | 难度 | 时间 |
|------|------|------|------|
| [Telegram 个人 AI](./10-recipes/personal-ai-on-telegram/) | 带记忆、搜索、个性化的私人 AI 助手 | 入门 | 30 分钟 |
| [客户支持机器人](./10-recipes/customer-support-bot/) | 多语言客服 + 升级规则 + 知识库 | 中级 | 30 分钟 |
| [代码审查团队](./10-recipes/code-review-team/) | 多智能体蜂群：安全 + 性能专家协作审查 | 高级 | 45 分钟 |
| [研究助手](./10-recipes/research-assistant/) | 网络调研、综合分析、结构化报告 + 定时推送 | 中级 | 30 分钟 |

---

## 为什么选 OpenClaw

| | 云端 AI API | ChatGPT/Claude 应用 | **OpenClaw** |
|---|---|---|---|
| **数据留在本地** | 否 | 否 | 是 |
| **使用任意模型** | 一次一个 | 不行 | 是，支持降级链 |
| **多渠道** | 自己搭 | 不行 | 内置 |
| **多专业智能体** | 自己搭 | 不行 | 内置 |
| **长期记忆** | 自己搭 | 有限 | LanceDB + 语义搜索 |
| **自定义工具** | 自己搭 | 有限 | Exec + MCP + 插件 |
| **成本** | 按 token 付费 | 每月 $20/应用 | 只需你自己的 API key |

---

## 核心概念

### 工作空间 — 智能体的大脑

```
~/.openclaw/workspace/
├── SOUL.md          # 人格与价值观
├── AGENTS.md        # 运行规则与策略
├── TOOLS.md         # 工具使用说明
├── IDENTITY.md      # 名字、头像、标识
├── USER.md          # 智能体对你的了解
├── MEMORY.md        # 长期记忆笔记
├── BOOT.md          # 启动健康检查
├── HEARTBEAT.md     # 周期性健康检查（可选）
├── skills/          # 可复用技能定义
└── checklists/      # 操作前安全检查
```

每个 Markdown 文件都在塑造智能体的思维和行为。这就是**人格编程** —— 用散文而非代码定义智能体的性格。

### 智能体 — 专业化人格

一个网关可以承载多个智能体，每个有自己的工作空间、模型和渠道绑定：

```
智能体: "客服"     → Claude      → Telegram @support_bot
智能体: "程序员"   → GPT-5      → Telegram @code_bot
智能体: "研究员"   → Gemini      → Discord Bot
```

### 插件 — 扩展一切

```bash
# 内置插件
openclaw plugins enable telegram
openclaw plugins enable memory-lancedb-pro

# 社区插件 (npm)
npm install -g @tencent-weixin/openclaw-weixin
openclaw plugins enable openclaw-weixin
```

---

## 前置条件

- **Node.js** 24（推荐）或 22.14+
- **一个 API Key**（Anthropic、OpenAI、Google、MiniMax 等任一供应商）
- **5 分钟**开始体验，或约 8 小时完成全部内容

可选：
- **Telegram 账号**（最常用的渠道集成）
- **tmux**（仅 ClawTeam 蜂群需要）
- **Python 3.10+**（仅 ClawTeam 需要）

---

## 参与贡献

发现问题？有配方创意？欢迎 PR。

- **添加配方**：在 `10-recipes/` 下创建新目录，包含 README、配置文件和验证步骤
- **修正错误**：开 issue 或 PR —— 准确性比覆盖面更重要
- **翻译**：我们在做中英双语 —— 欢迎翻译任何模块

---

## 许可证

MIT - 详见 [LICENSE](./LICENSE)

---

由 [CortexReach](https://github.com/CortexReach) 构建。基于 [OpenClaw](https://github.com/anthropics/openclaw) 驱动。
