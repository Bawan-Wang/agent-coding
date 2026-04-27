很簡單，核心就是**每次對話開始時給 AI 一個固定的啟動指令**。

---

## 操作流程

### 第一步：選 Agent 模式
VS Code Chat 選 **Agent**（不是 Ask）

---

### 第二步：固定的開場白

每次開新對話，貼這段：

```
請先讀：
- AGENTS.md（你的規則和權限）
- PLAN.md（整體進度）
- .docs/context.md（當前狀態）

讀完後告訴我現在的進度，然後等我說今天要做什麼。
```

---

### 第三步：下任務

AI 對齊狀態後，你直接說：

```
繼續 exec-plans/002-vlm-model-bridge.md
```
或
```
實作 product-specs/new-user-onboarding.md 的第一階段
```

---

### 第四步：任務結束後要求更新

```
完成了，請更新 PLAN.md 和 .docs/context.md
```

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

## 整個循環

```
開新對話
    ↓
/startup（AI 讀 AGENTS.md + PLAN.md + context.md）
    ↓
你說今天要做什麼
    ↓
AI 讀對應的 exec-plans/ 或 product-specs/
    ↓
執行任務
    ↓
你說「完成，更新狀態」
    ↓
AI 更新 PLAN.md + context.md
    ↓
明天重複
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
