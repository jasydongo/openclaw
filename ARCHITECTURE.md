# 🏗️ OpenClaw 项目架构分析

## 📊 项目概览

**OpenClaw** 是一个**个人 AI 助手**系统，支持多渠道消息交互（WhatsApp、Telegram、Discord、Slack 等），可以在用户的自有设备上运行，提供本地化、快速、始终在线的 AI 助手体验。

**核心特点：**
- 个人单用户助手（非多用户 SaaS）
- 支持多种消息渠道
- 跨平台支持（CLI、macOS、iOS、Android、Web）
- 本地优先的数据存储
- 可扩展的插件系统

---

## 🗂️ 整体架构分层

```
┌─────────────────────────────────────────────────────────────┐
│                     用户界面层 (UI Layer)                      │
├─────────────────────────────────────────────────────────────┤
│  CLI  │  macOS App  │  iOS App  │  Android App  │  Web UI   │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   核心控制层 (Core Layer)                     │
├─────────────────────────────────────────────────────────────┤
│  CLI 入口  │  Gateway 服务  │  TUI 终端界面  │  ACP 协议     │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                  业务逻辑层 (Business Logic)                   │
├─────────────────────────────────────────────────────────────┤
│  Agent 编排  │  工具执行  │  沙箱隔离  │  技能系统          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                 渠道接入层 (Channel Layer)                     │
├─────────────────────────────────────────────────────────────┤
│  Telegram │ WhatsApp │ Discord │ Slack │ Signal │ iMessage │
│  扩展渠道: Matrix, Zalo, BlueBubbles, MS Teams, ...        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                模型提供商层 (Provider Layer)                  │
├─────────────────────────────────────────────────────────────┤
│  Anthropic │ OpenAI │ GitHub Copilot │ Google Gemini │ ... │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│               基础设施层 (Infrastructure Layer)               │
├─────────────────────────────────────────────────────────────┤
│  配置管理  │  会话存储  │  记忆系统  │  文件系统  │  网络     │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 核心目录结构

### 1. **`src/`** - 核心源代码

#### **`cli/`** - CLI 入口
- `program.ts` - 命令行程序定义
- `deps.ts` - 依赖注入
- `progress.ts` - 进度显示
- `prompt.ts` - 用户交互提示

#### **`gateway/`** - Gateway 服务
- HTTP/WebSocket 服务器
- 控制面板 API
- 设备配对管理
- 会话管理

#### **`agents/`** - Agent 编排系统
**核心子模块：**
- `pi-embedded-runner/` - Pi 嵌入式 AI 运行器
- `pi-embedded-subscribe/` - 流式响应订阅
- `pi-tools/` - Pi 工具适配
- `bash-tools/` - Bash 工具执行
- `sandbox/` - 沙箱环境管理
- `skills/` - 技能系统
- `tools/` - 工具定义
- `model-*.ts` - 模型相关（选择、认证、回退、目录）
- `memory-search/` - 记忆搜索
- `compaction/` - 上下文压缩
- `subagent-*.ts` - 子代理管理
- `auth-profiles/` - 认证配置轮换

#### **`channels/`** - 消息渠道系统
- `registry.ts` - 渠道注册表
- `telegram/` - Telegram Bot
- `discord/` - Discord Bot
- `slack/` - Slack
- `signal/` - Signal
- `imessage/` - iMessage (macOS)
- `web/` - WhatsApp Web
- `plugins/` - 插件化渠道
- `allowlists/` - 白名单管理
- `command-gating.ts` - 命令 gating
- `mention-gating.ts` - 提及 gating
- `model-overrides.ts` - 模型覆盖

#### **`providers/`** - AI 模型提供商
- `github-copilot-*.ts` - GitHub Copilot 集成
- `qwen-portal-oauth.ts` - 通义千问 OAuth
- `google-*.ts` - Google 共享功能

#### **`memory/`** - 记忆系统
- `manager.ts` - 记忆管理器
- `embeddings-*.ts` - 向量嵌入
- `qmd-manager.ts` - QMD 记忆管理
- `sqlite-vec.ts` - SQLite 向量搜索
- `hybrid.ts` - 混合搜索
- `temporal-decay.ts` - 时间衰减

#### **`config/`** - 配置管理
- `config.ts` - 主配置加载
- `schema.ts` - 配置 schema
- `sessions/` - 会话配置
- `types.*.ts` - 各模块类型定义
- `legacy-*.ts` - 遗留配置迁移

#### **`plugins/`** - 插件系统
- `runtime.ts` - 插件运行时
- `registry.ts` - 插件注册表
- 插件发现和加载

#### **其他核心模块：**
- `tui/` - 终端 UI
- `acp/` - Agent Client Protocol
- `canvas-host/` - Canvas 渲染主机
- `media/` - 媒体处理
- `link-understanding/` - 链接理解
- `media-understanding/` - 媒体理解
- `browser/` - 浏览器自动化
- `tts/` - 文本转语音
- `auto-reply/` - 自动回复
- `cron/` - 定时任务
- `security/` - 安全模块
- `wizard/` - 配置向导

---

### 2. **`extensions/`** - 扩展插件

**渠道扩展：**
- `msteams` - Microsoft Teams
- `matrix` - Matrix 协议
- `zalo` / `zalouser` - Zalo
- `feishu` - 飞书
- `googlechat` - Google Chat
- `line` - LINE
- `nostr` - Nostr
- `nextcloud-talk` - Nextcloud Talk
- `irc` - IRC
- `twitch` - Twitch
- `voice-call` - 语音通话

**功能扩展：**
- `memory-core` / `memory-lancedb` - 记忆后端
- `device-pair` - 设备配对
- `copilot-proxy` - Copilot 代理
- `google-antigravity-auth` - Google 认证
- `minimax-portal-auth` - MiniMax 认证
- `llm-task` - LLM 任务
- `lobster` - UI 扩展
- `open-prose` - Prose 集成

---

### 3. **`apps/`** - 原生应用

- `android/` - Android 应用（Kotlin）
- `ios/` - iOS 应用（Swift）
- `macos/` - macOS 应用（Swift）
- `shared/` - 共享代码

---

### 4. **`ui/`** - Web UI

- React + Vite 前端
- Gateway 控制面板
- 配置界面

---

### 5. **`skills/`** - 技能包

大量预定义技能：
- GitHub 集成
- 日历管理
- 笔记同步
- 音乐控制
- 天气查询
- 等等...

---

### 6. **`packages/`** - Monorepo 包

- `clawdbot` - Clawdbot 包
- `moltbot` - Moltbot 包

---

## 🔧 核心技术栈

### **后端核心**
- **运行时**: Node.js 22+
- **语言**: TypeScript (ESM)
- **包管理**: pnpm
- **构建**: tsdown (Rolldown)
- **测试**: Vitest
- **Lint**: Oxlint + Oxfmt

### **移动端**
- **iOS/macOS**: Swift + SwiftUI
- **Android**: Kotlin + Jetpack Compose

### **Web UI**
- React + Vite
- TypeScript

### **AI 集成**
- Anthropic Claude (Claude Pro/Max)
- OpenAI GPT
- GitHub Copilot
- Google Gemini
- 其他提供商（通过插件）

### **数据存储**
- SQLite (会话、配置)
- 向量数据库 (记忆)
- 文件系统 (技能、缓存)

### **通信协议**
- WebSocket (实时通信)
- HTTP REST API
- Agent Client Protocol (ACP)
- 各渠道原生协议

---

## 🔄 核心工作流程

### **1. 消息接收流程**
```
用户消息 → 渠道 (WhatsApp/Discord/...) 
         → 渠道适配器 
         → 消息路由器 
         → Agent 编排器 
         → 模型提供商 
         → 工具执行器 
         → 响应生成 
         → 渠道发送回用户
