# 工作交接

- **任務**：導入 Claude／Codex 共用文件結構，並確立 `AGENTS.md` 與 `README.md` 的所有權切分。
- **更新時間／交出者**：2026-09-07／Claude（Opus 5）
- **Branch／HEAD**：`main`（`docs/agents-as-source` 已併入並刪除）。接手時請 `git log --oneline -3` 核對。

## 現行文件所有權（本次確立，動文件前先看這條）

| 檔案 | 擁有什麼 |
|---|---|
| `AGENTS.md` | **規則與紀律的唯一來源**：改哪個常數、改之前要做什麼、改完要跑什麼、哪些事不准做。人類貢獻者與 AI 代理共用 |
| `README.md` | **產品說明與使用方式的唯一來源**：這是什麼、怎麼查、值域、判定流程、資料來源、已知限制、執行環境 |
| `core/rules.py`、`docs/分類邏輯表.md` | **執行真相**。文件與程式不一致時以程式為準，回頭修文件 |

**兩份文件不互相複製**，需要對方內容時放連結。這是刻意的設計，不要「順手補回去」。

## 已完成

**第一階段（commit `6225efa`）**：安裝器建骨架 → `AGENTS.md`、`CLAUDE.md`（原本不存在）、`docs/ai/memory.md`、`docs/ai/handoff.md`。

**第二階段（本次）**：把 `AGENTS.md` 從「README 的改寫副本」改為**本文**——

- `README.md` §要改規則的時候（44 行）→ 縮為 7 行指標，紀律本文移入 `AGENTS.md`
- `README.md` §驗收 → 保留「五項檢查是什麼、為什麼可信」，指令段改為指向 `AGENTS.md`
- `AGENTS.md` → 新增所有權切分聲明；規則兩層、實掃母體、驗收閘門、資料邊界成為本文（85 行）

**未動任何程式碼、測試或資料。**

## 驗證與有效方法

- **`@AGENTS.md` 實際載入（已通過）**：於本目錄開新 session 提問，三題皆答對——加子分類要先進 `SUBGROUPS`、`_match_institutional()` 實掃母體、`py -3.12`。
- **無殘留重複（已通過）**：`_match_institutional`／`SUB_BY_MAJOR2`／`NAME_RULE_SUFFIX_EXCLUDE`／「空的例外表」在 `README.md` 的出現次數皆為 0。`emit_rule_table` 尚有 2 處，經確認為目錄結構列表與「想看規則明細就跑這個」的使用說明，用途與紀律不同，刻意保留。
- **併入 `feat/require-customer-name` 後的閘門（已通過）**：`py -3.12 -m pytest tests/` → **74 passed**；`py -3.12 tests/test_consistency.py` → 重現性三次同 SHA、順序無關、值域越界 0 筆、兩模式 15 筆逐欄相同，**全部通過**（含需連外的第四項）。`core/` 零變更故未重生 `docs/分類邏輯表`。
- **文件計數漂移實例（已修）**：`AGENTS.md` 原寫「43 項」，實跑為 74。已改為不寫數量——寫死的計數必然再次過期，這種事實應該由測試輸出說，不由文件說。

## 無效方法／原因

- **一度打算加一支 `tests/test_agents_doc.py` 斷言 README 的識別符仍存在，用來防止兩份文件漂移。使用者否決，理由正確**：那是在監控重複，而重複本身才是病灶。移除重複之後不需要那支測試，也就不必維護它。
- **讀 README 時人在 `feat/require-customer-name`、卻把成果 commit 到 `main`**。事後 diff 確認兩個 branch 的 §要改規則的時候／§驗收／§執行環境注意 逐字節相同，本次無影響；但這是運氣不是流程。**跨 branch 引用內容前先確認自己在哪條 branch。**

## 未完成／下一步

- 其餘 Lab 專案尚未導入。已完成：TWCA_DA、industry_classifier。

## 分支現況（2026-09-07 清理後）

`main` 是唯一分支，本地與 `origin` 皆然。

- `feat/require-customer-name` 已 rebase 後 fast-forward 併回 `main`（原 `fac71b2` → `f7a1359`），本地與遠端分支皆已刪除。
  刪除前已驗證兩個 SHA 的 patch **實際增刪行 md5 相同**（唯一差異是 blob index 行，因 README 基底變動必然不同），無內容遺失。
- `v5-pipeline`（`9b58643`）早已完全併入 `main`，本地分支已刪除（遠端本就沒有）。

## 待決事項

- 目前無。
