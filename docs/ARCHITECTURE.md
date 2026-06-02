# Litellm Admin — 架构与开发文档

## 1. 项目概述

Litellm Admin 是一个基于 **Tauri 2 + Vue 3** 的桌面管理工具，用于管理 Litellm AI 网关的用户注册。核心功能：通过 Litellm API 创建用户 → 生成邀请链接 → 通过 SMTP 发送欢迎邮件。

**技术栈**: Tauri 2 (Rust) + Vue 3 + TypeScript + shadcn-vue (Radix Vue) + Tailwind CSS

---

## 2. 整体架构

```
┌──────────────────────────────────────────────────┐
│                  Vue 3 前端                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │ Invite   │ │Dashboard │ │ Settings         │  │
│  │ 邀请用户  │ │ 用户看板  │ │ SMTP/API/主题设置 │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────────────┘  │
│       │             │            │                │
│       └─────────────┴────────────┘                │
│                     │ invoke()                    │
├─────────────────────┼────────────────────────────┤
│              Tauri 2 Rust 后端                     │
│  ┌──────────────────┴──────────────────────────┐  │
│  │            Tauri Commands (IPC)             │  │
│  │  invite_user / create_user / list_users     │  │
│  │  get_config / save_config / send_email      │  │
│  └──────────────┬──────────────────────────────┘  │
│                 │                                  │
│  ┌──────────────┴──────────────┐                   │
│  │         Services            │                   │
│  │  reqwest → Litellm HTTP API │                   │
│  │  lettre  → SMTP 邮件服务    │                   │
│  │  tauri-plugin-store → 本地配置│                  │
│  └─────────────────────────────┘                   │
└──────────────────────────────────────────────────┘
         │                    │
         ▼                    ▼
   ┌──────────┐        ┌──────────┐
   │ Litellm  │        │   SMTP   │
   │   API    │        │  Server  │
   └──────────┘        └──────────┘
```

### 数据流方向

| 方向 | 说明 |
|------|------|
| 前端 → Rust | `invoke(cmd, args)` 调用 Tauri 命令 |
| Rust → Litellm | `reqwest` HTTP 请求（API Key 鉴权） |
| Rust → SMTP | `lettre` SMTP 发送（SSL 465 / STARTTLS 587） |
| Rust → 磁盘 | `tauri-plugin-store` 持久化配置到 `config.json` |
| Rust → 前端 | JSON 序列化返回值（snake_case 字段名） |

### 安全设计

- **API Key 和安全凭据仅存储在 Rust 后端**，前端无法直接访问
- 所有对外 HTTP 请求和 SMTP 通信均由 Rust 处理
- 前端通过 Tauri IPC 调用命令，不直接操作网络或文件

---

## 3. 项目目录结构

