有幾種操作方式，從簡單到進階：

---

## 方式一：手動選擇 Agent（最簡單）

在 VS Code Chat 的 Agent Picker 選擇對應 agent，然後直接下指令：

```
[選 orchestrator] 我要實作 Issue #42，幫我規劃並分派任務
[選 backend-dev]  依照 exec-plans/002 實作 stream_recorder.py
[選 reviewer]     審查剛才修改的 src/backend/ 下的檔案
```

每次你自己決定找哪個 agent，適合**小任務或明確知道要做什麼**的時候。

---

## 方式二：全部交給 Orchestrator（推薦）

你只跟 orchestrator 說需求，它自己決定叫誰做：

```
[選 orchestrator] 幫我完成 Issue #42：串流斷線自動重連
```

Orchestrator 內部流程：
```
1. 讀 PLAN.md → 確認優先順序
2. 讀 .docs/agents/orchestrator.md → 上次做到哪
3. 判斷需要 architect + backend-dev
4. 呼叫 architect → 更新 exec-plans/
5. 呼叫 backend-dev → 修改 src/
6. 更新 PLAN.md 和 .docs/agents/orchestrator.md
```

但這需要 `orchestrator.agent.md` 裡有明確的委派指令。

---

## 方式三：每日啟動流程（固定儀式）

建立一個固定的開始方式，讓 orchestrator 自動對齊狀態：

```
[選 orchestrator] 早安，請讀 PLAN.md 和各 agent 的 context，
                  告訴我現在進度，然後問我今天要做什麼
```

Orchestrator 會回：
```
目前進度：
- ✅ Docker Compose 基礎架構
- 🔄 Issue #42 進行中（backend-dev 做到 stream_recorder.py）
- ⏳ Issue #45 待開始

今天要繼續 #42 還是開始 #45？
```

---

## 實際操作建議

**現階段最務實的做法：**

```
第一步（你）：   選 orchestrator，輸入任務
第二步（AI）：   orchestrator 規劃，你確認
第三步（AI）：   orchestrator 呼叫 subagent 執行
第四步（AI）：   執行完，更新 .docs/agents/ 的 context
第五步（你）：   檢查結果，若有問題直接對 subagent 說
```

---

## 關鍵：Orchestrator 要能「記住」狀態

`orchestrator.agent.md` 裡要加這條指令：

```markdown
## 每次任務結束
更新 .docs/agents/orchestrator.md，記錄：
- 完成了什麼
- 分派給哪個 agent
- 下次要繼續的事項
```

這樣下次開新對話，orchestrator 讀這份日誌就能接續，不會從頭來過。

## 如何建立 Orchestrator

你需要先在 `.github/agents/` 建立對應的 `.agent.md` 檔案，VS Code 才會自動掃描並顯示在 picker 裡。

在 VS Code Copilot Chat 的 **Agent Picker** 裡選擇，就是聊天框上方的模式選單：

```
Chat 輸入框上方：
┌─────────────────────────────────┐
│  🤖 orchestrator          ▼     │  ← 點這裡選
├─────────────────────────────────┤
│  Ask                            │
│  Edit                           │
│  Agent                          │
│  ─────────────                  │
│  orchestrator      ← 你建的      │
│  backend-dev       ← 你建的      │
│  docker-compose-advisor          │
└─────────────────────────────────┘
```

---

**如果目前 repo 還沒有這些 agent 檔案**， picker 就不會出現。