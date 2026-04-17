# Commit Scopes

本文件定義 prinsur-web、prinsur-api、prinsur-ai 三個 repo 各自的 commit scope 清單。

## Principles

1. **Scope 對齊 code topology**，不是 Linear labels。Scope 回答「程式碼哪一塊改了」，Linear label 回答「這個工作屬於哪類」，兩者是不同軸。
2. **每個 repo 擁有獨立的 scope 清單**，不跨 repo 共用。相同字（例如 `api`）在不同 repo 代表不同含義，由 repo 上下文區分。
3. **從固定清單中選用**，禁止臨時發明 scope。若現有 scope 不足以描述變更，改動本文件加入新 scope 後再提交 commit。
4. **一個 commit 只選一個 scope**。若跨多個 scope，優先使用主要改動的 scope；若真的跨越過半清單，省略 scope（使用 `type: subject` 格式）。
5. **Scope 與 Linear label 解耦**。Linear labels（如 `deps`, `infra`, `insurance`）自由成長用於 issue 分類，**不強制**等同 commit scope。兩者語意有重疊時純屬巧合。
6. **`deps` 是唯一不對齊 code topology 的例外 scope**。它是語意切面，專用於所有依賴為核心的變更（Dependabot/Renovate 自動產出、手動新增/升級/降級/移除、security patch）。三個 repo 皆有 `deps` scope 且語意相同，只是對應的依賴檔不同。其餘 scope 一律對齊目錄結構。

---

## prinsur-web

Frontend repo，Next.js 16 + React 19 + TailwindCSS v4 + shadcn/ui。Scope 來源對應 `src/` 目錄結構與功能模組。

