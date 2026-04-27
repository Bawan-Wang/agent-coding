多人多 Agent 的核心是：**每位成員各帶自己的 agents，透過 Git + TEAM.md 做非同步協調**。
Agents 之間不直接通話，透過 committed 文件同步狀態。

---

## 方式一：手動選擇 Agent（最簡單）

在 VS Code Chat 的 Agent Picker 選擇對應 agent，直接下指令：

```
[選 orchestrator] 我是 Bob，要實作 Issue #42，幫我確認有沒有衝突再規劃
[選 backend-dev]  依照 exec-plans/002 實作 stream_recorder.py
[選 reviewer]     審查我這個 PR branch 的 src/backend/ 改動
```

適合**明確知道要做什麼**，或**單人任務不需要協調**的時候。

---

## 方式二：全部交給 Orchestrator（推薦）

你只跟 orchestrator 說需求，它自己確認衝突、規劃、委派：

```
[選 orchestrator] 我是 Bob，要做 Issue #42：串流斷線自動重連
```

Orchestrator 內部流程：
```
1. 讀 AGENTS.md → 確認全域規則
2. 讀 TEAM.md   → 確認 Issue #42 沒有人在做、src/backend/stream.py 沒被佔用
3. 讀 PLAN.md   → 確認優先順序
4. 讀 .docs/developers/bob.md → 上次做到哪
5. 更新 TEAM.md → 登記 Bob 認領 #42
6. 呼叫 architect → 建立 exec-plans/002-stream-fix.md
7. 呼叫 backend-dev → 修改 src/
8. 更新 .docs/developers/bob.md
```

---

## 方式三：每日啟動流程（固定儀式）

每人開始工作時用個人化啟動 prompt：

```
[選 orchestrator] /startup
（startup.prompt.md 會帶入你的名字，自動讀入個人 context）
```

Orchestrator 會回：
```
Hi Bob，目前狀況：
- Alice 正在動 .docs/architecture.md（請勿修改）
- Charlie 正在做 src/frontend/auth/（無衝突）
- 你上次做到 stream_recorder.py 的第 3 步，還差錯誤處理

今天繼續 #42 還是有其他任務？
```

---

## 每日操作流程

```
開始工作
    ↓
git pull main
    ↓
/startup（AI 讀 AGENTS.md + TEAM.md + PLAN.md + .docs/developers/[你的名字].md）
    ↓
AI 告訴你：誰在動哪裡、你上次做到哪
    ↓
開 feature branch：feature/bob/issue-42
    ↓
認領任務（orchestrator 更新 TEAM.md，commit + push）
    ↓
用 orchestrator / specialist agent 開發
    ↓
任務完成，AI 更新 .docs/developers/[你的名字].md
    ↓
/pr-review（reviewer agent 審查，輸出報告）
    ↓
人工確認 → 發 PR → merge main
    ↓
TEAM.md 釋放佔用，PLAN.md 更新進度（orchestrator 處理）
    ↓
git push，明天重複
```

---

## Prompt 檔案設定

### startup.prompt.md
```markdown
---
description: "個人化啟動 session，對齊專案狀態"
---
我是 {{DEVELOPER_NAME}}。請依序讀：
1. AGENTS.md（全域規則）
2. TEAM.md（目前誰在做什麼，有沒有衝突風險）
3. PLAN.md（整體優先順序）
4. .docs/developers/{{DEVELOPER_NAME}}.md（我上次做到哪）

讀完後告訴我：
- 哪些模組目前有人佔用（我要避開）
- 我自己上次做到哪
- 當前最高優先的未認領任務

然後等我說今天要做什麼。
```

### claim-task.prompt.md
```markdown
---
description: "認領任務，更新 TEAM.md 防衝突"
---
我是 {{DEVELOPER_NAME}}，要認領 {{ISSUE_ID}}。
請：
1. 讀 TEAM.md，確認要觸碰的模組沒有人在用
2. 若有衝突，告訴我衝突的人是誰、碰的是哪個檔案
3. 若無衝突，在 TEAM.md 的認領表新增一列，然後 commit
```

### pr-review.prompt.md
```markdown
---
description: "PR 前的 AI 標準化審查"
---
[選 reviewer]
請審查目前 branch 相對於 main 的所有改動。
對照 .docs/rules.md 的 coding style 與 .docs/api.md 的介面規格，
輸出：
- 潛在問題列表（嚴重度：high / medium / low）
- 符合規範的部分
- 建議修改的具體行號與說明
```

---

## 關鍵：狀態記憶設定

各個 `.github/agents/*.agent.md` 裡都需要加入：

```markdown
## 每次任務結束
更新 .docs/developers/{{DEVELOPER_NAME}}.md，記錄：
- 完成了什麼
- 目前 focus 的檔案與行號
- 已知但暫未修的 bug
- 下次要繼續的步驟
```

---

## 與其他模式的操作差異

| | Solo Single Agent | Solo Multi-Agent | Multi-Persons Multi-Agent |
|---|---|---|---|
| 啟動咒語 | 固定 | 固定 | 個人化（帶入 {{DEVELOPER_NAME}}） |
| 任務開始前 | 直接做 | 直接做 | 先讀 TEAM.md 確認衝突 |
| 狀態記錄 | `context.md` | `.docs/agents/*.md` | `.docs/developers/[name].md` |
| 協調機制 | 無需 | Orchestrator session 內協調 | TEAM.md + Git 跨 session 協調 |
| Review | 無 | reviewer agent | PR + reviewer agent |
| branch 策略 | 無需 | 無需 | `feature/[name]/[issue]` |

---

## 共享文件的守門人規則

| 文件 | 誰能改 | 如何改 |
|------|--------|--------|
| `AGENTS.md` | Tech Lead | 直接 commit main |
| `.docs/api.md` | Tech Lead approve | PR → review → merge |
| `.docs/architecture.md` | Tech Lead approve | PR → review → merge |
| `.docs/rules.md` | Tech Lead approve | PR → review → merge |
| `PLAN.md` | Tech Lead + orchestrator | 任務完成後更新 |
| `TEAM.md` | 各成員的 orchestrator | 認領/釋放時更新 |
| `.docs/developers/[name].md` | 只有本人的 agent | 每次任務後更新 |
| `exec-plans/[plan].md` | 計畫的 owner | 任務進行中更新 |
