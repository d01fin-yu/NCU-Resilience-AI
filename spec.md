---
title: 開發規格：中大校園快報（NCU Resilience AI）

---

# 開發規格：中大校園快報（NCU Resilience AI）

> 版本：v2.3（2026-11-16）｜賴嘉妤
> 平台由另一門課的團隊開發；本課分工已向助教說明
> v2.3：依平台原始碼補上模型、技術架構、風險與突增參數、成績單換算、資料來源取得方式、檔案結構與函式清單；新增「作者與分工」。
> v2.2：依示範站後台實際資料修正 AI Skills、資料來源、風險評估、突增判斷、公開規則、What-if 與資料模型。
> v2.1：補上「問題證據」、「AI Skills 清單」、「檔案結構與主要函式」、「剩餘 4 週工作」的框架。
> 本文件只描述系統本身。時程、競賽、簡報、驗證計畫見 `docs/plan.md`。
> 標示：
> 【已實作】表示已實作完成
> 【待確認】尚未決定或尚未查證
> 【待填】依實際資料填寫，不要填預估值
> 【第二版】之後再做
> 後台內容依 2026-11-15 示範站後台，技術細節依平台原始碼（2026-11-16 讀取）。原始碼內另有開發團隊的需求規格書（`docs/requirements/`）與 62 項任務的執行計畫（`docs/plan/execution-plan.md`），本文件以實際程式為準。

---

## 0. 作者與分工

| 項目 | 說明 |
| --- | --- |
| 平台開發 | 由另一門課的團隊開發 |
| 開發方式 | 用 Claude Code 依 6 階段、62 項任務開發；每項任務有驗收方式，重大決定寫成 ADR（共 22 份） |
| 分工 | 已向助教說明；我負責的部分：規格整理、問卷與驗證、簡報與發表 |

---

## 1. 專案目標

把散落在學校公告、新聞、社群與匿名回報中的校園事件，由 AI 整理成快報、歸成系列，追蹤學校的改善措施，並用「發現、回應、恢復、調適」四個問題替學校的處理能力打分數。

**主案例**：腳踏車失竊。同一車棚一再失竊、資訊分散、改善有沒有效沒人追，熱點轉移也沒人發現。

### 1.1 問題證據

只放實際取得的資料；還沒取得的寫「蒐集中」，不要填預估值。

| 證據 | 內容 | 狀態 |
| --- | --- | --- |
| 問卷 | 有效份數＿＿；被偷過的比例＿＿；最常被偷的地點＿＿；有報案或通報的比例＿＿；平均沒車天數＿＿ | 題目已完成，蒐集中 |
| 訪談 | ＿＿位騎車同學的原話（去識別化） | 蒐集中 |
| 正式站真實來源 | 腳踏車失竊相關的公告或貼文＿＿則 | 補充中 |
| 基準線 | 人工整理同一批貼文：＿＿則花＿＿分鐘、錯誤＿＿處 | 下星期量測 |

示範站的 41 起為假資料，只用來展示功能，不當成問題證據。

---

## 2. 範圍

| 做 | 不做 |
| --- | --- |
| 8 類校園事件：設施、天氣、治安、交通、健康、火災、學業、資安 | 報案或緊急通報（頁面導向 110／119、校安中心） |
| 來源：中央大學各單位公告與新聞網、Facebook NCU 社團、中央氣象署警特報、匿名回報（見 6.6 節） | 主動寄信或通知學校單位 |
| 事件整理、系列分析、改善追蹤、韌性成績單 | 指認嫌犯、描述可疑人士外貌 |
| 公開站與管理後台 | 取代校安中心或學校既有報修系統 |

---

## 3. MVP 功能

| # | 功能 | 使用者做什麼 | 系統做什麼 | 狀態 |
| --- | --- | --- | --- | --- |
| M1 | 事件快報 | 依類別瀏覽、搜尋快報 | AI 把來源整理成摘要，標示查證狀態、風險、所屬系列 | 【已實作】 |
| M2 | 系列分析 | 點進一個系列 | 程式算累計件數、每週件數、主要地點、突增週、措施前後比較；AI 列規律與可能原因（標推測） | 【已實作】 |
| M3 | 校園韌性成績單 | 查看學校處理能力 | 依發現、回應、恢復、調適四面向計分 | 【已實作】 |
| M4 | 匿名回報 | 寫下看到的事，用收據碼查進度、補件 | 不記錄 IP 與裝置；原文只在後台 | 【已實作】 |