```

### **2. Agent 执行流程**
```
用户请求 → Pi 嵌入式运行器
         → 系统提示构建
         → 工具定义生成
         → 模型调用
         → 流式响应处理
         → 工具调用执行
         → 上下文压缩
         → 响应返回
```

### **3. 插件加载流程**
```
启动 → 扫描 extensions/ 
      → 加载插件 package.json
      → 注册渠道/工具/技能
      → 运行时注册表更新
      → 功能可用
```

---

## 🎯 架构特点

### **1. 高度模块化**
- 清晰的层次结构
- 插件化扩展机制
- 松耦合设计

### **2. 多渠道统一抽象**
- 统一的渠道接口
- 路由和 gating 系统
- 跨渠道一致性

### **3. 可扩展性**
- 插件系统支持第三方扩展
- 技能系统支持自定义能力
- 模型提供商可插拔

### **4. 本地优先**
- 本地数据存储
- 本地配置管理
- 隐私保护

### **5. 跨平台支持**
- CLI 通用
- 原生移动应用
- Web 控制面板

### **6. 智能编排**
- Agent 子代理系统
- 上下文压缩
- 记忆检索
- 工具策略

---

## 🔐 安全机制

- 配置加密存储
- 密钥扫描（detect-secrets）
- 白名单/黑名单
- 命令 gating
- 沙箱隔离

---

## 📦 部署方式

1. **npm 全局安装**
2. **Docker 容器**
3. **从源码构建**
4. **Nix flakes**
5. **原生应用包**

---

## 🎨 设计哲学

1. **个人优先** - 为单用户设计，不是多用户 SaaS
2. **本地化** - 数据本地存储，隐私保护
3. **渠道中立** - 支持用户喜欢的任意渠道
4. **可扩展** - 插件和技能系统支持无限扩展
5. **开发者友好** - 清晰的架构，良好的文档

---

## 📊 技术决策

### **为什么选择 Node.js/TypeScript？**
- 丰富的生态系统
- 跨平台支持
- 类型安全
- 易于扩展

### **为什么选择插件架构？**
- 核心保持轻量
- 社区可贡献
- 功能模块化
- 易于维护

### **为什么支持多渠道？**
- 用户自由选择
- 统一体验
- 渠道无关性
- 扩展性强

---

## 🚀 性能优化

1. **流式响应** - 实时反馈
2. **上下文压缩** - 减少token使用
3. **缓存机制** - 加速响应
4. **并行执行** - 工具并发
5. **增量更新** - 只传输变化

---

## 📈 可观测性

- 结构化日志
- 性能指标
- 错误追踪
- 使用统计
- 会话记录

---

这是一个设计精良、架构清晰、高度可扩展的个人 AI 助手系统，体现了现代软件工程的优秀实践。