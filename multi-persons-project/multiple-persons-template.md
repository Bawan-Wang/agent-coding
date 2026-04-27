```
project_template/
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ 啟動時讀取（全域規則）                                      │
├── AGENTS.md          # 所有 agent 共用：tone、工具權限、專案慣例
│  └─────────────────────────────────────────────────────────┘
│
├── PLAN.md            # 整體進度藍圖（Tech Lead 維護）
├── TEAM.md            # 成員名單、角色分工、當前模組佔用表
├── README.md          # 給人類開發者的專案說明
├── requirements.txt   # Python 依賴管理
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ .github/agents/  → VS Code 自動掃描，定義 Agent 身份      │
│  └─────────────────────────────────────────────────────────┘
├── .github/
│   ├── agents/
│   │   ├── orchestrator.agent.md   # 主 agent：讀 PLAN.md，分派任務給 subagent
│   │   ├── architect.agent.md      # 架構師：讀寫 architecture.md、exec-plans/
│   │   ├── backend-dev.agent.md    # 後端工程師：讀 api.md rules.md，寫 src/
│   │   ├── frontend-dev.agent.md   # 前端工程師：讀 api.md，寫 src/frontend/
│   │   └── reviewer.agent.md       # 審查員：唯讀 src/，輸出 review 報告
│   │
│   ├── prompts/
│   │   ├── startup.prompt.md       # 個人化啟動（帶入成員姓名，讀個人 context）
│   │   ├── claim-task.prompt.md    # 認領任務前更新 TEAM.md
│   │   └── pr-review.prompt.md     # PR 標準化 AI 審查
│   │
│   └── PULL_REQUEST_TEMPLATE.md    # PR 模板（含 AI review checklist）
│
│  ┌─────────────────────────────────────────────────────────┐
│  │ .docs/  → 靜態知識庫（agent 讀取的參考文件）                │
│  └─────────────────────────────────────────────────────────┘
├── .docs/
│   ├── api.md              # API / 函式介面定義（Tech Lead approve 才能改）
│   ├── architecture.md     # 模組依賴、資料夾結構設計決策
│   ├── rules.md            # Coding style、命名規範、禁用套件
│   ├── tools.md            # 可用的外部工具、CLI 指令
│   │
│   │  ┌──────────────────────────────────────────────────────┐
│   │  │ developers/  → 動態狀態（每位成員的工作日誌）            │
│   │  └──────────────────────────────────────────────────────┘
│   ├── developers/
│   │   ├── alice.md        # Alice 的當前任務、做到哪、暫存 bug
│   │   ├── bob.md          # Bob 的當前任務、做到哪、暫存 bug
│   │   └── charlie.md      # Charlie 的當前任務、做到哪、暫存 bug
│   │
│   │  ┌──────────────────────────────────────────────────────┐
│   │  │ exec-plans/  → 詳細執行計畫（每份計畫有 owner + reviewer）│
│   │  └──────────────────────────────────────────────────────┘
│   ├── exec-plans/
│   │   ├── 001-setup-env.md        # owner: alice  | reviewer: bob
│   │   ├── 002-stream-fix.md       # owner: bob    | reviewer: charlie
│   │   └── done/                   # 已完成的計畫封存
│   │
│   └── product-specs/              # 功能規格（實作前確認邏輯）
│       ├── index.md
│       └── new-user-onboarding.md
│
├── src/                            # 原始碼
└── tests/                          # 測試腳本
```

---

## 各層職責總覽

```
┌──────────────────────────────────────────────────────────────┐
│              成員 A / B / C（各自在自己的 branch）              │
└──────┬──────────────┬──────────────┬──────────────┬──────────┘
       ↓              ↓              ↓              ↓
  orchestrator    architect      backend-dev    frontend-dev
  讀：             讀：             讀：            讀：
  PLAN.md          architecture.md  api.md         api.md
  TEAM.md          exec-plans/      rules.md       rules.md
  寫：             寫：             寫：            寫：
  PLAN.md          exec-plans/      src/backend/   src/frontend/
  TEAM.md          architecture.md  developers/    developers/

                              ↓
                         reviewer
                         讀：src/（PR branch）
                         輸出：review 報告
```

---

## 三類檔案的本質差異

| 類型 | 位置 | 性質 | 誰寫 | 改變頻率 |
|---|---|---|---|---|
| **Agent 定義** | `.github/agents/*.agent.md` | 靜態身份（角色、工具、限制） | Tech Lead（一次設定） | 很少 |
| **知識庫** | `.docs/*.md`、`exec-plans/`、`product-specs/` | 靜態參考文件 | Tech Lead + architect | 任務開始前更新 |
| **個人工作日誌** | `.docs/developers/*.md` | 動態狀態（做到哪、注意事項） | 各成員的 agent 自動維護 | 每次任務後更新 |
| **協調中樞** | `TEAM.md` | 跨人防衝突（誰占哪個模組） | 各成員認領/釋放時更新 | 任務開始與結束時 |

---

## TEAM.md 範本

```markdown
# TEAM.md

## 成員與角色
| 成員    | 主責角色            | Git Handle  |
|---------|-------------------|-------------|
| Alice   | Tech Lead / Architect | @alice  |
| Bob     | Backend Dev       | @bob        |
| Charlie | Frontend Dev      | @charlie    |

## 當前任務認領（防衝突表）
| Issue / 任務         | 負責人   | 觸碰的模組 / 檔案        | 狀態     |
|---------------------|---------|------------------------|---------|
| #42 串流斷線重連      | Bob     | src/backend/stream.py  | 進行中   |
| #45 登入頁 redesign  | Charlie | src/frontend/auth/     | 進行中   |
| architecture 重構    | Alice   | .docs/architecture.md  | Review 中|

## 共享資源鎖定規則
- api.md / architecture.md：修改前必須在此表登記 + 通知 Alice
- 任何人 merge 到 main 前，reviewer agent 必須跑完才能發 PR
- exec-plans/ 下的計畫由 owner 負責維護，完成後移入 done/
```

---

## exec-plans 計畫檔頭部範本

```markdown
## Metadata
- **Owner**: Bob
- **Reviewer**: Alice
- **Branch**: feature/bob/issue-42
- **Touches**: src/backend/stream.py, tests/test_stream.py
- **Blocks**: Issue #47（依賴此模組，未完成前勿動）

## 執行步驟
...
```