**進階功能【已實作】**：改善進度追蹤、What-if 模擬、通知中心、事件關聯、趨勢解讀

**剩餘 4 週要做的**：問卷與驗證、記錄第 10 節已知問題並回報開發團隊，見第 15 節。平台程式的修改由開發團隊決定。

**第二版**：擴充類別、自動通知單位、自動搜尋新來源。

---

## 4. AI、規則、人的分工

| 角色 | 負責 | 不負責 |
| --- | --- | --- |
| AI | 判斷相關性、擷取事件、分類、對應地點、寫摘要；提出事件關聯與來源立場；風險分項打分；歸系列；找系列規律與可能原因；提出改善建議；趨勢解讀；What-if 結果解釋 | 決定是否公開；計算件數、趨勢、成績單等統計數字；決定風險等級；把推測改成「已確認」 |
| 程式規則 | 公開規則、個資遮蔽；風險等級加權；件數、每週趨勢、突增；措施前後比較；成績單計分；What-if 蒙地卡羅計算；AI 歸系列失敗時的備援規則 | 理解語意 |
| 人（管理者） | 查證、合併拆分、確認或駁回關聯與原因、建立與更新措施、下架、撤回通知 | — |

風險評估：AI（risk-assessor）只替五個分項打 0–100 分並附不確定度，程式依 `config/risk-policy.yaml` 計算：

- 加權：人身安全 0.30、營運 0.20、財產 0.15、聲譽 0.15、影響範圍 0.20（試行假設，未經實證校正）
- 分級：≥30 中、≥60 高、≥80 極高
- 人身安全 ≥80 直接列極高；任一分項 ≥80 至少為高
- 政策版本 `2026-10-21.1`，每筆評估都記錄版本

### 4.1 AI Skills 清單【已實作，依後台 Skills 頁】

共 15 個 Skill。每次執行都記錄版本、成功或失敗、是否放棄（abstain）與費用；證據不足時 Skill 會放棄，不硬給答案。

| Skill | 版本 | 做什麼 | 放棄或不呼叫的情況 | 近 7 天（示範站） |
| --- | --- | --- | --- | --- |
| relevance-filter | 1.2.0 | 判斷來源內容是否和校園事件有關 | — | 0 次 |
| event-extractor | 1.5.1 | 從來源擷取事件：標題、地點、發生日期等 | — | 72 次 |
| event-classifier | 1.0.0 | 判斷事件類別（8 類） | — | 71 次 |
| location-resolver | 1.0.0 | 把地點對應到標準地點與別名 | 無法確定時放棄 | 15 次，放棄 13 次 |
| event-linker | 1.0.0 | 提出事件之間的關聯：同一事件、相關事件、推測原因，附分數 | — | 118 次 |
| evidence-verifier | 1.0.0 | 判斷每則來源對事件是支持、反駁或只是提及 | — | 71 次 |
| risk-assessor | 1.0.0 | 替五個風險分項打分並附不確定度（等級由程式加權） | — | 71 次 |
| bulletin-writer | 1.0.1 | 寫公開快報的摘要 | — | 68 次 |
| series-matcher | 1.0.0 | 判斷事件要不要歸入既有系列（失敗時改用規則歸入） | — | 58 次 |
| series-analyst | 1.3.0 | 系列分析：突增說明、地點分組、規律、可能原因、改善建議，每條附引用證據 | 證據不足時放棄 | 21 次，放棄 7 次 |
| root-cause-analyst | 1.0.0 | 單一事件的原因分析 | — | 0 次 |
| solution-planner | 1.0.0 | 單一事件的改善方案 | — | 0 次 |
| trend-forecaster（介面標 S09） | 1.0.0 | 解讀程式偵測到的突增，提出質性的「值得留意」 | 沒有突增或資料覆蓋過低時不呼叫；證據不足時放棄 | 1 次 |
| scenario-simulator | 1.0.0 | 解釋 What-if 引擎的結果與假設 | 措施效果全部未知時不呼叫 | 1 次 |
| impact-evaluator | 1.0.0 | 評估採用哪些建議可能處理到哪些原因 | — | 1 次 |

