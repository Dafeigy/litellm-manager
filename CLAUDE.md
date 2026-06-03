# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

**Pontus** (Litellm Admin) — a Tauri 2 desktop app for managing Litellm AI gateway user onboarding. Core workflow: create a user via Litellm API → generate invitation link → send welcome email via SMTP.

**Stack**: Tauri 2 (Rust) + Vue 3 + TypeScript + shadcn-vue (Radix Vue) + Tailwind CSS

## Commands

```bash
npm install              # Install frontend dependencies
npm run tauri dev        # Dev mode (Vite + Tauri window)
npm run tauri build      # Production build
npx shadcn-vue@latest add <name>   # Add a shadcn-vue UI component
```

## Architecture

```
Vue 3 frontend (invoke)  ──IPC──>  Tauri 2 Rust backend  ──HTTP──>  Litellm API
                                   Tauri 2 Rust backend  ──SMTP──>  Email server
```

### Frontend (src/)

- **`main.ts`** — mounts Vue + Pinia + Router
- **`App.vue`** — checks `is_initialized_cmd` on mount; shows `InitialSetup` (API key entry) or `AppLayout` (sidebar + router-view)
- **Router** (`src/router/index.ts`) — hash-based, 3 routes: `/invite`, `/dashboard`, `/settings`; `/` redirects to `/invite`
- **Pinia store** (`stores/app.ts`) — holds `AppConfig` (camelCase) and `applyTheme()` which toggles `dark` class on `<html>`
- **Components**: `AppLayout.vue` (sidebar nav), `InitialSetup.vue` (first-run API key form), `ui/` (shadcn-vue components)
- **Views**: `InviteView.vue` (invite form), `DashboardView.vue` (paginated user table), `SettingsView.vue` (API/SMTP/theme config)

### Backend (src-tauri/src/)

- **`lib.rs`** — registers `tauri-plugin-store` and 9 commands
- **`commands/config.rs`** — `AppConfig` CRUD persisted via `tauri-plugin-store` to `config.json` (in app data dir); `AppConfig` uses `#[serde(rename_all = "camelCase")]` so JSON keys match the frontend
- **`commands/litellm.rs`** — proxies Litellm API (`POST /user/new`, `POST /invitation/new`, `GET /user/list`) using `reqwest::blocking`. Each operation has an internal sync function returning typed structs + an `async` Tauri command wrapping it in `tokio::task::spawn_blocking`. Litellm API responses use snake_case (no rename).
- **`commands/email.rs`** — SMTP sending via `lettre`; port 465 = implicit TLS, other ports = STARTTLS; `invite_user` orchestrates the full create-user → invite → email pipeline
- **`commands/email_template.rs`** — static HTML email template with `{username}`, `{user_email}`, `{invitation_link}`, `{api_key}` interpolation

### Data flow conventions

| Context | Naming | Example |
|---|---|---|
| `AppConfig` Rust struct | serialized as camelCase | `apiKey`, `litellmHost` |
| Litellm API response structs | snake_case (no rename) | `user_id`, `user_email` |
| Frontend TypeScript (config) | camelCase | `apiKey`, `litellmHost` |
| Frontend TypeScript (API responses) | snake_case | `user_id`, `user_email` |

### Security

API keys and SMTP credentials live only in the Rust backend. The frontend accesses them only through Tauri IPC commands. All external HTTP/SMTP calls happen from Rust.

### Key patterns

- **New Tauri command flow**: add internal fn in the relevant `commands/*.rs` → add `async` command wrapper with `spawn_blocking` → register in `lib.rs` `generate_handler![]` → call from frontend with `invoke("cmd_name", { args })`
- **Frontend calls**: `invoke()` from `@tauri-apps/api/core`; `toast.success()`/`toast.error()` from `vue-sonner` for feedback
- **Styling**: Tailwind with shadcn-vue CSS variables (defined in `src/assets/index.css`); `cn()` utility from `src/lib/utils.ts` merges classes; dark mode via `.dark` class on `<html>`

### Configuration storage

`config.json` stored in the Tauri app data directory (`%APPDATA%/com.litellm.admin/config.json` on Windows). `is_initialized_cmd` returns true when `api_key` is non-empty.
