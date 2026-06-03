# Pontus — LiteLLM Admin

LiteLLM 网关的桌面管理工具，用于用户注册与邀请管理。

**核心流程**: 创建 LiteLLM 用户 → 生成邀请链接 → 通过 SMTP 发送欢迎邮件

## 技术栈

| 层 | 技术 |
|---|---|
| 桌面框架 | Tauri 2 (Rust) |
| 前端 | Vue 3 + TypeScript |
| UI 组件 | shadcn-vue (Radix Vue) + Tailwind CSS |
| HTTP 客户端 | reqwest |
| 邮件发送 | lettre (SMTP) |

## 快速开始

```bash
npm install
npm run tauri dev      # 开发模式
npm run tauri build    # 生产构建
```

## 项目结构

```
src/                   # Vue 3 前端
  components/          # AppLayout, InitialSetup, ui/*
  views/               # InviteView, DashboardView, SettingsView
  stores/              # Pinia store (AppConfig)
  router/              # Vue Router (hash history)
src-tauri/             # Rust 后端
  src/commands/        # config, litellm, email, email_template
  tauri.conf.json      # 窗口与打包配置
docs/                  # 架构文档
```

首次启动进入两步引导设置：Litellm 连接配置 → SMTP 邮件配置（可跳过）。之后通过侧边栏导航进行用户邀请、看板查看和设置修改。

## 文档

- [架构与开发文档](docs/ARCHITECTURE.md)
