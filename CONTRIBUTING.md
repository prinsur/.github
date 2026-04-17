# Contributing to Prinsur

本文件為 prinsur organization 下所有 repo 的共用 contributing guide，透過 GitHub organization-level `.github` repo 機制自動套用。若個別 repo 有自己的 `CONTRIBUTING.md`，以該 repo 版本為準。

## Repos

| Repo                                                  | Tech                  | Purpose                 |
| ----------------------------------------------------- | --------------------- | ----------------------- |
| [prinsur-web](https://github.com/prinsur/prinsur-web) | Next.js 16 + React 19 | Frontend                |
| [prinsur-api](https://github.com/prinsur/prinsur-api) | Go + Gin + GORM       | Backend API             |
| [prinsur-ai](https://github.com/prinsur/prinsur-ai)   | Python + FastAPI      | AI Orchestrator Service |

## Branching

- 從 `main` 分出 feature branch
- Branch 命名：`<type>/<short-description>`，type 對齊 commit type（`feat/`、`fix/`、`docs/`、`refactor/`、`chore/` 等）
- Branch 名稱由 Linear issue 產生

## Pull Requests

- 策略：**Squash Merge only**（三個 repo 統一）
- **PR title 必須符合 Conventional Commits 規範**：主幹每個 commit = PR 的 squash commit，message 來自 PR title
- PR title 由 `amannn/action-semantic-pull-request` 在 CI 檢查，失敗則 merge 被阻擋

## Commit Conventions

三份文件組成完整規範：

1. **[docs/commit-tags.md](./docs/commit-tags.md)** — 11 個允許的 commit type，何時使用、UI/UX 變更分類、multi-type 優先序
2. **[docs/commit-scopes.md](./docs/commit-scopes.md)** — 各 repo 的 scope 清單（code topology）
3. **[docs/commit-enforcement.md](./docs/commit-enforcement.md)** — 強制機制、自動化、reusable workflow、Dependabot 政策

### Format 速查

```
<type>(<scope>): <subject>

<body, 中文說明為什麼>

<footer, Git trailers>
```

- `<type>`：必填，11 個之一（feat/fix/docs/style/refactor/perf/test/build/ci/chore/revert）
- `<scope>`：建議必填，各 repo 有自己的清單
- `<subject>`：祈使句、小寫開頭、不加句號、英文
- `<body>`：（選填）中文說明動機與影響
- `<footer>`：（選填）Git trailers 格式（`Closes #N`、`Refs: LINEAR-ID`、`BREAKING CHANGE: ...`）

### 關鍵規則

- 一個 commit 只能有一個 type
- Type 後加 `!` 或 footer `BREAKING CHANGE:` 表示破壞性變更
- **禁止在 commit message 加入 AI 署名**（`Co-Authored-By: Claude` 等）
- Issue 在 Linear，PR 在 GitHub；Linear issue 用 `Refs:`，GitHub issue/PR 用 `Closes:` / `Fixes:`

## Issue Tracking

Issues 統一在 Linear 管理，GitHub Issues 不使用。若需回報 bug 或提 feature request，請到對應的 Linear 專案。