- **模型**：OpenAI `gpt-5-mini`（推理強度 low，每次最多輸出 4000 token）；事件關聯的候選召回另用 `text-embedding-3-small`（1536 維）。設定集中在 `config/llm.yaml`。
- **費用上限**：每次呼叫送出前估算，超過 0.05 美元不送；每日累計上限 1 美元。
- **評測現況**：
  - repo 的 `eval/reports/` 有 15 個 Skill 用真實模型跑的評測報告，全部通過，指標都是 1.0。
  - 但每個 Skill 的測試案例只有 12–38 筆，都是開發者自己寫的，不是真實資料。
  - 2026-11-11 起（ADR-0021）已移除「準確度要達門檻」的測試，只保留輸出格式與證據可追溯的檢查。
  - 示範站後台的 Skills 頁沒有載入這些報告。
  - 因此仍需要用真實樣本補評測（剩餘工作 R9）。
- **示範站的 AI 結果**：灌入假資料時用程式內建的假模型（`DemoProvider`），依劇本產生回應、不呼叫 API；灌完之後在示範站上的新操作才呼叫真實模型。上表執行次數大多來自灌資料，不代表實際使用量。

共同規則：AI 不產生任何統計數字；所有輸出先標「推測」或「尚未查證」，由人確認。

---

## 5. 技術架構

```
資料來源 ──定期擷取──► AI 整理 ──► 事件 ──► 系列
                                   │         │
                               風險、查證   規律與原因（AI 推測）
                                            件數、趨勢、突增（程式）
                                   ▼         ▼
                     公開規則（程式）──► 公開站
                                   │
                     管理後台：查證、合併拆分、措施、下架、What-if
```

| 項目 | 內容 |
| --- | --- |
| 前端 | 單頁式網頁（Vite 打包）；公開站與後台同站，後台路徑 `/admin` |
| API | REST，`/api/v1/public`（公開）、`/api/v1/admin`（需登入） |
| 回應格式 | `{ "data", "meta": { "request_id", "timestamp", "next_cursor", "limit" }, "error" }`，列表以 cursor 分頁 |
| 時區 | 顯示一律 Asia/Taipei |
| 後端 | Python 3.12、FastAPI（模組化單體）、Alembic（資料庫遷移）、Celery＋Redis（排程與背景工作） |
| 資料庫 | PostgreSQL 16＋pgvector（事件向量） |
| 前端框架 | React 19、TypeScript、Vite、Tailwind CSS 4 |
| 部署與監控 | Docker Compose；Prometheus＋Grafana；示範站是獨立的 compose project（`ncu-demo`），資料庫分開 |
| AI | OpenAI `gpt-5-mini`＋`text-embedding-3-small`，15 個 Skill（第 4.1 節）；開發使用 Claude Code |
| 設定檔 | `config/` 下的 YAML：llm、risk-policy、publish-policy、notification-policy、sources、locations、recurrence、impact；數字只能寫在設定檔，不寫死在程式裡 |
| 部署 | 站主電腦＋Cloudflare 臨時通道；分「示範站（假資料）」與「正式站（真實資料）」兩個環境 |
| 機密 | API 金鑰與後台帳密放 `.env`，不進版本控制 |

---

## 6. 資料模型

### 6.1 事件【已實作】

```json
{
  "id": "uuid",
  "kind": "content",
  "version": 1,
  "published_at": "2026-11-13T11:03:49+00:00",
  "event_type": "security",
  "label": "治安",
  "title": "總圖前車棚腳踏車座墊被拆",
  "summary": "總圖前車棚有腳踏車的座墊與車燈被拆走。",
  "location": "總圖前車棚",
  "occurrence_date": "2026-11-10",
  "risk_level": "medium",
  "verification": "unverified",
  "verification_label": "尚未查證",
  "sources": [],
  "series": {"id": "uuid", "name": "腳踏車失竊", "ordinal": 41, "event_count": 41}
}
```

| 欄位 | 允許值 |
| --- | --- |
| `kind` | `content` 事件內容、`safety_notice` 安全提醒 |
| `event_type` | `facility`、`weather`、`security`、`traffic`、`health`、`fire`、`academic`、`infosec` |
| `risk_level` | `low`、`medium`、`high`、`critical` |
| `occurrence_date` | `YYYY-MM-DD`；不明時為 `null` |
| 事件狀態（後台） | `detected`、`analyzing`、`unverified`、`verified`、`disputed`、`false`、`monitoring`、`resolved`、`archived` |
| 風險分項（後台） | 人身安全、營運、財產、聲譽、影響範圍 |
| 來源立場（後台） | `supports` 支持、`contradicts` 反駁、`mentions` 提及 |

