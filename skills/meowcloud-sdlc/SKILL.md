---
name: meowcloud-sdlc
description: MeowCloud 完整軟體開發生命週期（SDLC）流程。當需要啟動新專案、執行開發迭代、建立 PR/Code Review 流程、或管理從需求到部署的完整開發週期時使用。涵蓋：(1) 需求收集與 Issue 建立 (2) 系統設計與文件產出 (3) GitHub Flow 開發（Branch → PR → Review → Merge）(4) 自動化 CI/CD 與 Preview 部署 (5) 驗收與正式部署。適用於 MeowCloud-ai GitHub Organization 下所有專案。
---

# MeowCloud SDLC

MeowCloud 的標準軟體開發流程。所有專案遵循相同的五階段流程。

## 角色分工

| 角色 | 執行者 | 職責 |
|------|--------|------|
| **Orchestrator** | 阿福 (OpenClaw) | 需求分析、系統設計、任務拆解、品質把關、進度回報 |
| **Coding Agent** | Claude Code Sub-agent | 根據 TASKS.md 逐任務開發，Branch 開發 |
| **Review Agent** | Claude Code Sub-agent | 在 GitHub PR 留 Comment，審碼品質/安全/效能 |
| **驗收人** | 軒哥 | 透過 Preview URL 實際操作驗證 |

**核心原則：Orchestrator 負責「想」，Claude Code 負責「做」。**

## 倉庫架構

- **MeowCloud-ai/meowcloud** — 公司層級（流程、模板、共用設定）
- **MeowCloud-ai/<project-name>** — 每個專案獨立 repo

## 五階段流程

### Phase 0: 需求收集

1. 軒哥在 Discord 或對話中提出需求
2. Orchestrator 釐清需求，產出標準化 GitHub Issue
3. Issue 格式使用 `assets/issue-templates/feature.md`

**產出物：** GitHub Issues（帶 User Story + Acceptance Criteria + Priority）

### Phase 1: 系統設計

1. 根據 Issues 分析需求
2. 產出設計文件（使用 `assets/templates/` 中的模板）：
   - `CLAUDE.md` — 開發指引（Claude Code 自動讀取）
   - `DECISIONS.md` — 架構決策紀錄
   - `docs/PRD.md` — 產品需求規格
   - `docs/ARCHITECTURE.md` — 技術架構
   - `docs/TASKS.md` — 任務拆解（每個 Task 對應一個 Issue）
3. 建立 `.claude/agents/` Agent 定義
4. 軒哥確認後進入開發

**產出物：** 設計文件 + Agent 定義，推送到 repo main branch

### Phase 2: 開發迭代（GitHub Flow）

每個 Task 執行以下流程：

1. **建立 Feature Branch**: `feat/task-N-description`
2. **Coding Agent 開發**: 根據 CLAUDE.md + TASKS.md 寫碼
3. **本地檢查**: Hooks 自動觸發 Lint + Type Check + Unit Test
4. **推送 + 建立 PR**:
   - PR 描述使用 `assets/github/pr-template.md`
   - 關聯 Issue: `Closes #N`
5. **CI Pipeline**: GitHub Actions 自動跑 Build + Test + Lint
6. **Preview 部署**: PR 觸發自動部署 Preview 環境
7. **Review Agent 審碼**: 在 PR 留結構化 Comment
   - ✅ Approve → 進入 Merge
   - ❌ Request Changes → Coding Agent 修改 → 重新 Push
8. **Merge to main**

**詳細 CI 設定**: 見 `references/ci-setup.md`
**Review 標準**: 見 `references/review-standards.md`

### Phase 3: 驗收部署

1. PR 建立時自動產生 Preview URL
2. 通知軒哥驗收（提供 Preview URL + 測試要點）
3. 軒哥實際操作驗證
4. Approve PR → Merge to main → 自動部署 Production

### Phase 4: 維運監控

1. Production 部署後監控錯誤
2. 定期健康檢查
3. Bug → 回到 Phase 0 建 Issue

## 初始化新專案

當啟動新專案時，執行 `scripts/init-project.sh`：

```bash
bash scripts/init-project.sh <project-name>
```

此腳本會：
1. 在 MeowCloud-ai org 建立新 repo
2. 從 `assets/templates/` 複製所有模板檔案
3. 從 `assets/github/` 複製 CI + PR/Issue templates
4. 從 `assets/agents/` 複製 Agent 定義
5. 推送初始 commit

初始化後，進入 Phase 0 開始需求收集。

## 專案目錄結構

每個專案初始化後的結構：

```
project-name/
├── CLAUDE.md                  # 開發指引
├── DECISIONS.md               # 架構決策
├── README.md
├── docs/
│   ├── PRD.md                 # 需求規格
│   ├── ARCHITECTURE.md        # 技術架構
│   └── TASKS.md               # 任務拆解
├── .claude/
│   └── agents/
│       ├── coder.md           # Coding Agent
│       └── reviewer.md        # Review Agent
├── .github/
│   ├── workflows/
│   │   └── ci.yml             # CI Pipeline
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│       ├── feature.md
│       └── bug.md
└── src/
```