```
E:\Prog\Litellm\
├── package.json                  # Node.js 依赖 (Vue, shadcn-vue, Tauri CLI)
├── vite.config.ts                # Vite 构建配置 (端口 1420, @ 别名)
├── tsconfig.json                 # TypeScript 配置
├── tailwind.config.js            # Tailwind + shadcn-vue 主题变量
├── components.json               # shadcn-vue 配置
├── index.html                    # SPA 入口
│
├── src/                          # === Vue 3 前端 ===
│   ├── main.ts                   # 应用入口: 挂载 Vue + Pinia + Router
│   ├── App.vue                   # 根组件: 初始判断 → InitialSetup / AppLayout
│   ├── vite-env.d.ts             # TypeScript 声明
│   ├── router/index.ts           # 路由: /invite, /dashboard, /settings
│   ├── stores/app.ts             # Pinia Store: AppConfig + 主题切换
│   ├── lib/utils.ts              # cn() 工具函数 (clsx + tailwind-merge)
│   ├── assets/index.css          # Tailwind 指令 + shadcn CSS 变量
│   │
│   ├── components/
│   │   ├── AppLayout.vue         # 侧边栏导航布局 (3 个菜单项)
│   │   ├── InitialSetup.vue      # 首次启动 API Key 设置卡片
│   │   └── ui/                   # shadcn-vue 组件库
│   │       ├── Button.vue        # 按钮 (default/destructive/outline/ghost)
│   │       ├── Input.vue         # 文本输入框
│   │       ├── Label.vue         # 表单标签
│   │       ├── Select.vue        # 下拉选择 (基于 Radix Vue)
│   │       ├── Switch.vue        # 开关 (暗黑模式)
│   │       ├── Card.vue          # 卡片容器
│   │       ├── Dialog.vue        # 对话框 (基于 Radix Vue)
│   │       ├── Table.vue + TableHeader/TableBody/TableRow/TableHead/TableCell.vue
│   │       ├── Sonner.vue        # Toast 通知组件
│   │       ├── Separator.vue     # 分隔线
│   │       └── Skeleton.vue      # 骨架屏
│   │
│   └── views/
│       ├── InviteView.vue        # 主页面: 邀请用户表单
│       ├── DashboardView.vue     # 用户看板: 分页用户列表
│       └── SettingsView.vue      # 设置页: API/SMTP/主题配置
│
├── src-tauri/                    # === Rust 后端 ===
│   ├── Cargo.toml                # Rust 依赖 (tauri, reqwest, lettre, serde)
│   ├── tauri.conf.json           # Tauri 窗口 & 打包配置
│   ├── build.rs                  # Tauri 构建脚本
│   ├── capabilities/default.json # 权限声明 (core:default, store:default)
│   ├── icons/                    # 应用图标
│   └── src/
│       ├── main.rs               # Rust 入口
│       ├── lib.rs                # Tauri Builder: 注册插件和命令
│       └── commands/
│           ├── mod.rs            # 模块声明
│           ├── config.rs         # 配置管理 (CRUD + 持久化)
│           ├── litellm.rs        # Litellm API 调用 (用户/邀请 CRUD)
│           ├── email.rs          # SMTP 邮件发送 + invite_user 工作流
│           └── email_template.rs # HTML 邮件模板
│
└── requirements.md               # 原始需求文档
```

---

## 4. Rust 后端详解

### 4.1 lib.rs — 应用入口与命令注册

```rust
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_store::Builder::new().build())  // 本地持久化插件
        .invoke_handler(tauri::generate_handler![...])        // 注册 9 个 Tauri 命令
        .run(tauri::generate_context!())
}
```

### 4.2 config.rs — 配置存储

**核心数据结构**:
```rust
#[derive(Serialize, Deserialize, Clone)]
#[serde(rename_all = "camelCase")]  // Rust snake_case ↔ JS camelCase
pub struct AppConfig {
    pub api_key: String,          // Litellm API 鉴权密钥
    pub litellm_host: String,     // Litellm 服务地址 (默认 http://localhost:4000)
    pub smtp_host: String,        // SMTP 服务器
    pub smtp_port: u16,           // 端口 (465=SSL, 587=STARTTLS)
    pub smtp_sender_email: String, // 发件人邮箱
    pub smtp_username: String,    // SMTP 登录用户名
    pub smtp_password: String,    // SMTP 密码/授权码
    pub theme: String,            // "light" | "dark"
}
```

存储路径：`tauri-plugin-store` 在应用数据目录下生成 `config.json`。

**公开命令**:

| 命令 | 签名 | 说明 |
|------|------|------|
| `is_initialized_cmd` | `() → Result<bool>` | 检查 api_key 是否已设置 |
| `get_config_cmd` | `() → Result<AppConfig>` | 读取完整配置（解析失败回退默认值） |
| `save_config_cmd` | `(config: AppConfig) → Result<()>` | 保存配置 |
| `reset_api_key_cmd` | `() → Result<AppConfig>` | 清空 api_key，返回更新后的配置 |

### 4.3 litellm.rs — Litellm API 代理

**分层设计**：每个 API 操作拆分为内部函数 + 异步 Tauri 命令：

```
内部函数 (同步, 返回类型化结构体)       Tauri 命令 (async, 返回 serde_json::Value)
─────────────────────────────────      ──────────────────────────────────────────
create_user_internal()      ───────→  create_user()       [POST /user/new]
generate_invitation_internal() ────→  generate_invitation() [POST /invitation/new]
list_users_internal()       ───────→  list_users()        [GET /user/list?page=&page_size=]
```

**数据结构对照** (Litellm API 使用 snake_case):

| 结构体 | 用途 | 关键字段 |
|--------|------|---------|
| `CreateUserRequest` | 请求体 | user_email, user_alias, user_role |
| `CreateUserResponse` | 响应体 | user_id, key (API Key), user_email |
| `InvitationRequest` | 请求体 | user_id |
| `InvitationResponse` | 响应体 | id (即为 INVITATION_ID) |
| `UserInfo` | 用户信息 | user_id, user_email, user_role, spend, key_count |
| `UserListResponse` | 分页列表 | users: Vec\<UserInfo\>, total, page, total_pages |

