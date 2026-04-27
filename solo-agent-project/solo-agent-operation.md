核心是**每次對話開始先對齊知識庫，確認沒問題再動 code，commit 前強制同步狀態**。

---

## 操作流程

### 第一步：選 Agent 模式
VS Code Chat 選 **Agent**（不是 Ask）

---

### 第二步：啟動並對齊狀態

每次開新對話，貼這段（或用 `/startup`，見下方）：

```
請先讀：
- AGENTS.md（你的規則和權限）
- PLAN.md（整體進度）
- .docs/context.md（當前狀態）

讀完後告訴我現在的進度，然後等我說今天要做什麼。
```

---

### 第三步：確認知識庫是否需要更新

**在下任務之前**，請 Agent 先核對相關文件：

```
請先確認 .docs/product-specs/、architecture.md、.docs/api.md
是否與今天要做的任務一致，有落差先告訴我。
```

若有落差，**先更新 `.docs/` 對應檔案**，再繼續。

---

### 第四步：寫 exec-plan 並審核

Agent 確認知識庫後，產生執行計畫：

```
請依據剛才確認的內容，建立 exec-plans/XXX.md
```

exec-plan 結構建議**拆成兩段**：

```markdown
## 文件變更（若有）
- 更新 .docs/architecture.md：新增 XXX 模組
- 更新 .docs/api.md：新增 /stream/reconnect endpoint

## 實作步驟
1. ...
2. ...
```

**你審核 exec-plan 後**，再讓 Agent 執行。

---

### 第五步：執行實作

```
exec-plans/XXX.md 確認沒問題，開始執行。
```

Agent 寫 `src/` + `tests/`，依照 exec-plan 逐步進行。

---

### 第六步：commit 前強制更新（必做）

實作完成後，commit 之前：

```
完成了，請執行 commit 前 checklist：
更新 PLAN.md、.docs/context.md，並把 exec-plans/XXX.md 移入 done/
```

或直接把這條寫進 `AGENTS.md`，讓 Agent 自動執行，不需要你提醒（見下方）。

---

## 把開場白變成 Prompt 檔案（一勞永逸）

建立 `.github/prompts/startup.prompt.md`：

```markdown
---
description: "啟動開發 session，對齊專案狀態"
---
請先讀 AGENTS.md、PLAN.md、.docs/context.md。
讀完後摘要當前進度，然後等我指示今天的任務。
```

之後在 Chat 輸入 `/startup` 就會自動觸發，不用每次手動貼。

---

## 在 AGENTS.md 設定 Commit 前 Checklist（推薦）

把以下內容加入 `AGENTS.md`，Agent 每次完成任務後會自動執行，不需要你提醒：

```markdown
## Commit 前必做
- [ ] PLAN.md 進度已更新
- [ ] .docs/context.md 已更新（下次焦點、已知問題）
- [ ] exec-plans/XXX.md 移入 done/
```

---

## 整個循環

```
開新對話
    ↓
/startup（AI 讀 AGENTS.md + PLAN.md + context.md）
    ↓
確認 spec / architecture / api 是否需要更新
有落差 → 先更新 .docs/ 對應檔案
    ↓
AI 寫 exec-plans/XXX.md（文件變更 + 實作步驟兩段）
你審核後再執行
    ↓
AI 執行實作（寫 src/ + tests/）
    ↓
commit 前強制更新：
  - PLAN.md（進度）
  - .docs/context.md（當前焦點）
  - exec-plans/XXX.md 移入 done/
    ↓
git commit → 明天重複
```
---
## 分工對照表

### 👤 你（Human）要寫 & 看的

| 檔案/資料夾 | 你要做什麼 |
|------------|-----------|
| `README.md` | 寫專案概述、背景說明 |
| `requirements.txt` | 管理套件依賴 |
| `.docs/rules.md` | 定義編碼風格、命名規範、禁用套件 |
| `.docs/product-specs/` | **寫需求規格**，告訴 Agent 要做什麼功能 |
| `AGENTS.md` | 設定 Agent 的權限、語氣、可用工具 |
| `.docs/api.md` | 定義 API 介面規格（你決定） |

---

### 🤖 Agent 要寫 & 看的

| 檔案/資料夾 | Agent 要做什麼 |
|------------|--------------|
| `PLAN.md` | **讀 + 更新**整體進度與藍圖 |
| `.docs/architecture.md` | **讀**確認模組結構再動手寫 code |
| `.docs/context.md` | **讀**目前開發焦點、已知 bug |
| `.docs/skill.md` | **讀**自己能用哪些工具/Function |
| `.docs/exec-plans/` | **寫執行計畫**，完成後移到 `done/` |
| `src/` | **寫主要程式碼** |
| `tests/` | **寫測試腳本** |

---

### 🔄 你們都要看的（溝通橋樑）

| 檔案 | 說明 |
|------|------|
| `AGENTS.md` | 你寫規則，Agent 每次啟動都讀 |
| `PLAN.md` | 你確認進度，Agent 持續更新 |
| `.docs/exec-plans/` | 你審核計畫，Agent 執行後回報 |

---

**核心流程：**
```
你寫 product-specs → Agent 讀 → Agent 寫 exec-plan → 你審核 → Agent 寫 src/ + tests/