後台的事件另外包含：

| 欄位 | 說明 |
| --- | --- |
| `occurrence_start_date` | 發生日期（公開 API 輸出為 `occurrence_date`） |
| `merged_into` | 被併入的目標事件；未合併為 `null` |
| `summary` | 摘要文字、引用的來源 ID、產生它的 AI 執行紀錄 |
| `evidence[]` | 每則來源：來源代碼、發布時間、立場、摘錄 |
| `risk_assessments[]` | 總分、等級、五個分項分數與不確定度、風險政策版本、AI 執行紀錄 |
| `series_id` | 所屬系列 |

### 6.2 系列【已實作】

| 欄位 | 說明 |
| --- | --- |
| `title`、`title_source` | 系列名稱與來源（`ai` 或人工改名） |
| `events[]` | 每起事件與加入方式：`ai`（series-matcher）或 `fallback`（規則） |
| `analysis.bursts` | 突增：週別、件數、期望值、說明 |
| `analysis.location_groups` | 依地點分組與件數 |
| `analysis.patterns` | 規律：描述、件數、引用原文片段與證據 ID、是否提到特定對象 |
| `analysis.causes` | 可能原因：描述、件數、引用證據、待查證事項（`to_verify`）、狀態 `hypothesized` 推測／`confirmed` 已確認 |
| `analysis.measures` | 改善建議：類型（`prevent` 預防／`respond` 應變）、期程（短中期）、對象（學校／學生）、負責單位、優先度、是否已加入追蹤 |
| `actions` | 已追蹤的改善措施 |

### 6.3 改善措施【已實作】

| 欄位 | 說明 |
| --- | --- |
| `state` | `discovered` 已發現 → `handling` 處理中 → `recovered` 已恢復／`improved` 已改進；或 `dropped` 不追蹤 |
| `title`、`owner`、`due_at`、`proof`、`overdue` | 名稱、負責單位、期限、佐證、是否逾期 |
| `origin` | `manual` 人工建立，或來自事件分析 |

### 6.4 問卷回覆【建議格式】

問卷資料不轉成平台事件，避免一份問卷被算成一起失竊。是否匯入平台由開發團隊決定。

| 欄位 | 說明 |
| --- | --- |
| `response_id` | 編號；不存姓名、學號、聯絡方式 |
| `ever_stolen`、`times` | 是否被偷過、次數 |
| `location`、`approx_month` | 地點（套用地點別名）、大約月份 |
| `reported` | 是否報案或通報 |
| `no_bike_days` | 沒車可騎的天數 |
| `wanted_measures` | 希望的措施（複選） |

### 6.5 其他後台資料【已實作】

| 資料 | 說明 |
| --- | --- |
| 地點 | 標準名稱、別名（例：「女十四舍車棚」併入「宿舍區車棚」）、上下層關係、事件數 |
| 事件關聯 | 類型 `same_event`、`related_event`、`hypothesized_cause`；提出方式 `ai`／`rule`／`manual`；分數；人工決定 `pending`／`confirmed`／`rejected` |
| 通知 | 對象目前為站主（`owner`）；內容用固定範本，未查證的寫「有未經查證的訊息…目前無法確認是否屬實」；狀態 `candidate` → `policy_checked` → `eligible`／`blocked` → `published` → `updated`／`retracted`／`expired` |
| 公開內容 | 對象（事件或系列）、層級（`content`／`safety_notice`／`none`）、狀態、版本、公開理由、是否需人工檢查（第 8 節） |
| 匿名回報 | 原文、地點、補件、分析狀態、歸入的事件；後台也不顯示收據碼 |
| 稽核紀錄 | 執行者（管理者或系統服務）、動作、對象、時間、政策版本 |

### 6.6 資料來源【已實作】

| 代碼 | 來源 | 擷取間隔 |
| --- | --- | --- |
| `anonymous-report` | 匿名回報 | 10 分鐘 |
| `facebook` | Facebook NCU 社團 | 1 小時 |
| `cwa-alerts` | 中央氣象署天氣警特報與地震報告 | 10 分鐘 |
| `ncu-announcements` | 中央大學校園公告 | 12 小時 |
| `ncu-news` | 中央大學新聞網 | 12 小時 |
| `ncu-aa`、`ncu-osa-life`、`ncu-sec`、`ncu-oia` | 教務處、學務處生活輔導組、秘書室、國際事務處公告 | 12 小時 |
| `ncu-cc`、`ncu-lib`、`ncu-epsc` | 電算中心、圖書館、環安中心公告 | 12 小時 |

