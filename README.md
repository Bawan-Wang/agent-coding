# agent-coding

三種在 VS Code 中使用 GitHub Copilot Agent 進行 AI 輔助開發的專案架構範本與操作指南。

---

## 目錄結構

```
agent_coding/
├── solo-agent-project/       # 單人 + 單一 Agent
├── multi-agents-project/     # 單人 + 多專責 Agent
└── multi-persons-project/    # 多人 + 多 Agent 協作
```

---

## 三種模式比較

| 模式 | 適用情境 | 核心機制 |
|------|---------|---------|
| **Solo Agent** | 個人專案、快速驗證 | 單一 Agent 搭配 `AGENTS.md` + `PLAN.md` + `context.md` |
| **Multi-Agents** | 單人複雜專案、需分工角色 | Orchestrator 統籌，委派給 architect / backend-dev / frontend-dev / reviewer |
| **Multi-Persons** | 多人團隊、需防模組衝突 | Git + `TEAM.md` 非同步協調，每人帶自己的 agents |

---

## Solo Agent Project

[solo-agent-project/](solo-agent-project/) — 最輕量的起點。

**核心概念：** 每次對話開始時，讓 Agent 讀取固定的啟動文件對齊狀態，任務結束後更新狀態文件。

### 推薦專案結構

```
project/
├── AGENTS.md             # Agent 權限、語氣、可用工具
├── PLAN.md               # 整體進度藍圖
├── README.md
├── .docs/
│   ├── api.md            # API / 函式介面定義
│   ├── architecture.md   # 模組依賴、資料夾結構
│   ├── context.md        # 當前開發焦點、已知 bug
│   ├── rules.md          # Coding style、命名規範
│   ├── exec-plans/       # 詳細執行計畫
│   └── product-specs/    # 功能需求規格
├── src/
└── tests/
```

### 操作流程

```
/startup（AI 讀 AGENTS.md + PLAN.md + context.md）
    ↓ 說今天要做什麼
    ↓ 執行任務
    ↓ 「完成，請更新 PLAN.md 和 context.md」
    ↓ 明天重複
```

詳見 [solo-agent-project/solo-agent-operation.md](solo-agent-project/solo-agent-operation.md)

---

## Multi-Agents Project

[multi-agents-project/](multi-agents-project/) — 單人使用多個專責 Agent。

**核心概念：** 在 `.github/agents/` 定義多個角色，由 Orchestrator 統籌分派。

### Agent 角色分工

| Agent | 職責 | 讀 | 寫 |
|-------|------|----|----|
| `orchestrator` | 理解需求、分派任務、更新進度 | `AGENTS.md`, `PLAN.md` | `PLAN.md`, `.docs/agents/` |
| `architect` | 設計架構、維護執行計畫 | `architecture.md`, `exec-plans/` | `exec-plans/`, `architecture.md` |
| `backend-dev` | 實作後端邏輯 | `api.md`, `rules.md` | `src/backend/` |
| `frontend-dev` | 實作前端介面 | `api.md`, `rules.md` | `src/frontend/` |
| `reviewer` | 審查程式碼 | `src/`（唯讀） | review 報告 |

### 推薦專案結構

```
project/
├── AGENTS.md
├── PLAN.md
├── .github/
│   └── agents/
│       ├── orchestrator.agent.md
│       ├── architect.agent.md
│       ├── backend-dev.agent.md
│       ├── frontend-dev.agent.md
│       └── reviewer.agent.md
├── .docs/
│   ├── api.md / architecture.md / rules.md / tools.md
│   ├── agents/            # 各 Agent 的動態工作日誌
│   ├── exec-plans/        # 詳細執行計畫（architect 維護）
│   └── product-specs/
├── src/
└── tests/
```

詳見 [multi-agents-project/multiple-agents-operation.md](multi-agents-project/multiple-agents-operation.md)

---

## Multi-Persons Project

[multi-persons-project/](multi-persons-project/) — 多人團隊，各自帶 agents 協作。

**核心概念：** Agents 之間不直接通話，透過 Git commit 文件做非同步狀態同步；`TEAM.md` 防止多人同時修改同一模組。

### 額外機制

- **`TEAM.md`**：模組佔用表，任務認領/釋放時更新，防衝突
- **`.docs/developers/`**：每人獨立的工作日誌（`alice.md`, `bob.md` ...）
- **`.github/prompts/`**：`startup.prompt.md`、`claim-task.prompt.md`、`pr-review.prompt.md`

### 每日操作流程

```
git pull main
    ↓ /startup（AI 讀 AGENTS.md + TEAM.md + PLAN.md + 個人日誌）
    ↓ 確認誰在動哪裡，認領任務（更新 TEAM.md）
    ↓ 開 feature branch，用 agents 開發
    ↓ /pr-review → reviewer agent 審查 → 發 PR
    ↓ merge 後釋放 TEAM.md 佔用，更新 PLAN.md
    ↓ git push，明天重複
```

詳見 [multi-persons-project/multi-persons-operation.md](multi-persons-project/multi-persons-operation.md)

---

## 快速選擇指南

```
只有我一個人，任務不複雜？
    → solo-agent-project

只有我一個人，但任務需要架構/開發/審查分工？
    → multi-agents-project

有多位成員，需要防止衝突？
    → multi-persons-project
```

---

## 前置需求

- VS Code（支援 `.github/agents/*.agent.md` 自動掃描）
- GitHub Copilot（Agent 模式）
