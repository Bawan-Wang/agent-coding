```
project_template/
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ 啟動時讀取（全域規則）                                      │
├── AGENTS.md          # 所有 agent 共用：tone、工具權限、專案慣例
│  └─────────────────────────────────────────────────────────┘
│
├── PLAN.md            # 整體進度藍圖（orchestrator 讀寫）
├── README.md          # 給人類開發者的專案說明
├── requirements.txt   # Python 依賴管理
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ .github/agents/  → VS Code 自動掃描，定義 Agent 身份      │
│  └─────────────────────────────────────────────────────────┘
├── .github/
│   └── agents/
│       ├── orchestrator.agent.md   # 主 agent：讀 PLAN.md，分派任務給 subagent
│       ├── architect.agent.md      # 架構師：讀寫 architecture.md、exec-plans/
│       ├── backend-dev.agent.md    # 後端工程師：讀 api.md rules.md，寫 src/
│       ├── frontend-dev.agent.md   # 前端工程師：讀 api.md，寫 src/frontend/
│       └── reviewer.agent.md       # 審查員：唯讀 src/，輸出 review 報告
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ .docs/  → 靜態知識庫（agent 讀取的參考文件）                │
│  └─────────────────────────────────────────────────────────┘
├── .docs/
│   ├── api.md              # API / 函式介面定義
│   ├── architecture.md     # 模組依賴、資料夾結構設計決策
│   ├── rules.md            # Coding style、命名規範、禁用套件
│   ├── tools.md            # 可用的外部工具、CLI 指令（原 skill.md）
│   │
│   │  ┌──────────────────────────────────────────────────────┐
│   │  │ agents/  → 動態狀態（agent 的工作日誌，隨任務更新）      │
│   │  └──────────────────────────────────────────────────────┘
│   ├── agents/
│   │   ├── orchestrator.md    # 目前任務佇列、分派狀態
│   │   ├── architect.md       # 架構待決事項、已做的 trade-off
│   │   ├── backend-dev.md     # 目前 focus、做到哪、暫時不動的 bug
│   │   └── frontend-dev.md    # 目前 focus、UI 狀態
│   │
│   │  ┌──────────────────────────────────────────────────────┐
│   │  │ exec-plans/  → 詳細執行計畫（architect 負責維護）        │
│   │  └──────────────────────────────────────────────────────┘
│   ├── exec-plans/
│   │   ├── 001-setup-env.md
│   │   ├── 002-docker-compose.md
│   │   └── done/              # 已完成的計畫封存
│   │
│   └── product-specs/         # 功能規格（實作前確認邏輯）
│       ├── index.md
│       └── new-user-onboarding.md
│
├── src/                       # 原始碼
└── tests/                     # 測試腳本
```
---
各層職責總覽
```
┌──────────────────────────────────────────────────────────────┐
│                        使用者輸入需求                           │
└──────────────────────┬───────────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────────┐
│  orchestrator.agent.md                                        │
│  讀：AGENTS.md、PLAN.md、.docs/agents/orchestrator.md          │
│  職責：理解需求 → 查 PLAN.md → delegate → 更新進度              │
└──────┬──────────────┬──────────────┬──────────────┬──────────┘
       ↓              ↓              ↓              ↓
  architect       backend-dev    frontend-dev    reviewer
  讀：              讀：            讀：            讀：
  architecture.md  api.md         api.md          src/
  exec-plans/      rules.md       rules.md
  寫：              寫：            寫：            輸出：
  exec-plans/      src/backend/   src/frontend/   review 報告
  architecture.md  .docs/agents/  .docs/agents/
```
三類檔案的本質差異