- 每個來源記錄健康狀態、連續失敗次數、成功率、待處理失敗項目。
- 示範站只啟用匿名回報，其他來源停用。
- **Facebook 社團**：Meta 已移除社團 API，沒有合法的機器存取管道。目前做法是用 Claude in Chrome 在開發者本人已登入的瀏覽器讀取，每天最多一次、只讀不互動，再用人工匯入格式進系統；不存發文者資訊，原始檔不進 git。
  - Meta 條款禁止未經授權的自動化擷取，帳號風險由開發者自行承擔（ADR-0008）。
- **不核准的來源**：Dcard（條款限制，改走人工匯入）、中央社、Google News。

---

## 7. 計算規則（全部由程式計算）

### 7.1 系列分析

| 項目 | 規則 |
| --- | --- |
| 累計件數 | 系列內事件數 |
| 每週件數 | 依 `occurrence_date` 分週；可切 1 個月、3 個月、6 個月、1 年、全部 |
| 4 週平均 | 每週的移動平均，滿 4 週資料才顯示 |
| 突增週 | Poisson 變化偵測：先用之前各週算 EWMA（α＝0.3）當期望值，下限 0.5，再算 P(X ≥ 觀察值)。同時符合以下三條才算突增：至少有 4 週歷史、p < 0.001、當週至少 5 起。例：總圖前車棚 11/2 那週觀察 7 起、期望 0.5 起（示範資料） |
| 措施前後比較 | 以措施開始處理日為界，比較前後的每週平均與變化比例；任一邊少於 5 起時狀態為「資料還不夠」。例：宿舍區車棚加裝監視器，前 26 起、後 13 起（8 週），每週 3.25 → 1.625 起，變化 −50%（示範資料） |

### 7.2 韌性成績單

| 面向 | 指標 |
| --- | --- |
| 發現 | 發生日期 → 系統收到的天數 |
| 回應 | 收到 → 措施「處理中」的天數 |
| 恢復 | 處理中 → 已恢復／已改進的天數 |
| 調適 | 措施完成率、重複發生比例 |

- 預設統計 180 天，四面向權重各 25%
- 任一指標樣本少於 5 筆顯示 N/A；四面向都有分數才算總分
- 天數換算分數：線性換算，≤ 最佳天數為 100 分、≥ 最差天數為 0 分。發現 1–14 天、回應 1–30 天、恢復 3–60 天；這些錨點是本專題自訂，不是校方標準
- 調適：措施完成率，與重複發生比例（同地點、同類別有 1 起以上）
- 公開版只統計已查證並公開的事件；後台可切換為統計全部事件
- 發生日期不明的事件不列入「發現」；尚未恢復的措施不列入「恢復」

### 7.3 What-if 模擬

| 項目 | 規則 |
| --- | --- |
| 輸入 | 期間天數、類別與地點（選填）、各措施名稱、效果上下限、成本、依據、隨機種子 |
| 基準件數 | 預設由程式依歷史件數自動算（記錄用了哪段期間、幾週、觀察幾起），也可人工輸入 |
| 計算 | 蒙地卡羅模擬 2000 次，各措施效果在上下限之間抽樣；假設措施效果互相獨立 |
| 輸出 | 點估計、中位數、5%–95% 範圍、預估減少比例、各措施敏感度排名（龍捲風圖）、總成本與成本等級；記錄引擎版本 |
| AI 解釋 | scenario-simulator 只解釋引擎結果與假設，不產生數字 |
| 限制標示 | 一律標「假設情境，不是成效預測」；效果未知的措施不納入；基準件數少於 5 時標「結果不穩定」 |

---

## 8. 公開規則與個資