**异步化机制**：
```rust
#[tauri::command]
pub async fn list_users(app: AppHandle, page: u32, page_size: u32) -> Result<Value, String> {
    let result = tokio::task::spawn_blocking(move || {
        list_users_internal(&app, page, page_size)  // 阻塞 I/O 在线程池中运行
    }).await.map_err(...)??;
    serde_json::to_value(&result)  // 转为 JSON 返回前端
}
```

所有阻塞 I/O（`reqwest::blocking`）通过 `tokio::task::spawn_blocking` 从主线程卸载，确保 UI 不冻结。

**调试日志**：每个 API 调用输出 `[函数名] status=..., body=...` 到 stderr。

### 4.4 email.rs — 邮件发送与邀请工作流

**`send_email()` 内部函数**：

```
                    ┌─ 端口 465 → SSL (SmtpTransport::relay)
                    │   + dangerous_accept_invalid_certs(true)
SMTP TLS 模式选择 ──┤
                    └─ 其他端口 → STARTTLS (SmtpTransport::starttls_relay)
```

TLS 证书处理：使用 `dangerous_accept_invalid_certs(true)` 兼容内网自签名 SMTP 证书。

**`invite_user` 工作流** (对前端暴露的单一命令)：

```
Step 1: create_user_internal()    → POST /user/new     → 获得 user_id + api_key
Step 2: generate_invitation_internal() → POST /invitation/new → 获得 invitation_id
Step 3: 构建邀请链接: {LITELLM_HOST}/ui?invitation_id={id}
Step 4: send_email()              → SMTP 发送 HTML 邮件
Step 5: 返回 InviteResult { user_id, api_key, invitation_id, invitation_link }
```

**`send_invite_email` 命令**：允许单独发送邀请邮件（已独立暴露给前端）。

**`validate_smtp_config()`**：发送前检查 SMTP 配置完整性，缺失项给出中文提示。

### 4.5 email_template.rs — HTML 邮件模板

移动端适配的 HTML 邮件，包含：
- 蓝色渐变头部（Litellm 品牌色 `#2563eb`）
- 个人信息卡片（注册邮箱 + API Key，API Key 在 code 块中方便复制）
- CTA 按钮：`点击完成注册` → 邀请链接
- 纯文本备选：邀请链接明文显示（邮件客户端不支持 HTML 时）

四个变量插值：`{username}`, `{user_email}`, `{invitation_link}`, `{api_key}`

---

## 5. Vue 前端详解

### 5.1 应用启动流程

```
main.ts                    App.vue (onMounted)
   │                           │
   ├─ createApp()              ├─ invoke("is_initialized_cmd")
   ├─ use(pinia)               │
   ├─ use(router)              ├─ 未初始化 ──→ <InitialSetup />
   └─ mount("#app")            │    └─ 用户输入 API Key
                               │    └─ invoke("save_config_cmd")
                               │    └─ emit("complete") → 切换到 AppLayout
                               │
                               └─ 已初始化 ──→ <AppLayout />
                                    ├─ invoke("get_config_cmd")
                                    ├─ applyTheme(config.theme)
                                    └─ <router-view /> 渲染当前页面
```

### 5.2 路由

| 路径 | 组件 | 说明 |
|------|------|------|
| `/` | 重定向 → `/invite` | — |
| `/invite` | InviteView.vue | 主页面：邀请用户表单 |
| `/dashboard` | DashboardView.vue | 用户看板：分页表格 |
| `/settings` | SettingsView.vue | 设置：API/SMTP/主题 |

使用 `createWebHashHistory`（`#/invite` 格式），兼容 Tauri 文件协议。

### 5.3 AppLayout.vue — 布局组件

左侧固定 224px 侧边栏 + 右侧自适应内容区。

侧边栏三个导航项：
1. **邀请用户** (UserPlus 图标) → `/invite`
2. **用户看板** (LayoutDashboard 图标) → `/dashboard`
3. **设置** (Settings 图标) → `/settings`

当前激活项高亮：`bg-primary/10 text-primary`。

### 5.4 Pinia Store (stores/app.ts)

