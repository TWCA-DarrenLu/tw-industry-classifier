# 工作交接

- **任務**：導入 Claude／Codex 共用文件結構（`AGENTS.md` ＋ `docs/ai/`），比照 TWCA_DA 2026-09-07 的做法。
- **更新時間／交出者**：2026-09-07／Claude（Opus 5）
- **Branch／HEAD**：`docs/cross-agent-setup`，自 `main` 開出（開分支時 `main` 與 `origin/main` 同步、工作樹乾淨）。接手時請 `git log --oneline -3` 核對。

## 已完成／修改路徑

| 檔案 | 變更 |
|---|---|
| `AGENTS.md` | **新增**。安裝器 managed 區塊 ＋ **指路型**代理工作規則（規則兩層、實掃母體、驗收閘門、環境、資料邊界） |
| `CLAUDE.md` | **新增**。原本不存在；僅含 `@AGENTS.md` managed 區塊 |
| `docs/ai/memory.md` | **新增**。目前兩條：下游消費者 TWCA_DA 的值域影響史、`py -3.12` 環境事實 |
| `docs/ai/handoff.md` | **新增**（本檔） |

**未動任何程式碼、測試或資料。**

## 與 TWCA_DA 做法的差異（重要）

TWCA_DA 是把既有 430 行 `CLAUDE.md` **逐字節搬進** `AGENTS.md`。本專案**不能照抄那個做法**，因為：

- 本專案原本**沒有 `CLAUDE.md`**，沒有東西可搬
- 知識已在 `README.md`（485 行）與 `docs/` 五份文件，且 `docs/分類邏輯表.md` 是**由 `core/emit_rule_table.py` 生成**的——複製一份進 `AGENTS.md` 會立刻產生第二個真相來源，違反本專案「規則表與文件同源」的設計

因此 `AGENTS.md` 寫成**指路文件**：只放「代理動手前不知道就會做錯」的規則與去哪讀，規格本文一律指回 `README.md` 與 `docs/`。

## 驗證與有效方法

- **`@AGENTS.md` 實際載入（已通過）**：於本目錄開新 session 提問，三題皆答對——加子分類要先進 `SUBGROUPS`、`_match_institutional()` 實掃母體、`py -3.12`。
- **中文編碼（已通過）**：三個新檔皆回讀確認無亂碼。
- 未動程式碼，故未跑 `pytest`／`test_consistency.py`。

## 未完成／下一步

1. `AGENTS.md` 的內容源自 `README.md`（2026-09-07 當時版本）。**README 若大改，本檔要跟著核對**——目前沒有自動檢查機制防止兩者漂移，這是已知弱點。
2. 本分支尚未 merge 回 `main`、尚未 push（交接當下狀態；實際以 `git log` 為準）。
3. 其餘 Lab 專案尚未導入。已完成：TWCA_DA、industry_classifier。

## 待決事項

- `feat/require-customer-name` 有一筆未併入 `main` 的 commit（`fac71b2` 名稱改為必填），非本次任務範圍，未處理。