1. 是否公開由程式規則決定，不由 AI 決定；公開規則有版本號，每次公開都記錄當時的規則版本。
2. 只有官方來源證實或管理者查證的事件標「已查證」，其他一律標「尚未查證」。
3. AI 提出的規律與原因只出現在系列頁，標「推測，尚未查證」；管理者確認後才標「已確認」。
4. 社群來源與匿名回報只顯示 AI 摘要，不顯示原文、連結、發文者。
5. 匿名回報原文永不公開；不記錄 IP 與裝置、不要求聯絡方式。
6. 公開內容遮蔽電話、Email。
7. 提到特定個人、群體、店家或單位時，標記「需人工檢查」並提醒管理者，管理者可立即下架。
8. 頁面固定說明：回報變多不等於校園變危險；件數變少不一定是措施的功勞。
9. 每頁標示：學生專題、非校方系統、不是報案管道。
10. 每則公開內容記錄公開理由：`not_confirmed` 尚未證實、`speculation` 含推測、`possible_allegation` 疑似指控、`remind` 提醒。

---

## 9. 介面與 API

### 9.1 公開站頁面【已實作】

| 頁面 | 路徑 | 內容 |
| --- | --- | --- |
| 快報首頁 | `/` | 依類別分區的快報卡、搜尋、成績單與匿名回報入口 |
| 快報全文 | `/events/{id}` | 類別、查證、風險、日期、地點、摘要、官方來源、所屬系列 |
| 系列分析 | `/series/{id}` | 第 7.1 節各項、規律與原因、每一起清單、地點分布、改善建議與追蹤 |
| 事件查詢 | `/events` | 8 類篩選與件數 |
| 改善進度 | `/progress` | 各系列措施與狀態 |
| 韌性成績單 | `/resilience` | 第 7.2 節各項與資料限制 |
| 匿名回報 | `/report` | 10–2000 字、選填地點、收據碼查詢與補件 |
| 關於本站 | `/about` | 專題聲明、公開規則 |

### 9.2 後台頁面【已實作】

事件工作台、通知中心、改善追蹤、事件關聯、系列、地點、匿名回報、韌性成績單、What-if 模擬、公開內容、Skills、運維。

### 9.3 公開 API【已實作，依原始碼】

| 路徑 | 用途 |
| --- | --- |
| `GET /api/v1/public/feed` | 快報列表（cursor 分頁） |
| `GET /api/v1/public/notices` | 安全提醒 |
| `GET /api/v1/public/categories` | 類別與件數 |
| `GET /api/v1/public/events`、`/events/{id}` | 事件查詢、單則快報 |
| `GET /api/v1/public/series/{id}` | 系列分析 |
| `GET /api/v1/public/progress` | 各系列改善措施 |
| `GET /api/v1/public/resilience` | 成績單 |
| `POST /api/v1/public/reports` | 匿名回報，回傳收據碼（202） |
| `POST /api/v1/public/reports/supplements` | 補件與進度查詢 |

### 9.4 後台 API【已實作，前綴 `/api/v1/admin`】

| 模組 | 路徑 |
| --- | --- |
| 事件 | `/events`、`/events/{id}`、`/verify`、`/merge`、`/split`、`/analyze`、`/actions` |
| 改善措施 | `/actions`、`/actions/{id}`、`/transitions` |
| 通知 | `/alerts`、`/alerts/{id}/retract`、`/recurrences` |
| 事件關聯 | `/relations`、`/relations/{id}/decision` |
| 系列 | `/series`、`/series/{id}`、`/timeline`、`/actions`、`/causes/{id}/decision`、`/events/{id}/split`、`/dissolve` |
| 地點 | `/locations`、`/locations/{id}/split` |
| 公開內容 | `/publications`、`/{id}/takedown`、`/{id}/restore` |
| 匿名回報 | `/anonymous-reports` |
| 成績單與趨勢 | `/resilience`、`/trends/forecast` |
| What-if | `/scenarios`、`/scenarios/baseline`、`/scenarios/impact` |
| 運維 | `/dashboard`、`/ingestion-runs`、`/ingestion-runs/health`、`/audit-logs`、`/skills` |

---

## 10. 已知問題（正式站，2026-11-14）

| 問題 | 例子 | 期望行為 |
| --- | --- | --- |
| 預定日期被當成發生日期 | 「風雨球場 12/15 開工」記成 2026-12-15 發生 | 未來日期或「預計」字樣不填入 `occurrence_date` |
| 發生日期缺漏 | 數則為 `null` | 允許 `null`，但不列入「發現」計算（已符合） |
| 系列名稱像句子 | 「校內規劃在12月中動工的風雨球場，預計4月底完工」 | 系列名稱不超過 12 字、不含日期 |
| 類別或範圍可能錯 | 「住家門口抽菸」歸治安，也不一定在校內 | 校外事件不收或另外標示 |
| 單一事件自成系列 | 多個系列只有 1 起 | 【待確認：系列成立門檻】 |
| 文件說法不一致 | 平台 README 與 ADR-0008 寫「個人使用、不對外公開」，但 ADR-0015 起已有公開網站 | 回報開發團隊更新 |