```typescript
interface AppConfig {
  apiKey: string;        // 注意：前端使用 camelCase
  litellmHost: string;
  smtpHost: string;
  smtpPort: number;      // Rust u16 ↔ JS number
  smtpSenderEmail: string;
  smtpUsername: string;
  smtpPassword: string;
  theme: "light" | "dark";
}
```

**关键方法**：`applyTheme(theme)` — 操作 `document.documentElement.classList` 添加/移除 `dark` 类，触发 Tailwind 暗黑模式。

### 5.5 核心视图

#### InviteView.vue — 邀请用户

**表单字段**：
- 用户名（user_alias）：文本输入
- 用户邮箱（user_email）：email 类型输入，带格式校验
- 用户角色（user_role）：Radix Vue Select 下拉框，4 个选项

**提交流程**：
1. 前端校验 → 邮箱格式正则 `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`
2. `invoke("invite_user", { userEmail, userAlias, userRole })`
3. 成功 → toast 提示 + 清空表单
4. 失败 → toast 显示错误信息

**角色选项** (定义在 `roleOptions` 常量)：
| 值 | 显示名 | 说明 |
|----|--------|------|
| `proxy_admin` | 网关管理员 | 全部权限 |
| `proxy_admin_viewer` | 审计管理员 | 只读管理权限 |
| `internal_user` | 普通用户 | 标准使用权限 |
| `internal_user_viewer` | 普通只读用户 | 默认角色 |

#### DashboardView.vue — 用户看板

**核心功能**：
- 分页调用 `invoke("list_users", { page, pageSize: 25 })`
- 表格列：用户名 / 邮箱 / 角色 / Key 数量 / 消费 / 创建时间
- 分页控件：上一页 / 下一页按钮 + 页码显示
- 加载状态：Skeleton 骨架屏（首次加载时）
- 空状态：居中提示 "暂无用户数据"

**数据接口** (与 Rust 对应，字段名保持 snake_case)：
```typescript
interface UserListResponse {
  users: UserInfo[];
  total: number;
  page: number;
  page_size: number;
  total_pages: number;
}
```

#### SettingsView.vue — 设置页面

**三个配置卡片**：

| 卡片 | 字段 | 说明 |
|------|------|------|
| Litellm API | litellmHost, apiKey | 服务地址 + 鉴权密钥 |
| Litellm API | 重置 API Key 按钮 | 调用 `reset_api_key_cmd`，清空后回到 InitialSetup |
| SMTP 邮件 | smtpHost, smtpPort, smtpSenderEmail, smtpUsername, smtpPassword | 发件服务器配置 |
| 外观 | 暗黑模式开关 | `Switch` 组件，实时切换 `dark` 类 |

**保存流程**：`invoke("save_config_cmd", { config })` → 更新 Pinia Store → 应用主题。

### 5.6 shadcn-vue UI 组件库

所有组件在 `src/components/ui/` 下，基于 **Radix Vue** 无样式组件 + **Tailwind CSS** 样式。

| 组件 | 文件 | 基础库 | 关键 Props |
|------|------|--------|-----------|
| Button | Button.vue | 原生 `<button>` | variant, size, disabled |
| Input | Input.vue | 原生 `<input>` | modelValue (v-model), type, disabled |
| Label | Label.vue | 原生 `<label>` | for |
| Select | Select.vue | Radix SelectRoot/Trigger/Content/Value | modelValue, placeholder, disabled |
| Switch | Switch.vue | Radix SwitchRoot/Thumb | modelValue (boolean) |
| Card | Card.vue | 原生 `<div>` | — |
| Table | Table.vue + 5 个子组件 | 原生 `<table>` | — |
| Sonner | Sonner.vue | vue-sonner | — |
| Separator | Separator.vue | 原生 `<div>` | orientation, label |
| Skeleton | Skeleton.vue | 原生 `<div>` | — |

**组件使用约定**：
- `v-model` 用于双向绑定（Input, Select, Switch）
- `toast.success()` / `toast.error()` 用于操作反馈
- 通过 Tailwind class 传递自定义样式，不走内联 style

---

## 6. Rust ↔ 前端数据交互

### 字段命名约定

| 层级 | 命名风格 | 示例 |
|------|---------|------|
| `AppConfig` (Rust 内部存储) | snake_case + `#[serde(rename_all = "camelCase")]` | api_key → JSON: apiKey |
| Litellm API 响应结构体 | snake_case (默认) | user_id, user_email |
| TypeScript 接口 (API 响应) | snake_case | user_id, user_email |
| TypeScript 接口 (AppConfig) | camelCase | apiKey, litellmHost |