```
project_template/
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ 啟動時讀取（全域規則）                                      │
├── AGENTS.md          # 所有 agent 共用：tone、工具權限、專案慣例
│  └─────────────────────────────────────────────────────────┘
│
├── PLAN.md            # 整體進度藍圖（orchestrator 讀寫）
├── README.md          # 給人類開發者的專案說明
├── requirements.txt   # Python 依賴管理
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ .github/agents/  → VS Code 自動掃描，定義 Agent 身份      │
│  └─────────────────────────────────────────────────────────┘
├── .github/
│   └── agents/
│       ├── orchestrator.agent.md   # 主 agent：讀 PLAN.md，分派任務給 subagent
│       ├── architect.agent.md      # 架構師：讀寫 architecture.md、exec-plans/
│       ├── backend-dev.agent.md    # 後端工程師：讀 api.md rules.md，寫 src/
│       ├── frontend-dev.agent.md   # 前端工程師：讀 api.md，寫 src/frontend/
│       └── reviewer.agent.md       # 審查員：唯讀 src/，輸出 review 報告
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ .docs/  → 靜態知識庫（agent 讀取的參考文件）                │
│  └─────────────────────────────────────────────────────────┘
├── .docs/
│   ├── api.md              # API / 函式介面定義
│   ├── architecture.md     # 模組依賴、資料夾結構設計決策
│   ├── rules.md            # Coding style、命名規範、禁用套件
│   ├── tools.md            # 可用的外部工具、CLI 指令（原 skill.md）
│   │
│   │  ┌──────────────────────────────────────────────────────┐
│   │  │ agents/  → 動態狀態（agent 的工作日誌，隨任務更新）      │
│   │  └──────────────────────────────────────────────────────┘
│   ├── agents/
│   │   ├── orchestrator.md    # 目前任務佇列、分派狀態
│   │   ├── architect.md       # 架構待決事項、已做的 trade-off
│   │   ├── backend-dev.md     # 目前 focus、做到哪、暫時不動的 bug
│   │   └── frontend-dev.md    # 目前 focus、UI 狀態
│   │
│   │  ┌──────────────────────────────────────────────────────┐
│   │  │ exec-plans/  → 詳細執行計畫（architect 負責維護）        │
│   │  └──────────────────────────────────────────────────────┘
│   ├── exec-plans/
│   │   ├── 001-setup-env.md
│   │   ├── 002-docker-compose.md
│   │   └── done/              # 已完成的計畫封存
│   │
│   └── product-specs/         # 功能規格（實作前確認邏輯）
│       ├── index.md
│       └── new-user-onboarding.md
│
├── src/                       # 原始碼
└── tests/                     # 測試腳本
```

---

## 各層職責總覽

```
┌──────────────────────────────────────────────────────────────┐
│                        使用者輸入需求                           │
└──────────────────────┬───────────────────────────────────────┘
                       ↓
┌──────────────────────────────────────────────────────────────┐
│  orchestrator.agent.md                                        │
│  讀：AGENTS.md、PLAN.md、.docs/agents/orchestrator.md          │
│  職責：理解需求 → 查 PLAN.md → delegate → 更新進度              │
└──────┬──────────────┬──────────────┬──────────────┬──────────┘
       ↓              ↓              ↓              ↓
  architect       backend-dev    frontend-dev    reviewer
  讀：              讀：            讀：            讀：
  architecture.md  api.md         api.md          src/
  exec-plans/      rules.md       rules.md
  寫：              寫：            寫：            輸出：
  exec-plans/      src/backend/   src/frontend/   review 報告
  architecture.md  .docs/agents/  .docs/agents/
```

---

## 三類檔案的本質差異
| 類型 | 位置 | 性質 | 誰寫 | 改變頻率 |
|---|---|---|---|---|
| **Agent 定義** | `.github/agents/*.agent.md` | 靜態身份（角色、工具、限制） | 你（一次設定） | 很少 |
| **知識庫** | `.docs/*.md`、`exec-plans/`、`product-specs/` | 靜態參考文件 | 你 + architect | 任務開始前更新 |
| **工作日誌** | `.docs/agents/*.md` | 動態狀態（做到哪、注意事項） | Agent 自動維護 | 每次任務後更新 |