---

## 11. 驗收條件

| # | 條件 | 怎麼驗 |
| --- | --- | --- |
| A1 | 每則快報都顯示查證狀態；沒有官方來源也沒人工查證的，一律是「尚未查證」 | 正式站逐則檢查 |
| A2 | 公開內容不出現電話號碼與 Email | 送含電話、Email 的匿名回報，確認公開摘要已遮蔽 |
| A3 | 匿名回報送出後拿到收據碼，可查進度與補件；10 分鐘內第 21 次查詢被擋 | 實際操作 |
| A4 | 系列的累計件數、每週件數和手算一致 | 匯出腳踏車系列，用 Excel 對照 |
| A5 | 措施前後任一邊少於 5 起時，顯示「資料還不夠」，不給比較結果 | 示範站「總圖前車棚張貼防竊提醒」 |
| A6 | 成績單任一面向樣本少於 5 筆時顯示 N/A，且不算總分 | 正式站目前回應、恢復為 N/A |
| A7 | AI 規律與原因預設標「推測，尚未查證」；只有管理者確認後才變「已確認」 | 後台改狀態後看前台 |
| A8 | 轉貼或同一起失竊的多篇貼文，合併成一個事件 | 40–60 則標註樣本：另請一人或自己隔幾天再標一次，取得標準答案；計算平台的誤合併數、漏合併數，與人工基準線比較。目標值等量完基準線再訂；結果寫進對應 Skill 的評測報告 |
| A9 | What-if 結果一律標「假設情境，不是成效預測」 | 執行一次模擬 |
| A10 | 被下架的內容在公開站與 API 都查不到 | 後台下架後查 `/events/{id}` |
| A11 | 預定日期不會變成發生日期（第 10 節修正後） | 送一則「下週開工」的公告 |

---

## 12. 錯誤處理

| 情況 | 行為 |
| --- | --- |
| 擷取來源失敗 | 來源健康標為不穩定或失敗；不影響已公開內容 |
| AI 輸出格式不合或執行失敗 | 執行時檢查 JSON Schema、證據 ID 是否存在、原句是否逐字出自來源、個資；不合格記為失敗，列入 Skills 失敗率 |
| 預估費用超過上限 | 不送出呼叫（每次 0.05 美元、每日 1 美元） |
| AI 證據不足 | Skill 放棄（abstain），不產生結果；例如地點無法確定時不填地點 |
| AI 歸系列失敗 | 改用規則歸入（加入方式記為 `fallback`） |
| API 錯誤 | 回應 `error` 物件；前台顯示「發生未預期的錯誤」 |
| 後台登入逾時 | 導回登入頁 |
| 查無事件或系列 | 顯示「不存在，或可能已經下架」 |
| 公開版無任何快報 | 顯示「目前沒有公開的快報」 |

---

## 13. 檔案結構【已實作，依原始碼】

```
ncu-resilience-ai/
├── README.md、AGENTS.md、CLAUDE.md   # 專案說明；AI 開發規範
├── Makefile                          # make up／test／migrate／demo-up／demo-share 等指令
├── .env.example                      # 環境變數範本（金鑰只放本機 .env）
├── backend/                          # FastAPI 後端
│   └── app/
│       ├── api/v1/{public,admin}     # 公開與後台 API
│       ├── ingestion/                # 來源擷取（各來源的 Adapter）
│       ├── pipeline/                 # 每則來源的處理流程（相關性 → 擷取 → 分類 → 關聯）
│       ├── ai/                       # Skill Runner、模型呼叫、評測
│       ├── events/、relations/、series/、locations/  # 事件合併拆分、關聯、系列、地點
│       ├── verification/、risk/       # 查證規則、風險政策計算
│       ├── publishing/、alerts/       # 公開規則引擎、下架、通知與個資遮蔽
│       ├── analytics/                # 時間序列、突增偵測、趨勢
│       ├── resilience/               # 改善措施、韌性成績單、再發生
│       ├── simulations/              # What-if 引擎與基準件數
│       └── demo/                     # 示範站假資料與假模型
├── frontend/                         # React 公開站與後台
├── skills/                           # 15 個 AI Skill：SKILL.md、輸出 schema、測試案例
├── config/                           # 政策與參數 YAML（第 5 節）
├── eval/                             # 評測門檻與報告
├── infra/                            # Docker、資料庫遷移、監控、tunnel
├── scripts/                          # Facebook 匯入、標註、評測、報表等工具
└── docs/                             # 需求規格、執行計畫、架構、ADR、治理、進度
```