### Tauri 命令调用模式

```typescript
// 前端调用
const result = await invoke("command_name", { arg1: value1, arg2: value2 });

// Rust 接收
#[tauri::command]
pub async fn command_name(app: AppHandle, arg1: String, arg2: u32) -> Result<serde_json::Value, String>
```

- `app: AppHandle` 由 Tauri 自动注入，不需要前端传递
- 返回值 `Result<T, String>`：`Ok` 序列化为 JSON 返回，`Err` 作为异常抛出
- 同步命令阻塞主线程 → 所有 I/O 命令标记为 `async` + `spawn_blocking`

---

## 7. 依赖清单

### Rust (Cargo.toml)

| Crate | 版本 | 用途 |
|-------|------|------|
| tauri | 2.x | 桌面框架 |
| tauri-plugin-store | 2.x | 本地 JSON 文件存储 |
| reqwest | 0.12 (blocking) | HTTP 客户端 (调用 Litellm API) |
| lettre | 0.11 (tokio1-native-tls) | SMTP 邮件发送 |
| serde / serde_json | 1.x | JSON 序列化/反序列化 |
| tokio | 1.x (full) | 异步运行时 (spawn_blocking) |

### Node.js (package.json)

| 包 | 用途 |
|----|------|
| vue / vue-router / pinia | 前端框架 |
| radix-vue | 无样式可访问组件原语 |
| lucide-vue-next | 图标库 |
| vue-sonner | Toast 通知 |
| tailwindcss / tailwindcss-animate | CSS 框架 |
| clsx / tailwind-merge | class 合并工具 |
| @tauri-apps/api / @tauri-apps/plugin-store | Tauri 前端 API |

---

## 8. 开发指南

### 启动开发环境

```bash
npm install              # 安装前端依赖
npm run tauri dev        # 启动 Vite + Tauri 开发窗口
```

### 构建生产版本

```bash
npm run tauri build      # 编译 Rust + 打包前端 → 生成安装包
```

### 调试

- **Rust 日志**：`eprintln!()` 输出到终端 stderr，启动 `tauri dev` 时可见
- **前端日志**：浏览器 DevTools Console（`console.log`）
- **配置存储路径**：`%APPDATA%/com.litellm.admin/config.json` (Windows)

### 添加新功能的步骤

1. **新增 API 调用**：在 `litellm.rs` 中添加入口函数 → 内部函数 + Tauri 命令模式
2. **注册命令**：在 `lib.rs` 的 `generate_handler![]` 中添加
3. **前端调用**：`import { invoke } from "@tauri-apps/api/core"` → `await invoke("cmd", args)`
4. **添加 UI 组件**：`npx shadcn-vue@latest add <name>` 或手动写入 `src/components/ui/`

### 添加新 shadcn-vue 组件

```bash
# 标准方式（将组件写入 src/components/ui/）
npx shadcn-vue@latest add dialog

# 手动方式（创建 .vue 文件，基于 radix-vue 构建）
# 参考现有组件模式：<script setup> + <template> + Tailwind class
```

---

## 9. 常见问题

### API Key 不持久化 → 每次启动都要求输入

原因：Rust `AppConfig` 的 `#[serde(rename_all = "camelCase")]` 与前端字段名不匹配。确认该注解存在且前端 `AppConfig` 接口使用 camelCase。

### 设置页保存无响应

原因：UI 组件的 `inheritAttrs: false` 阻止了 `@click` 事件传递。检查相关组件是否已移除该选项。

### API 调用报 "Parse error: error decoding response body"

原因：Litellm API 返回 snake_case，但 Rust 结构体标注了 `#[serde(rename_all = "camelCase")]`。检查 `litellm.rs` 中的响应结构体是否有此注解。

### 邮件发送报 "response error: incomplete response"

原因：SMTP 端口/协议不匹配。检查 SMTP 端口是否正确（465=SSL 隐式 TLS，587=STARTTLS），以及用户名是否为完整邮箱地址。

### 程序卡死

原因：同步 Tauri 命令阻塞主线程。确认所有网络 I/O 命令标记为 `async` 且使用 `tokio::task::spawn_blocking`。
