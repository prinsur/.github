# Commit Enforcement

本文件定義 Prinsur 三個 repo（prinsur-web、prinsur-api、prinsur-ai）如何強制 commit 規範，以及相關自動化工具的設定政策。

規範定義見：

- [commit-tags.md](./commit-tags.md) — 11 個 commit type
- [commit-scopes.md](./commit-scopes.md) — 各 repo scope 清單

---

## 為什麼是 PR Title Lint，不是 Commit Lint

三個 repo 統一採用 GitHub **Squash Merge** 策略。這有兩個重要意涵：

1. **主幹每個 commit = PR 的 squash commit**，其 message 來自 PR title（body 為 GitHub 自動匯總）。
2. **Feature branch 上的本地 commits 不會進入 main**，因此對本地 commit 做嚴格檢查（commitlint husky hook）ROI 很低，反而常被 contributor 抱怨影響開發節奏。

結論：強制點放在 **PR title**，不放在本地 commit。

參考資料：

- [GitHub: Default to PR titles for squash merge](https://github.blog/changelog/2022-05-11-default-to-pr-titles-for-squash-merge-commit-messages/)
- Satellytes 的 [commit message 與 PR title 強制指南](https://www.satellytes.com/blog/post/writing-and-enforcing-conventional-commit-messages-and-pull-request-titles/)

---

## 工具：action-semantic-pull-request

使用 [`amannn/action-semantic-pull-request@v6`](https://github.com/amannn/action-semantic-pull-request) 在每個 PR 層檢查 title。

### 三個 repo 共用的 Reusable Workflow

本 repo `.github/workflows/pr-title-lint.yml` 提供 `workflow_call` 介面，三個 repo 呼叫此 workflow 即可，不需各自維護完整設定。

**各 repo 的 caller workflow**（放在 `<repo>/.github/workflows/lint-pr-title.yml`）：

```yaml
name: Lint PR Title

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

permissions:
  pull-requests: read

jobs:
  lint:
    uses: prinsur/.github/.github/workflows/pr-title-lint.yml@main
    with:
      scopes: |
        # 替換為當前 repo 的 scope 清單
```

### 各 repo 的 scopes 輸入

**prinsur-web：**

```
app
ui
chat
insurance
auth
landing
i18n
api
common
layouts
infra
```

**prinsur-api：**

```
admin
agent
conversation
customer
insurance
auth
storage
verification
db
http
infra
```

**prinsur-ai：**

```
chat
agent
api
core
models
infra
```

### 檢查內容

- `type` ∈ 11 個允許的 type（固定於 reusable workflow，無需 caller 傳入）
- `scope` ∈ caller 傳入的 scope 清單（可為空，因為規範允許 scope 省略）
- `subject` 不得以大寫字母開頭（`^(?![A-Z]).+$`）

### 觸發條件說明

使用 `pull_request`（而非 `pull_request_target`）的理由：

- prinsur 三個 repo 皆為 organization 內部 private repo，不預期接收 fork PR，`pull_request_target` 提供的 fork 支援價值低。
- `pull_request` 會從 PR 的 HEAD（feature branch）讀取 workflow 定義，**首次加入本 workflow 時可自我驗證**；若改用 `pull_request_target`，GitHub 會從 base branch（main）讀取 workflow，首次 PR 因檔案尚未在 main 而無法觸發（chicken-and-egg）。
- 本 workflow 只讀取 PR metadata，不 checkout 原始碼、不需要 secret，`pull_request` 的 read-only `GITHUB_TOKEN` 已足夠。

若未來需接收 fork PR，可依 [`amannn/action-semantic-pull-request`](https://github.com/amannn/action-semantic-pull-request) 文件改用 `pull_request_target`。

---

## Dependabot 前綴政策

Dependabot 是 commit 來源之一。為避免 changelog 混亂，三個 repo 採統一前綴：

| 依賴類型                      | 前綴           | 原因                                     |
| ----------------------------- | -------------- | ---------------------------------------- |
| 語言層依賴（pnpm、uv、gomod） | `chore(deps):` | 與歷史 commit 一致，changelog 不特別突出 |
| GitHub Actions workflow 依賴  | `ci(deps):`    | 本質上是 CI 設定變更                     |

### 各 repo 當前設定檢核表

在 `<repo>/.github/dependabot.yml` 確認：

```yaml
# Dependabot 使用 npm / pip / gomod 作為 ecosystem 值，
# 會自動偵測專案實際使用的 pnpm / uv / poetry 等 sub-tool
- package-ecosystem: "npm" # prinsur-web (pnpm)
  commit-message:
    prefix: "chore(deps)"
- package-ecosystem: "pip" # prinsur-ai (uv)
  commit-message:
    prefix: "chore(deps)"
- package-ecosystem: "gomod" # prinsur-api
  commit-message:
    prefix: "chore(deps)"

# GitHub Actions（若有）
- package-ecosystem: "github-actions"
  commit-message:
    prefix: "ci(deps)"
```

### 手動升級依賴的建議

手動新增或升級 **runtime** 依賴時，建議使用 `build(deps):`（更精確表達該變更會影響產出的 artifact）。但若只是小量升級、與歷史一致優先，使用 `chore(deps):` 亦可。開發依賴（dev-only）統一用 `chore(deps):`。

---

## Renovate（若未來改用）

目前三個 repo 都用 Dependabot。若未來改用 Renovate，需在 `renovate.json` 明確指定 `semanticCommitType` 與 `semanticCommitScope`，與上述政策一致：

```json
{
  "semanticCommits": "enabled",
  "semanticCommitType": "chore",
  "semanticCommitScope": "deps"
}
```

---

## 為什麼不導入本地 commitlint（husky hook）

考慮過但刻意不做。原因：

1. **Squash Merge 下，本地 commit 不上 main**。強制本地 commit 格式價值低。
2. **社群回報本地 hook 常被抱怨**。開發中常有 WIP、fixup、實驗性 commit，被 hook 擋下反而阻礙迭代。
3. **PR title lint 已覆蓋主要價值**。主幹 commit 一致性 + 未來 changelog 基礎皆已滿足。

若團隊成長到需要嚴格化（例如導入 semantic-release 自動發版、或需要本地 commit 做 fine-grained changelog），再評估加入 commitlint CI check（非 husky）。

---

## 未來擴充方向

| 時機                 | 考慮導入                                                                    |
| -------------------- | --------------------------------------------------------------------------- |
| 需要自動 changelog   | `conventional-changelog` + release-please / semantic-release                |
| 需要自動 semver bump | `semantic-release`（僅 node 生態友善，Go/Python 需替代品）或 release-please |
| 需要更嚴格檢查       | `commitlint` CI check（不用 husky 本地 hook）                               |
| 新 repo 加入 org     | 直接套用本 repo 的 reusable workflow，無額外設定                            |

---

## 失敗排查

### PR title lint 失敗常見原因

1. **Type 拼錯** — 例如 `refractor`、`feature`（應為 `refactor`、`feat`）
2. **Scope 不在清單** — 例如 prinsur-api 用了 `middleware`（應為 `http`）
3. **Subject 以大寫開頭** — 例如 `feat(ui): Add dark mode`（應為 `add dark mode`）
4. **缺少冒號空格** — 例如 `feat(ui):add dark mode`（冒號後需空格）

修正 PR title 後 check 會自動重跑，無需重推 commit。

### Reusable workflow 被修改後三個 repo 立即生效

本 repo `@main` 為 caller 參考的版本。若需凍結特定版本，caller 改用 commit SHA 或 tag（例如 `@v1.0.0`）。目前保持 `@main` 以利統一演進。

---

## 參考資料

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- [@commitlint/config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/@commitlint/config-conventional)
- [GitHub: Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [GitHub: Creating a default community health file](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