| Scope       | 對應範圍                                                                                                                               | 使用時機                                                                                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app`       | `src/app/[locale]/`                                                                                                                    | App Router 的 page、layout、loading、error、route handler                                                                                                               |
| `ui`        | `src/components/ui/`                                                                                                                   | shadcn/ui 原子組件、基礎視覺元件                                                                                                                                        |
| `chat`      | `src/components/chat/`、`src/components/ai-elements/`                                                                                  | AI 對話介面、streaming UI、conversation list                                                                                                                            |
| `insurance` | `src/components/insurance/`、保險產品頁面                                                                                              | 保險商品瀏覽、比較、詳情頁                                                                                                                                              |
| `auth`      | `src/components/auth/`                                                                                                                 | 登入、註冊、忘記密碼、驗證碼、session UI                                                                                                                                |
| `landing`   | `src/components/landing/`、`src/components/sections/`                                                                                  | 行銷頁、首頁區塊、靜態內容頁                                                                                                                                            |
| `i18n`      | `src/i18n/`、`src/app/[locale]/` 內翻譯檔                                                                                              | 翻譯字串、locale 設定、next-intl 相關                                                                                                                                   |
| `api`       | `src/lib/api/`、`src/lib/generated/`                                                                                                   | 後端 API client、OpenAPI 生成的型別、API hooks                                                                                                                          |
| `common`    | `src/components/common/`、`src/providers/`、`src/types/`、`src/hooks/` 跨功能 hook、`src/stores/` 跨功能 store、`src/lib/` 非 api 部分 | 跨功能 / 無特定 domain 的元件、hook、store、lib utility。例：`ErrorBoundary`、`ThemeToggle`、`use-debounce`、`ui-store`、`logger.ts`、`error-tracking.ts`、shared types |
| `layouts`   | `src/components/layouts/`                                                                                                              | 頁面骨架元件：`Header`、`Footer`、`MainLayout`、`SplitPanelLayout`、`MobileAppLayout`                                                                                   |
| `infra`     | 根目錄設定、CI/CD、Docker、部署                                                                                                        | `next.config.ts`、`tsconfig.json`、`.github/workflows/`、環境變數、Dockerfile                                                                                           |
| `deps`      | 依賴檔與依賴自動化設定                                                                                                                 | `package.json`、`pnpm-lock.yaml`、`.github/dependabot.yml`；Dependabot 產出、手動新增/升級/降級/移除依賴、security patch                                                |

**歸屬原則：** 功能特定的 hook / store / helper 跟著功能走（例：`use-insurance-list.ts` → `insurance`、`conversation-store.ts` → `chat`、`use-profile-form.ts` → `auth`）；跨功能、無特定 domain 的才進 `common`。

### 常見歷史 scope 映射

| 舊 scope                            | 新 scope                                                                            |
| ----------------------------------- | ----------------------------------------------------------------------------------- |
| `web`, `frontend`                   | 功能元件 → 對應 feature scope；通用元件/hook/store → `common`；頁面骨架 → `layouts` |
| `ui` 用於 chat 相關                 | `chat`                                                                              |
| `ai`, `ai-chat`                     | `chat`                                                                              |
| `conversation`, `profile`           | 依內容：UI 改 → 對應元件 scope；API 串接 → `api`                                    |
| `hooks`, `stores`, `providers`      | 功能特定 → 跟 feature scope；跨功能 → `common`                                      |
| `ci`, `cd`, `build`, `cors`         | `infra`（type 選 `build`/`ci`/`chore` 視情況）                                      |

> TODO: 過段時間這套規範穩定之後把這邊移除。

---

## prinsur-api

Backend repo，Go + Gin + GORM。Scope 來源對應 `internal/` 目錄下的 module 與共用層。

| Scope          | 對應範圍                                                                                      | 使用時機                                                                                                                                                                      |
| -------------- | --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `admin`        | `internal/modules/admin/`                                                                     | 後台管理功能（帳號管理、內容管理介面）                                                                                                                                        |
| `agent`        | `internal/modules/agent/`                                                                     | 保險顧問、agent 相關邏輯                                                                                                                                                      |
| `conversation` | `internal/modules/conversation/`                                                              | 對話紀錄、conversation 管理、title 生成                                                                                                                                       |
| `customer`     | `internal/modules/customer/`                                                                  | 使用者 profile、customer 領域邏輯                                                                                                                                             |
| `insurance`    | `internal/modules/insurance/`                                                                 | 保險商品、分類、公司、內容                                                                                                                                                    |
| `auth`         | `internal/shared/auth/`、`internal/shared/session/`                                           | 登入、JWT、session、密碼、OAuth                                                                                                                                               |
| `storage`      | `internal/shared/storage/`                                                                    | S3/物件儲存、檔案上傳                                                                                                                                                         |
| `verification` | `internal/shared/verification/`                                                               | 驗證碼、email/sms 驗證                                                                                                                                                        |
| `db`           | `internal/infrastructure/db/`、`migrations/`                                                  | DB connection、GORM 設定、migration 檔。migration 的 **type** 按意圖選：新功能用 `feat(db)`、修 bug 用 `fix(db)`、內部重構不動 schema 行為用 `refactor(db)`，不固定為 `chore` |
| `http`         | `internal/infrastructure/http/`、`internal/infrastructure/middleware/`、`api/v1/`             | HTTP server 設定、middleware、CORS、router 組裝、handler 層共用                                                                                                               |
| `infra`        | `internal/infrastructure/config/`、`internal/infrastructure/di/`、`cmd/`、CI/CD、Docker、部署 | 環境設定、DI 容器、entry point、`.github/workflows/`、`Dockerfile`、`railway.toml`                                                                                            |
| `deps`         | 依賴檔與依賴自動化設定                                                                        | `go.mod`、`go.sum`、`.github/dependabot.yml`；Dependabot 產出、手動新增/升級/降級/移除依賴、security patch                                                                    |

### 常見歷史 scope 映射

| 舊 scope                                                                     | 新 scope                                                     |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `backend`, `api`                                                             | 依內容選具體 module scope；若跨多個或是共通 handler → `http` |
| `cors`                                                                       | `http`                                                       |
| `middleware`                                                                 | `http`                                                       |
| `database`, `config`                                                         | `db`（schema/連線）或 `infra`（env 設定）                    |
| `cd`, `build`, `ci`, `cmd`, `di`, `dependencies`, `type`, `notifier`         | `infra`                                                      |
| `deps` (historical)                                                          | `deps`（保留為獨立 scope，不再合併）                         |
| `health`, `handlers`                                                         | `http`                                                       |
| `profile`                                                                    | `customer`                                                   |
| `chat`                                                                       | `conversation` 或 `agent` 視內容                             |
| `pdfextract`                                                                 | 已移出為 serverless function，不再適用本 repo                |

> TODO: 過段時間這套規範穩定之後把這邊移除。

---

## prinsur-ai

AI repo，Python + FastAPI + pydantic-ai + Gemini。Scope 來源對應 `app/` 下的層級與 agent 結構。

| Scope    | 對應範圍                                                                          | 使用時機                                                     |
| -------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `chat`   | chat endpoint、streaming、Vercel AI adapter                                       | `/chat` route、stream 格式、SSE 協議                         |
| `agent`  | `app/services/ai/`、pydantic-ai agents、tools、prompt                             | agent 定義、tool 函式、prompt 模板、suggestion agent         |
| `api`    | `app/api/routes/`、`app/schemas/`                                                 | FastAPI route 層、request/response schema（非 LLM 直接使用） |
| `core`   | `app/core/`                                                                       | config、logging、middleware、CORS、dependency injection      |
| `models` | `app/models/`                                                                     | 資料結構、DTO、領域模型                                      |
| `infra`  | `Makefile`、`Dockerfile`、`.github/workflows/`、`railway.toml`                    | build、CI/CD、部署                                           |
| `deps`   | 依賴檔與依賴自動化設定                                                            | `pyproject.toml`、`uv.lock`、`.github/dependabot.yml`；Dependabot 產出、手動新增/升級/降級/移除依賴、security patch |

**目錄歸屬補充：**

- `app/utils/` → `agent`（實際為 AI tool helpers，如 `ai_tools.py`）
- `tests/` → 無獨立 scope，跟著被測對象走（測 agent 用 `test(agent):`、測 chat stream 用 `test(chat):`、測 format util 用 `test(agent):`）
- `docs/`（repo 內的 docs 目錄）→ 用 `docs` type，通常無 scope

### 常見歷史 scope 映射

| 舊 scope                                 | 新 scope                                                                                      |
| ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| `ai`, `ai-chat`                          | `chat` 或 `agent` 視內容：若動對話 protocol → `chat`；若動 agent 定義、tool、prompt → `agent` |
| `tools`, `prompt`                        | `agent`                                                                                       |
| `cors`                                   | `core`                                                                                        |
| `build`, `cd`, `ci`, `gitignore`         | `infra`                                                                                       |
| `deps` (historical)                      | `deps`（保留為獨立 scope，不再合併）                                                          |

---

## Cross-Repo Scope Collisions

同名 scope 在不同 repo 可能代表不同程式碼區塊。下表釐清：

| Scope       | prinsur-web                                             | prinsur-api                               | prinsur-ai                                            |
| ----------- | ------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------------- |
| `api`       | 後端 API client（`src/lib/api/`、`src/lib/generated/`） | 不使用（HTTP 層用 `http`）                | FastAPI route 層（`app/api/routes/`、`app/schemas/`） |
| `chat`      | 對話 UI（`components/chat/`、`ai-elements/`）           | 不使用                                    | chat endpoint、streaming、SSE 協議                    |
| `agent`     | 不使用                                                  | agent module（`internal/modules/agent/`） | pydantic-ai agents、tools、prompts                    |
| `insurance` | 保險 UI / 產品頁                                        | 保險 module                               | 不使用（邏輯走 agent tool）                           |
| `auth`      | 登入 UI                                                 | auth module（JWT / session）              | 不使用                                                |
| `admin`     | 不使用                                                  | 後台 module                               | 不使用                                                |
| `infra`     | CI/CD、build、deployment config                         | CI/CD、build、deployment config           | CI/CD、build、deployment config                       |
| `deps`      | `package.json`、`pnpm-lock.yaml`、`dependabot.yml`      | `go.mod`、`go.sum`、`dependabot.yml`      | `pyproject.toml`、`uv.lock`、`dependabot.yml`         |

寫 commit 時不需額外標示 repo — 從當前所在 repo 的上下文自動解讀。

---

## Cross-Cutting Changes

當一個 commit 同時觸碰多個 scope：

1. **主要變更 + 附帶調整** → 使用主要變更的 scope（例如 `feat(agent)` commit 中附帶改一個 `core/` 設定是可接受的）。
2. **跨越過半清單** → 省略 scope：`refactor: rename profile_type -> user_type`。
3. **純依賴類**（只動依賴檔、lock 檔、Dependabot 設定）→ 歸 `deps` scope。
4. **純 infra/CI 類**（涉及多個 module 但本質是同一件事，非依賴變更）→ 歸 `infra` scope 或 `ci` type。

---

## How to Add a New Scope

清單應保持精簡。若你認為需要新增 scope，先確認：

1. 是否已有 scope 可以涵蓋？（90% 的新需求都落在既有清單）
2. 這個新 scope 對應的 code 是否已穩定存在於 repo 中？
3. 未來是否會有 ≥ 5 個 commit 使用到它？

通過以上三問才修改本文件加入新 scope。**新 scope 的加入本身應該是一個 `docs(commit-scopes)` commit。**

---

## Enforcement

scope 的強制檢查由各 repo 的 PR title lint workflow 執行，scope 清單透過 `amannn/action-semantic-pull-request` 的 `scopes:` 輸入傳入。完整設定與政策見 [commit-enforcement.md](./commit-enforcement.md)。