## 14. 主要函式清單【已實作，依原始碼】

| 功能 | 函式 | 所在檔案 | 對應驗收 |
| --- | --- | --- | --- |
| 處理一則來源（擷取、分類、關聯） | `analyze_item` | `backend/app/pipeline/analyze_item.py` | A8、A11 |
| 執行 Skill 並驗證輸出 | `run_skill` | `backend/app/ai/runner/run.py` | 4.1 節 |
| 找出個資 | `find_pii` | `backend/app/ai/runner/validate.py` | A2 |
| 遮蔽個資 | `mask_pii` | `backend/app/alerts/pii.py` | A2 |
| 查證狀態判斷 | `is_official_source` 等 | `backend/app/verification/rules.py` | A1 |
| 風險等級加權 | `calculate_risk` | `backend/app/risk/engine.py` | — |
| 公開規則判斷 | `evaluate`、`evaluate_series` | `backend/app/publishing/policy.py` | A1、A7 |
| 下架與重新上架 | `take_down`、`restore` | `backend/app/publishing/takedown.py` | A10 |
| 事件合併、拆分 | `merge_events`、`split_event` | `backend/app/events/merge.py` | A8 |
| 歸入系列 | `assign_series` | `backend/app/series/grouping.py` | A8 |
| 每週件數 | `weekly_counts`、`count_series` | `backend/app/series/analysis.py`、`backend/app/analytics/timeseries.py` | A4 |
| 突增偵測 | `detect_changes`、`ewma`、`poisson_sf` | `backend/app/analytics/timeseries.py` | A4 |
| 措施前後比較 | `before_after` | `backend/app/series/timeline.py` | A5 |
| 韌性成績單 | `compute_cri`、`linear_score` | `backend/app/resilience/cri.py` | A6 |
| 改善措施狀態 | `transition_action`、`is_overdue` | `backend/app/resilience/actions.py` | — |
| What-if 模擬 | `simulate` | `backend/app/simulations/engine.py` | A9 |
| What-if 基準件數 | `compute_baseline` | `backend/app/simulations/baseline.py` | A9 |
| 匿名回報收據碼 | `new_receipt`、`receipt_hash` | `backend/app/api/v1/public/reports.py` | A3 |

## 15. 剩餘 4 週工作（W11–W14）

平台主體已完成。接下來只做下列項目；對應到驗收條件的，以該條通過為完成標準；對應到章節的，以該節的【待填】與＿＿全部填完為完成標準。

| # | 工作 | 對應 | 週次 | 狀態 |
| --- | --- | --- | --- | --- |
| R1 | 量人工基準線 | 1.1 節、A8 | W11 | 未開始 |
| R2 | 問卷回收與分析 | 1.1 節、6.4 節 | W11–W12 | 題目已完成 |
| R3 | 測試並記錄「預定日期被當成發生日期」，回報開發團隊 | 第 10 節、A11 | W11 | 未開始 |
| R4 | 整理第 10 節其他問題的實例，回報開發團隊 | 第 10 節 | W11 | 未開始 |
| R5 | 標註樣本與合併正確率 | A8 | W11–W12 | 未開始 |
| R6 | Excel 核對件數、前後比較、成績單 | A4–A6 | W12 | 未開始 |
| R7 | 平台來源說明、本課負責部分、AI 協助說明 | 課程評分 | W12 | 未開始 |
| R8 | 驗收條件全數檢查一遍 | A1–A11 | W13 | 未開始 |
| R9 | 用真實樣本補評測：至少 event-extractor、event-classifier、event-linker、series-matcher（現有評測只用開發者自建的 12–38 筆案例） | 4.1 節、A8、A11 | W11–W12 | 未開始 |

不在清單上的新功能（擴充類別、自動通知單位、自動搜尋新來源）一律排到第二版。