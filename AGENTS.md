
<!-- oharalab-cross-agent:start -->
開始或接續工作：讀 docs/ai/memory.md 與 docs/ai/handoff.md，再核對 branch、HEAD、git status 與相關 diff。
重要階段、正常結束或交接時更新 handoff.md；純問答與唯讀 review 不因本規則授權改檔。
依 session-handoff Skill 記錄證據、失敗原因與下一步；共同記憶不等於聊天全文。
<!-- oharalab-cross-agent:end -->

# industry_classifier — 代理工作規則

> 本檔是給 Claude／Codex 的**工作規則與指路**。
> 產品說明與完整規格在 `README.md`（485 行）與 `docs/`，**本檔不重複、不另立一套**。
> 規則本文永遠以 `core/rules.py` 與 `docs/分類邏輯表.md` 為準；本檔與它們衝突時以它們為準。

## 這個專案是什麼

輸入統一編號 → 回傳官方登錄的正式名稱 ＋ 產業分類（產業大類＋產業子類）＋ 判定依據。

**全程不使用 AI 判斷、不需人工介入**：163 條通用規則的有序瀑布，首次命中即定案，最後一層兜底。

> ⚠️ **不要在這個專案裡引入 LLM 判斷、機率模型或啟發式猜測。**
> 「同一份輸入搭配同一份資料快照，任何人執行都得到完全相同的結果」是本專案的核心保證，
> 且由 `tests/test_consistency.py` 把關。任何引入不確定性的「改良」都是在打破產品定義。

## 動手前先讀哪一份

| 你要做的事 | 先讀 |
|---|---|
| 改分類規則 | `README.md` §要改規則的時候 → `core/rules.py` |
| 看規則全表 | `docs/分類邏輯表.md`（**由 `core/emit_rule_table.py` 生成，不要手改**） |
| 判定流程與層級（L0–L4） | `README.md` §判定流程 |
| 值域有哪些 | `README.md` §值域 → 定義在 `core/rules.py` 的 `SUBGROUPS` |
| 資料來源、快照與更新 | `docs/資料來源與更新.md` |
| 名冊維護與爬蟲 | `docs/名冊維護與爬蟲.md` |
| 做不到的事 | `docs/已知限制.md` |

## 規則分兩層，改的地方不同

**通用規則 → `core/rules.py`**（純資料，零 import、零路徑）

- 稅籍分類改版 → 改 `SUB_BY_MAJOR2` 與 `MAJOR2_LABEL`
- 要加子分類 → **必須先加進 `SUBGROUPS` 值域**，否則值域自我檢查會擋下來
- 制度性名稱規則 → `INSTITUTIONAL_RULES`；一般名稱關鍵字 → `NAME_RULES`。**兩者順序即優先序**
- 防呆清單：`CORP_SUFFIX_EXCLUDE`／`CORP_SUFFIX_GUARD_RULES`／`INSTITUTIONAL_KEYWORD_EXCLUDE`／`NAME_RULE_SUFFIX_EXCLUDE`

**存量裁決紀錄 → `exceptions/local_exceptions.json`**

- 只記「回溯既有名單時的一次性裁決」（帳務佔位碼、已解散舊戶、官方分類與自身業務口徑不同的個案）
- **分類 pipeline 對未來的新客戶不依賴人工裁決**。丟一個沒看過的統編進來，答案由官方名冊、稅籍碼與名稱規則決定
- **空的例外表不是待辦事項**，不要主動去填它；它隨存量名單長，不隨新客戶長
- **不要把裁決內容併回 `core/rules.py`**——這類裁決的理由涉及業務往來關係，混進通用規則會讓規則檔變成一份客戶清單，換一個組織使用時本來就該重新檢視

### 改名稱規則前，一律先實掃母體

用 `core/engine.py` 的 `_match_institutional()` 直接掃稅籍全檔，數出一條規則會命中幾筆、命中的是什麼。

v5 的四道防呆全部是這樣掃出來的——**用想的會漏掉「動物醫院」「大學城自助餐」這種**。

## 驗收閘門（改完必跑）

```bash
py -3.12 -m pytest tests/          # 規則與值域（43 項，不需外網）
py -3.12 tests/test_consistency.py # 重現性與兩模式一致性（5 項全過才算可交付）
py -3.12 core/emit_rule_table.py   # 改過規則就要重生 docs/分類邏輯表
```

- `test_consistency.py` 五項：重現性／冷啟動／輸入順序無關／值域／兩模式逐欄一致
- 無外網加 `--skip-api`（只跑前四項）；連 GCIS 名稱解析都不打再加 `--offline-local`
- 規則表與文件由同一份來源生成，**不會出現「文件寫的跟程式跑的不一樣」**——所以改了規則卻沒重生規則表，就是缺漏，不是「文件之後再補」

## 執行環境

- **一律用 `py -3.12`**。本機裸 `python` 可能指向其他虛擬環境
- 爬蟲產線（`crawlers/`）需 `py -3.12 -m pip install -r crawlers/requirements.txt`；分類器本身不需要

## 資料與隱私邊界

- **交付包不含任何名單資料**。`input/taxids_sample.csv` 是公開機構的示範資料
- golden 測試個案**一律只用公開個體**（上市公司、公立私立學校、法定金融基礎設施），F／P 案例用合成值。新增測試個案比照辦理，**不要放真實客戶**
- 規則測試（`test_v5_golden.py`）用合成 provider 與合成名冊，**不依賴 `data/` 下的真檔**；只有 3 項真資料個案會在缺檔時跳過。新增規則測試維持這個性質
